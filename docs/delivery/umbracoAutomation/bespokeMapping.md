# Bespoke mapping

We have discovered that it's naive to think that direct mapping between the UI blocks and CMS content definition will be enough for anything but the simplest of sites. Bespoke mapping elevates Yuzu Import's potential use by giving the Delivery developer the tools to mould the schema as it is parsed into create the CMS schema. 

There are two way that mapping can be moulded.

## Name and property type changes

Through the viewmodel mapping dashboards the name of document type, their property names and types call all be changed. These values are saved into the config file.

## Grouped 

When creating viewmodel schemas sometimes it makes sense to split down similar properties into sub objects. By default in Yuzu Import these sub objects would be added as a new child document type link to the parent using a nested content datatype. By adding these sub blocks into Umbraco groups (tabs) we can reduce the number of document types and data types that are created and make it easier for the content author to understand the scope of the subBlock data.

![alt text](/images/bespoke_mapping_grouped_.jpg "Grouped bespoke mapping")

### Discovering and adding all possible groups

Parent and child subBlock relationships that can be grouped always follow a very specific set of conditions

- a one to one relationship between the parent and child
- the parent and child relationship only happens once in the system

We have created a process that hunts through all the inter block relationships and discovers all possible groups in a site and can add all these groups from at once. When building a site we found it complicated to discover these groups, this process takes makes groups easy to discover and manage and we would recommend using this process rather than adding groups manually.

### Nested groups

It is possible to have a chain of group relationships that can collapse down into multiple groups on a root document type. For example, with three subblock relationships

- `vmPage_Home` with a property `Header` of type `vmBlock_SiteHeader`
- `vmBlock_SiteHeader` with a property of `Nav` of type `vmBlock_SiteNav`

When grouped this it will form the following document type

Document type `Home`
Group `Header` with the properties from `SiteHeader`
Group `Nav` with the properties from `Nav`

?> Nesting groups in this way is only supported up to 2 levels deep as above

## Global content

Content that is shared between UI blocks should be stored globally as a external content resource that can be used anywhere in the site. Where the content from different UI blocks isn't exactly the same the properties are matched on properties names. 

![alt text](/images/bespoke_mapping_global_.jpg "Grouped bespoke mapping")

## Config file

Yuzu Import uses direct mapping at its default mode, any changes are stored in a json config file that is added to the root of the web application (location changeable in config). This file stores only the changes from the default mapping and is easy for a developer to understand and if need edit manually. 

It is possible to commit this file in source control and use it to generate the same setup on another developers setup with one caveat. When adding global items, the parent content id my not be present, successful import may require manually changing these values. 

