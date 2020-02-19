# Umbraco Core

Contains logic that bootstraps the CMS agnostic core package into Umbraco dependency injection container.

It also contains mappings for Link and Image viewmodels to the Umbraco default objects.

## Default Core Config

Creates a default instance of the core Yuzu configuration settings that are integrated and specific to Umbraco. 

### Web config settings

Adds the following web config app settings to the site

| Property    			    	| Purpose 			                        | 
| ----------------------------- | ------------------------------------------|
| YuzuPages                    	| File location of the pages files          |
| YuzuPartials              	| File location of the partials files       |
| YuzuSchemaMeta            	| File location of the schema meta files    |

### Caching

Uses Umbraco internal Runtime caches to cache compiled templates and html fragments

### Viewmodel and CMS models

Add all Viewmodels from the provided single assembly

Adds all objects that have a base type of PublishedElementModel and PublishedContentModel from provided single assembly

## Default Viewmodel builder config

Creates a default instance of the core Yuzu Viewmodel Builder configuration settings that are integrated and specific to Umbraco. 

### Web config settings

Adds the following web config app settings to the site

| Property    			    	            | Purpose 			                                     |                      |
| ----------------------------------------- | -------------------------------------------------------|----------------------|
| YuzuViewmodelBuilderActive   	            | Yes / No                                               |                      |
| YuzuViewmodelBuilderAcceptUnsafeDirectory | Can the viewmodels be out of the web root              |                      |
| YuzuViewmodelBuilderDirectory            	| the directory to add viewmodels (relative and absolute | /App_Data/ViewModels |

## Umbraco Import Config

Ignore the data image and link from IgnoreViewmodels and ExcludeViewmodelsAtGeneration.

## ToModel

When building Umbraco sites working with teh model builder it is common to end up with single standard Umbraco nodes objects (IPublishedContent and IPublishedElement). It can be difficult to safely convert these into strongly typed object. We have created an extension method safely makes this conversion.

``` c#
content.ToModel<HomePage>();
content.ToElement<CTA>();
```

