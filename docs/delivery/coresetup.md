# Core setup and config

The core of Yuzu delivery is a CMS agnostic library that contains all common logic.

```
PM> Install-Package YuzuDelivery.Core
```

Yuzu is initialized using a default configuration object as follows (not a complete example)

``` c#
Yuzu.Initialize(new YuzuConfiguration()
{
    ViewModelAssemblies = new Assembly[] { localAssembly, gridAssembly },
    TemplateLocations = new List<ITemplateLocation>()
    {
        new TemplateLocation()
        {
            Name = "Pages",
            Path = Server.MapPath(pagesLocation),
            Schema = Server.MapPath(pagesLocation.Replace("src", "schema")),
            RegisterAllAsPartials = false,
            SearchSubDirectories = false
        }
    },
}
```

When we implement Yuzu for a CMS we create a default instance of this config file that we use on each site we create.

## Configuration Properties

Properties in bold are required

| Property    			    	| Purpose 			                        | Default Value             |
| ----------------------------- | ------------------------------------------|---------------------------|
| ViewModelAssemblies       	| Assemblies that contain viewmodels        |                           |
| MappingAssemblies            	| Assemblies for mapping strategies         |                           |
| CMSModels 	            	| Set to the CMS generated models           |                           |
| TemplateLocations			    | Locations for hbs templates               |                           |
| SchemaMetaLocations       	| Locations for schema meta files           |                           |
| GetTemplatesCache 		    | function to get cached complied templates |                           |
| SetTemplatesCache 	        | function to set cached compiled templates |                           |
| GetRenderedHtmlCache      	| function to get cached rendered content   |                           |
| SetRenderedHtmlCache	    	| function to set cached rendered content   |                           |


#### Templates locations

| Property    			    	| Purpose 			                                        |
| ----------------------------- | ----------------------------------------------------------|
| Name 			                |                                                           |
| Path 			                | The **absolute** path to the templates directory          |
| Schema        	            | The **absolute** path to the template schema directory    |
| RegisterAllAsPartials         | If these templates are also registered as partial         |
| SearchSubDirectories         	| Searches further sub directories for templates            |

#### Schema meta locations

| Property    			    	| Purpose 			                                        |
| ----------------------------- | ----------------------------------------------------------|
| Name 			                |                                                           |
| Path 			                | The **absolute** path to the schema meta directory        |

## Caching setup

The core setup to Yuzu allows you to specify how and where caching takes place. We have done this to give you full control of how this works and to make the core implementation CMS agnostic. 

### Compiled definition templates

On startup the Yuzu will reads all the templates from the specified locations, pre-compiles and stores them in a cache. This slightly affects boot-up time but significantly improves the speed of pages at runtime. 

The location and expiry length of that cache is up to you. 

### Html cache

As part of the rendering process we have a simple caching mechanism setup to cache rendered HTML. This allows you to define where and how that data is cached.

### Yuzu Constants

This is a store of common string we use throughout the framework

| Property    			    	| Purpose 			                        | Default Value             |
| BlockPrefix               	| Block prefix for generated viewmodels     | vmBlock_                  |
| SubPrefix                 	| SubBlock prefix for generated viewmodels  | vmSub_                    |
| PagePrefix                	| Page prefix for generated viewmodels      | vmPage_                   |
| BlockRefPrefix                | how blocks are defined in definition      | /par_                     |
| TemplateFileExtension       	| file extension used                       | .hbs                      |

We have added the ability to change the prefix values for viewmodel naming but we haven't fully implemented this feature. Currently, changing this will break everything!