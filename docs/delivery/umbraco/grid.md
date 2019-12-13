# Umbraco Grids

This package brings together Umbraco Grid, [Doc Type Grid Editor](https://our.umbraco.com/packages/backoffice-extensions/doc-type-grid-editor/), [SkyBrud Umbraco GridData](https://github.com/skybrud/Skybrud.Umbraco.GridData) and Yuzu to create a powerful out of the box, config ready grid solution. It the quickest and simplest way that we have found to implement the grid in Umbraco. 

Hooking into the Yuzu pattern library makes it easy for definition developers to add a selection of blocks available to content editors, including live preview within the Umbraco back office. 

All mapping happens through our grid item mapping service that pulls together a collection of grid aware block. This service can be overridden in the contains to implement your own mapping instance.

Couple with out Umbraco Imports package we can automate the generation of Grid document types, including the creation of Doc Type Grid Editor config. We've even included config setting defined within the pattern library that are automatically copied over and implemented in Grid editor config.

## Config

There is no config for this package.

## Mapping

There are two standard data structures for grid, vmBlock_DataGridRows and vmBlock_DataGridRowsColumns. The first is for row only grid and the second is for grids with rows and columns. Both rows and columns can include config object, but they are not mandatory.

### Grids

We have created some mapping wrapper classes that make it easier to work with grids in Yuzu. Which one used is dependant on how the other properties in the same Viewmodel are mapped.

When only adding grid mapping to a viewmodel the command is very simple

```c#
cfg.AddGridWithRows<SectionGridPage, vmPage_SectionGridPage>(src => src.Grid, dest => dest.Grid);
```

When manual mapping on other viewmodel properties is required then it gets a little more complex.

```c#
CreateMap<HomePage, vmPage_HomePage>()
    .ForMember(x => x.Content, opt => opt.MapFrom<GridRowConvertor<HomePage, vmPage_HomePage, vmBlock_GridConfig>, GridDataModel>(y => y.Content));
```

### Grid Items

## Grid setting manipulation