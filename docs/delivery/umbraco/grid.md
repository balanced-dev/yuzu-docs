# Umbraco Grids

This package brings together Umbraco Grid, [Doc Type Grid Editor](https://our.umbraco.com/packages/backoffice-extensions/doc-type-grid-editor/), [SkyBrud Umbraco GridData](https://github.com/skybrud/Skybrud.Umbraco.GridData) and Yuzu to create a powerful out of the box, config ready grid solution. It's the quickest and simplest way that we have found to implement the grid in Umbraco. 

Hooking into the Yuzu pattern library makes it easy for definition developers to add a selection of blocks that content editors can add to grids with live previews within the Umbraco back office. 

All mapping is routed through our GridItemMapping service that utilizes data structures specific to the grid. This service can be overridden in the container to implement your own mapping instance.

Coupled with our Umbraco Import package automates the creation of Umbraco Grid data types, including the creation of items in the Doc Type Grid Editor config file. We can also import grid config settings defined within the pattern library into the Grid property editor definition.

## Data Structures

There are two data structures for grids, vmBlock_DataRows and vmBlock_DataGrid (rows only and grids with rows and columns). Both structures can include config objects for both rows and columns, they are optional.

### Grid Items

On startup all block Viewmodels that can be directly mapped to a Umbraco Models (matched on name) are added as grid items using a generic class called `DefaultGridItem`. These generic implementations are used in the Yuzu grid builder to map Umbraco Models to the Viewmodels and fill the grid.

This `DefaultGridItem` class has three method

- `IsValid` : tests on the document type alias in the grid
- `CreateVM` : hydrates the viewmodel from the model using Automapper. Also used to generate the Umbraco back office preview. 
- `Apply` : Used by the grid builder in production to get the data from the grid and run the CreateVM above

There are many situations where the direct nature of generic class is not suitable. To add bespoke logic it is possible to either inherit this class and override its methods or implement your own class that implements the IGridItem interface. When doing so, it's important to follow the pattern above where Apply calls the CreateVm method, previews in the back-office will then reflect the production site.

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
        if (content != null && content.Content)
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

The above code uses a data repository to get teacher profiles instead of mapping Umbraco model data directly.

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

Because Yuzu is just a collections of blocks working with the grid in the back office is easier. For each cell added we have swapped out the rendering engine of DTGE to render the Yuzu template for the viewmodel using the preview data from the grid. A very simple and elegant solution.

To add styling we create a backoffice stylesheet during the definition dist process. This stylesheet is created from core and blocks styles only. In the DTGE preview renderer above we apply the css styles

``` css
 all: initial; 
 pointer-events: none; 
``` 
 
 to the preview wrapper div. This removes all Umbraco styles and prevents any pointer event from the block from affecting interaction. 

## Grid manipulation

Occasionally we have found that we need an extra level of control in changing how a setting or config item are applied to a cell or row in the grid. Below are a few examples of how we can use Yuzu to make these changes using the IAutomaticGridConfig interface. 

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
| Settings               		| Safe to get settings from rows and cells            |
| HasEditor               		| Does the current cell use the editor                |
| IsEditor               		| Is this sell item of editor type                    |
| HasContentType (DTGE)     	| Does the current cell item use the content type     |
| IsContentType (DTGE)        	| Is this cell items of content type                  |

AutomaticGridConfig is just one of the ways that we manipulate viewmodel data as it moves through the system. By understanding where we can make these changes and creating external services to implement only the required change then we can easily create complex or specific updates to the ViewModel data whilst upholding Single Responsibility.

## Skybrud.Umbraco.GridData and DocTypeGridEditor

In this package we use Skybrud.Umbraco.GridData to easily convert the json data created by the Grid Umbraco property into a strongly typed model. 

We have included part of an unreleased add-on package for GridData called Skybrud.Umbraco.GridData.Dtge. It converts data saved by DocTypeGridEditor in the grid into a strongly typed object.

These strongly types models can then be mapped to the viemodels as normal.

``` c#
using Skybrud.Umbraco.GridData.Dtge;

((GridControl)control).GetValue<GridControlDtgeValue>();
```
