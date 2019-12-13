# Umbraco Grids

This package brings together Umbraco Grid, [Doc Type Grid Editor](https://our.umbraco.com/packages/backoffice-extensions/doc-type-grid-editor/), [SkyBrud Umbraco GridData](https://github.com/skybrud/Skybrud.Umbraco.GridData) and Yuzu to create a powerful out of the box, config ready grid solution. It the quickest and simplest way that we have found to implement the grid in Umbraco. 

Hooking into the Yuzu pattern library makes it easy for definition developers to add a selection of blocks available to content editors, including live preview within the Umbraco back office. 

All mapping happens through our grid item mapping service that pulls together a collection of grid aware block. This service can be overridden in the contains to implement your own mapping instance.

Couple with out Umbraco Imports package we can automate the generation of Grid document types, including the creation of Doc Type Grid Editor config. We've even included config setting defined within the pattern library that are automatically copied over and implemented in Grid editor config.

## Config

| Property    			    	| Purpose 			                        |
| ----------------------------- | ------------------------------------------|
| **UmbracoModelsAssembly**		| Assemblies that contain Umbraco Models    |

## Mapping

There are two data structures for grid, vmBlock_DataGridRows and vmBlock_DataGridRowsColumns. The first is for row only grids and the second is for grids with rows and columns. Both structures can include config objects for both rows and columns, but they are not mandatory.

### Grids

We have created some mapping wrapper classes that make it easier to work with grids in Yuzu. Which one used is dependant on how the other properties in the same Viewmodel are mapped.

When only adding grid mapping to a viewmodel the command is very simple

```c#
cfg.AddGridWithRows<HomePage, vmPage_HomePage>(src => src.GridContent, dest => dest.GridContent);
```

When manual mapping on other viewmodel properties is required then it gets a little more complex.

```c#
CreateMap<HomePage, vmPage_HomePage>()
    .ForMember(x => x.Content, opt => opt.MapFrom<GridRowConvertor<HomePage, vmPage_HomePage, vmBlock_GridConfig>, GridDataModel>(y => y.GridContent));
```

### Grid Items

Upon startup all block Viewmodels that can be directly mapped to a Umbraco Models (on name) are added as possible grid items using a generic class called `DefaultGridItem`. These generic implementations are added to a collection in the IOC container and are chosen using a strategy pattern when grid viewmodels are hydrated. 

The `DefaultGridItem` class has three method

- `IsValid` : picks the grid item by matching the grid docy type alias to the document type alias
- `CreateVM` : hydrates the viewmodel from the model using Automapper. Used in the Umbraco back office to generate the preview grid items. 
- `Apply` : Used in the strategy pattern to get the model data and then call the CreateVM above

There are many situations where this generic class is not suitable and it possible to either inherit from this class overriding methods or implement your own class that implements the IGridItem interface. When doing so, it's important to follow the pattern above where Apply calls the CreateVm method, previews in the backoffice will then reflect the production site.

here's an example taken from our One School demo site

```c#
public class TeacherProfilesGridItem : IGridItem
{
    protected TeacherProfilesService svc;

    public TeacherProfilesGridItem(TeacherProfilesService svc)
    {
        this.svc = svc;
    }

    public Type ElementType { get { return typeof(TeacherProfiles); } }

    public virtual bool IsValid(string name, GridControl control)
    {
        var content = control.GetValue<GridControlDtgeValue>();
        if (content != null)
        {
            return content.Content.ContentType.Alias == "teacherProfiles";
        }
        return false;
    }

    public virtual object Apply(GridItemData data)
    {
        var model = data.Control.GetValue<GridControlDtgeValue>();

        dynamic config = new ExpandoObject();
        config.gridSize = data.Control.Area.Grid;

        return CreateVm(model.Content, data.Html, config);
    }

    public virtual object CreateVm(IPublishedElement model, HtmlHelper html, dynamic config = null)
    {
        return svc.GetProfiles(0);
    }
    }
```

The above code uses a data respository to get teacher profiles instead of mapping Umbraco model data directly.

Register your own grid items in a composer during Umbraco bootup.

```c#
[RuntimeLevel(MinLevel = RuntimeLevel.Run)]
[ComposeBefore(typeof(YuzuGridStartup))]
public class YuzuGridComposer : IUserComposer
{
    public void Compose(Composition composition)
    {
        var assembly = Assembly.GetAssembly(typeof(YuzuGridComposer));

        var config = new YuzuDeliveryGridConfiguration()
        {
            UmbracoModelsAssembly = assembly
        };

        var types = assembly.GetTypes().Where(x => x.GetInterfaces().Any(y => y == typeof(IGridItem)));
        foreach (var f in types)
        {
            composition.Register(typeof(IGridItem), f);
        }

        YuzuDeliveryGrid.Initialize(config);
    }
}
```

## Grid Item Previews

Because Yuzu is just a collections of blocks it makes working with the grid in the back office easy. For each cell added we swap out the rebdering engine of DTGE to render the specific template for the viewmodel using the preview data. a very simple and elegant solution.

To add styling we create a backoffice stylesheet during the definition dist process. This stylesheet is crated from the core and blocks styles. We apply an all: initial; pointer-events: none; style tags the block preview wrapper div. This removes all Umbraco styles and prevents any pointer event from the block from affecting interaction. 

## Grid manipulation

Occasionally there needs to be an extra level of control to change how a setting or config item are applied to a cell or row in the grid. Below are a few exmaples of how we can use Yuzu to make these changes using the IAutomaticGridConfig interface. 

```c#
public class BackgroundRowImageGridConfig : IAutomaticGridConfig
{
    private readonly IMapper mapper;
    private readonly IPublishedContentQuery contentQuery;

    public BackgroundRowImageGridConfig(IMapper mapper, IPublishedContentQuery contentQuery)
    {
        this.mapper = mapper;
        this.contentQuery = contentQuery;
    }

    public Dictionary<string, object> AddAreaSettings(Dictionary<string, object> settings, GridRow row, GridArea currentArea)
    {
        return settings;
    }

    public Dictionary<string, object> AddRowConfig(Dictionary<string, object> settings, GridRow row)
    {
        if (settings.Any(x => x.Key == "backgroundImage") && settings["backgroundImage"] != null)
        {
            var content = contentQuery.Content(((int)settings["backgroundImage"])).ToModel<GridRowBackgroundImage>();
            if (content != null)
            {
                settings["backgroundImage"] = mapper.Map<vmBlock_ImageCropped>(content.Image);
            }
        }
        return settings;
    }
}
```
This changes backgroundImage grid setting in a row (media picker) and replaces the setting with a Cropped Image block from the pattern library. 

```c#
public class VerticalAlign_ImageGridConfig : IAutomaticGridConfig
{

    public Dictionary<string, object> AddAreaSettings(Dictionary<string, object> settings, GridRow row, GridArea currentArea)
    {
        var otherAreas = row.OtherAreas(currentArea);
        var index = row.Areas.CurrentIndex(currentArea);

        if (currentArea.HasContentType("gridImage") && otherAreas.Any(x => x.HasContentType("gridRte")))
        {
            if (!settings.Any(x => x.Key == "centreVertically"))
                settings.Add("centreVertically", true);
        }

        return settings;
    }

    public Dictionary<string, object> AddRowConfig(Dictionary<string, object> settings, GridRow row)
    {
        return settings;
    }
}
```
This adds a new setting to a cell that contains an image when an adjacent cell contains an rte. 

There are a few static helper methods we use to make working with AutomaticGridConfigs easier.

| Property    			    	| Purpose 			                                  |
| ----------------------------- | ----------------------------------------------------|
| CurrentIndex          		| Finds the index of the current cell in all cells    |
| OtherAreas              		| Gets all other cells in the row except the current  |
| Settings               		| Safe wato to get settings from rows and cells       |
| HasEditor               		| Does the current cell use the editor                |
| IsEditor               		| Is this sell item of editor type                    |
| HasContentType (DTGE)     	| Does the current cell item use the content type     |
| IsContentType (DTGE)        	| Is this cell items of content type                  |