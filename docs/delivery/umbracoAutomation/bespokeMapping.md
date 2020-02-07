# Bespoke mapping

As we have discovered. It's naive to think that direct mapping between the UI blocks and CMS content definition will be enough for anything but the simplest of sites. Bespoke mapping elevates Yuzu Import's potential use by giving the Delivery developer the tools to mold the schema as it is parsed into create the CMS schema. 

There are three way that mapping can be changed.

## Name and property type changes

Through the viewmodel mapping dashboards the name of document type, their property names and types call all be changed. These values are saved into the config file.

## Grouped 

When creating viewmodel schemas sometimes it makes sense to split down similar properties into sub objects. By default in Yuzu Import these sub objects would be added as a new child document type link to the parent using a nested content datatype. By adding these sub blocks into Umbraco groups (tabs) we can reduce the number of document types and data types that are created and make it easier for the content author to understand the scope of the sub clock data.

![alt text](/images/bespoke_mapping_grouped_.jpg "Grouped bespoke mapping")

Manual mapping it needed to map the grouped viewmodel to the Umbraco group. Here's an example of how this is done.

``` c# 

//push the parent umbraco model into the subobject viewmodel 
CreateMap<AboutUs, vmBlock_AboutUs>()
    .ForMember(x => x.Video, opt => opt.MapFrom(y => y));

RecognizePrefixes("Video");
CreateMap<AboutUs, vmSub_AboutUsVideo>();

```

## Global content

Content that is shared between UI blocks should be stored globally as a external content resource that can be used anywhere in the site. Where the content from different UI blocks isn't exactly the same the properties are matched on properties names. 

![alt text](/images/bespoke_mapping_global_.jpg "Grouped bespoke mapping")

## Config file

Yuzu Import uses direct mapping at its default mode, any changes are stored in a json config file that is add to the root of the web application (location changeable in config). This file stores only the changes from the default mapping and is easy for a developer to understand and if need edit manually. 

Currently it is possible to commit this file in source control and use it to generate the same setup on another developers setup with one caveat. When adding global items, the parent content id my be present, successful import may require manually changing these values. 

