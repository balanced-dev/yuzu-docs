# Manual Mapping

The simplest mapping solution is to manually create instances of viewmodels, but we quickly found that this became too verbose, hard to maintain and we had difficultly reusing mappings.

``` c#
var output = new vmBlock_Contact
{
    FirstName = Model.FirstName,
    LastName = Model.LastName
    ... etc
}
```

Manual mapping this way leads to replication, especially when we are mapping between properties that have the same or similar property names. Any changes that are released from definition require the delivery developer to understand what has changed and update the code correctly. 

One solution is to use automatic mapping e.g. Automapper, we use this package in Yuzu (it can be swapped out for any other solution). By creating parallel lines between our Models and Viewmodels we can reduce the need for manual mapping and automate the mapping process as much as possible. This makes for a very lean solution that contains little or no code. 

With automapping, when definition add a property to the viewmodel schema, delivery only need create the new property in the CMS and, as long as the property names match, then the new content will be filled from the CMS. No need for any extra code. Of course, this is very dependant upon creating parallel lines (property names that match) between the definition and delivery sides of the application. We have automated this process using our Import package.

But there are always occasions where automatic mapping is not possible due to complexity or integrating an external data source.

### Yuzu Mapping Api

We see a website as a cloud of interconnecting UI blocks that are combined to create the pages of the site. Each of these UI blocks has at least one corresponding viewmodel defining its dynamicity. In turn each of these viewmodels can be mapped to CMS objects that supply the content. Most content will be filled from the CMS source. 

Our mapping API allows the developer to replace, amend or ignore this mapping, before rendering of any viewmodel or viewmodel property at any point in this cloud of UI blocks. The Yuzu mapping API simplifies mapping definitions into single lines of code per mapping.

All mapping definitions are created in objects that inherit from *YuzuMappingConfig*, they are added using extension methods on the ManualMaps property. 

``` c#
public class ManualMapping : YuzuMappingConfig
{
    public Test()
    {
        ManualMaps.AddPropertyReplace<SiteHeaderNavMemberResolver, vmBlock_SiteHeader>(x => x.Nav);
    }
}
```

Above assigns the resolver `SiteHeaderNavMemberResolver` to the `SiteHeader` block's Nav property. When the `SiteHeader` viewmodel is hyrdated, for the property `Nav`, instead of mapping from the CMS this resolver is used instead. This gives us fine control over which content is displayed for the site at any point. 

Below is the Member resolver from the above example.

``` c#
public class SiteHeaderNavMemberResolver : IYuzuPropertyReplaceResolver<Template, vmBlock_SiteNav>
{
    public vmBlock_SiteNav Resolve(Template source, UmbracoMappingContext context)
    {
        
    }
}
```

Implementing the `IYuzuPropertyReplaceResolver` interface that requires two generic properties of the Source (`Template` DocumentType) and Destination Member Type (`vmBlock_SiteNav` viewmodel). The `Resolve` method returns the viewmodel, we are free to define how the CMS source is used to create the viewmodel. 

In the `Resolve` method a context object parameter is passed that contains the following data.

## Mapping Context

| Property    			    	| Details 			                                  |
| ----------------------------- | ----------------------------------------------------|
| Model          		        | The in context root CMS object                      |
| Html              		    | The current HtmlHelper                              |
| HttpContext               	| The current HttpContext                             |
| Items                    		| Any other items passed from the root view           |

A new item is passed into the mapping context as below

``` c#
@(Html.RenderYuzu<vmBlock_SiteHeader>(Model, false, 
    new Dictionary<string, object>() { 
        { "item", "test" } 
    }))
```

### Resolvers, Convertors and Factories

Manual mapping can be assigned to Viewmodels or Viewmodels properties in three different ways

- Replace : overrides the default mapping and passes the CMS source. Ties the resolver to the CMS source type.
- After : allows the Viewmodel data to be changed after is has been mapped but before rendering.
- Factory : plain creation of the Viewmodel without any reference to the CMS. Can be reused anywhere where the destination viewmodel is used. 

There are 6 interfaces for Viewmodels and their properties

| Interface    			    	| Generic properties 			|
| ----------------------------- | ------------------------------|
| IYuzuPropertyReplaceResolver  | Source, Dest Member Type      |
| IYuzuPropertyAfterResolver    | Source, Dest Member Type      |
| IYuzuTypeConvertor            | Source, Dest                  |
| IYuzuTypeAfterConvertor       | Source, Dest                  |
| IYuzuTypeFactory              | Dest                          |

### Dependency injection

All of these interfaces are automatically added to the IOC container at startup making dependency injection available.

``` c#
public class SiteHeaderNavMemberResolver : IYuzuPropertyReplaceResolver<Template, vmBlock_SiteNav>
{
    private readonly IMapper mapper;

    public SiteHeaderNavMemberResolver(IMapper mapper)
    {
        this.mapper = mapper;
    }

    public vmBlock_SiteNav Resolve(Template source, UmbracoMappingContext context)
    {

    }
}
```

### Import Plugin Integration

Using the Import CMS plugin makes it easier to create, assign and visualize these resolvers.  

## Automapper Profiles

Sometimes we have more complex mapping needs, for this this we can fall back on the full Automapper API using profiles. However, the process becomes a little more manual, e.g we have to manage which maps are ignore from automapping.

``` c#

    public class CommentsProfile : Profile
    {
        public CommentsProfile(IYuzuDeliveryImportConfiguration config)
        {
            config.IgnoreUmbracoModelsForAutomap.Add<Comments>();

            CreateMap<Comments, vmBlock_Comments>()
                .ForMember(x => x.Endpoints, opt => opt.MapFrom<CommentsEndpointResolver>());
        }
    }

```

### Dependency injection

All Automapper profiles, resolvers and converters are added to the IOC container at startup. 

``` c#
public class ContactFormResolver : IValueResolver<IPublishedContent, vmPage_SectionGridPage, vmBlock_Contact>
{
    private readonly IMapper mapper;
    private readonly IPublishedContentQuery contentQuery;

    public ContactFormResolver(IMapper mapper, IPublishedContentQuery contentQuery)
    {
        this.mapper = mapper;
        this.contentQuery = contentQuery;
    }

    public vmBlock_Contact Resolve(IPublishedContent source, vmPage_SectionGridPage destination, vmBlock_Contact destMember, ResolutionContext context)
    {
        var template = contentQuery.ContentAtRoot().Select(x => x.Descendant<Template>()).FirstOrDefault();
        return mapper.Map<vmBlock_Contact>(template.Contact);
    }
}
```

### Resolution Context

As part of the IValueResolver interface a ResolutionContext object is included. This object includes context items that are passed from the parent mapping context in the same way as items in the Yuzu mapping API. 

``` c#
public class LogoutFormResolver : IValueResolver<AccountNav, vmBlock_AccountNav, string>
{
    public string Resolve(AccountNav source, vmBlock_AccountNav destination, string destMember, ResolutionContext context)
    {
        var Html = context.Items["HtmlHelper"] as HtmlHelper;

        return Html.Partial("Logout").ToString();
    }
}
```

By passing the MVC HtmlHelper service in as a mapping context item we can render a partial into an HTML string to be injected into the block. We use this when ASP.net MVC needs to add extra elements to the form, which it does quite a lot!

We also use this method for passing in-context Model data from the page into the mapping resolver.
