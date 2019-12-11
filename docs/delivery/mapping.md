# Mapping strategies

This is the act of taking the model data from one or more sources and using this data to hydrate the viewmodel. On first look it appears to be a simple process but we have found that deciding which mapping strategy to use in any particular circumstance takes a little skill and experience. With a little practice this nuanced approach allows many levels of mapping resulting in code that is minimal, descriptive and maintainable. 

The simplest solution would be to manually create instances of the viewmodels, but we quickly found that this became verbose and hard to maintain.  

``` c#
var output = new vmBlock_Contact
{
    FirstName = Model.FirstName,
    LastName = Model.LastName
}
```

Manuaul mapping leads to a lot of replication, you are just mapping between properties that have the same or similar property names. Any changes that are released from definition will require the delivery developer to update the code to implement that changes. 

The obvious solution is to use an automapping solution, we use Automapper. By creating parallel lines between our Models and Viewmodels we can reduce the need for manual mapping and automate the mapping process as much as possible. This makes for a very lean solution that contains little or no code. 

Working this way, when a new property is added by definition all the delivery need do is create the new property in the CMS and as long as the property names match then the new property will be mapped. No need for any extra code. Of course, this is very dependant upon creating parallel lines (property names that match) betweem the definition and delivery sides of the application. We automate this process using our Umbraco import package.

But there are always occasions where automatic mapping is not possible. This is where we have found that use two types of mapping strategies in addition manual mapping. 

## Property Member mapping 

The best of both worlds, here we create an automap for most of the properties but then apply manual maps to specific properties

``` c#
CreateMap<SiteHeaderNavLink, vmSub_SiteHeaderNavLink>()
    .ForMember(x => x.IsActive, opt => opt.MapFrom(z => true));
```

This will automap every property apart from IsActive where the output will always be mapped as true. 

The above example is OK for simple mappings. But as the code resolving the output gets more complex then we use a class that implements the IValueResolver to contain the code.

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

As part of the IValueResolver interface we are passed a ResolutionContext object. This object includes items that are passed through in context of the map and allows us to do some remarkable things. 

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

By passing the MVC HtmlHelper service in as a mapping context item we can render a partial into an HTML string to be injected into block. We use this when ASP.net MVC needs to add extra elements to the form, which it does quite a lot!

We also use this method for passing in context model data from the page into the mapping resolver.

### Dependancy injection

Unfortunately we've not worked out how we can use additional services from an IOC container with these resolvers. As a result we end up doing this. 

``` c#
public class ContactFormResolver : IValueResolver<IPublishedContent, vmPage_SectionGridPage, vmBlock_Contact>
{
    private readonly IMapper mapper;
    private readonly IPublishedContentQuery contentQuery;

    public ContactFormResolver()
    {
        mapper = mapper = DependencyResolver.Current.GetService<IMapper>();
        contentQuery = DependencyResolver.Current.GetService<IPublishedContentQuery>();
    }

    public vmBlock_Contact Resolve(IPublishedContent source, vmPage_SectionGridPage destination, vmBlock_Contact destMember, ResolutionContext context)
    {
        var template = contentQuery.ContentAtRoot().Select(x => x.Descendant<Template>()).FirstOrDefault();
        return mapper.Map<vmBlock_Contact>(template.Contact, opts => opts.Items["HtmlHelper"] = context.Items["HtmlHelper"]);
    }
}
```

## Bespoke mapping

This is where we use a class that implements ITypeConvertor to define how an object is created from Model to Viewmodel. 

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

You may ask if there is a difference between straight manual mapping and this method, yes, there is a big difference. In the example above this means that whenever automapper comes across a source of IPublishedContent and a destination of a link viewmodel then it will always apply this converter and we have only defined this once.

This greatly reduces the code required and follows closely the ideas of [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle).

By using Automapper we create a collection of mapping definitions that are in a container. It doesn't matter how they link together, or if that changes over time, it will always just work. In the same way that IOC containers work. It is very efficient. 