# Mapping strategies

Taking model data from one or more sources and hydrating viewmodels look to be a simple process, but, we have found that deciding which mapping strategy to use in any particular circumstance takes a little skill and experience. With practice this nuanced approach allows several types of mapping creating code that is minimal, descriptive, maintainable and testable. 

The simplest solution would be to manually create instances of the viewmodels, but we quickly found that this became verbose and hard to maintain.  

``` c#
var output = new vmBlock_Contact
{
    FirstName = Model.FirstName,
    LastName = Model.LastName
    ... etc
}
```

Manual mapping leads to lots of replication, you are just mapping between properties that have the same or similar property names. Any changes that are released from definition requires the delivery developer to know what has changed and update the code correctly. 

The solution was to use an auto mapping solution, we use Automapper. By creating parallel lines between our Models and Viewmodels we can reduce the need for manual mapping and automate the mapping process as much as possible. This makes for a very lean solution that contains little or no code. 

With automapping, when a new property is added by definition all the delivery need do is create the new property in the CMS and as long as the property names match then the new property will be mapped. No need for any extra code. Of course, this is very dependant upon creating parallel lines (property names that match) between the definition and delivery sides of the application. We have automated this process using our Umbraco Import package.

But there are always occasions where automatic mapping is not possible. This is where we have found that use two types of mapping strategies in addition manual mapping. 

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

## Profiles

Profiles in Automapper allow us to create modules of mapping definitions. 

``` c#

    public class GridProfile : Profile
    {
        public GridProfile(IYuzuDeliveryImportConfiguration config)
        {
            config.IgnoreUmbracoModelsForAutomap.Add<Home>();
            this.AddGridWithRows<Home, vmPage_Home>(x => x.Body, y => y.Body);
        }
    }

```

## Property Member mapping 

The best of both worlds, we create an Automap for the majority of properties but then apply manual mapping to specific properties

``` c#
CreateMap<SiteHeaderNavLink, vmSub_SiteHeaderNavLink>()
    .ForMember(source => source.IsActive, opt => opt.MapFrom(dest => true));
```

This will Automap every property apart from IsActive where the output will always be mapped as true. 

The above example is OK for simple mappings. But as the code required to generate the output gets more complex we use a class that implements the IValueResolver. This class contains just the code for this one mapping logic.

``` c#
public class CommentsEndpointResolver : IValueResolver<Comments, vmBlock_Comments, vmSub_CommentsEndpoints>
{
    public vmSub_CommentsEndpoints Resolve(Comments source, vmBlock_Comments destination, vmSub_CommentsEndpoints destMember, ResolutionContext context)
    {
        return new vmSub_CommentsEndpoints()
        {
            CommentFormUrl = "?alttemplate=commentsform"
        };
    }
}
```

And implementing in the map as follows

``` c#
CreateMap<Comments, vmBlock_Comments>()
    .ForMember(x => x.Endpoints, opt => opt.MapFrom<CommentsEndpointResolver>());
```

### Resolution Context

As part of the IValueResolver interface a ResolutionContext object is included. This object includes context items that are passed from the parent mapping context allowing us to do some interesting things. 

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

## Bespoke mapping

This is where we use a class that implements ITypeConvertor to manually create the Viewmodel using one or more source of data. 

``` c#
public class LinkIPublishedContentConvertor : ITypeConverter<IPublishedContent, vmBlock_DataLink>
{
    public vmBlock_DataLink Convert(IPublishedContent link, vmBlock_DataLink destination, ResolutionContext context)
    {
        if (link != null)
        {
            return new vmBlock_DataLink()
            {
                Title = link.Name,
                Label = link.Name,
                Href = link.Url,
            };
        }
        else
            return null;
    }
}
```

And this would be implemented in the map by

``` c#
CreateMap<IPublishedContent, vmBlock_DataLink>()
        .ConvertUsing<LinkIPublishedContentConvertor>();
```

You are right to ask if there is any difference between straight manual mapping and this method. Yes, there is a big difference. In the example shown, whenever Automapper comes across a source of IPublishedContent and a destination of a link viewmodel then it will always apply this converter and we have only defined this once.

This greatly reduces the code required and follows closely the ideas of [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).

By using Automapper we create a collection of mapping definitions that are in a container. It doesn't matter how they link together, or if that changes over time, it will always just work. In the same way that IOC containers work. It is very efficient. 