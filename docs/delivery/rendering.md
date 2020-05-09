# Rendering Template

In Yuzu there are two ways of rendering templates; in a razor view or output from a controller.

The option we use the most is rendering in a razor view. This is because it's easy to implement, understand and doesn't require a rebuild after a change. We have created a couple of Html helpers for this purpose. 

## RenderFETemplate Html Helpers

``` c#
Html.RenderYuzu<vmPage_Home>(Model)
```

The above method uses AutoMapper to hydrate the Viewmodel vmPage_Home from the Model data context and renders the definition template called home.

The ShowJson property will display the json used to at the bottom of the page. Useful for debugging when something isn't working.

``` c#
Html.RenderYuzu<vmPage_Home>(true)
```

Passing items into the mapping context

``` c#
@(Html.RenderYuzu<vmPage_Home>(Model, false, 
    new Dictionary<string, object>() { 
        { "item", "test" } 
    }))
```

This is a different method that we use when we need more granular control.

``` c#
Html.RenderYuzu(new RenderSettings()
    {
        Template = "home",
        Data = () => {
            return new
            vmPage_HomePage() { };
        },
        ShowJson = false,
    })
```

Both methods also come with some basic caching using the properties CacheName and CacheExpiry. This will cache the html output after it has been rendered using the cache rendering configured in setup. 

## Controller

We have a service based on the interface IYuzuDefinitionTemplates that can be injected into a controller. It has the same members as above for rendering using automapper and manually. 

``` c#
return yuzuTemplates.Render<vmBlock_CommentList>(
new RenderSettings()
{
    Template = "parCommentList",
    MapFrom = comments
}, 
null);
```