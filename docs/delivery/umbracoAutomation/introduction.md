# Umbraco Automation

We have created a commercial package called Yuzu Delivery Import that handles all aspects of Umbraco Automation in Yuzu. It is split into the following sections

- Viewmodel mapping : an overview of all the generated viewmodels and their inter relationships mapped to document type definitions.
- Schema creation : processes for creating document types and data types from generated viewmodel definitions.
- Content creation : using page states and images from the definition pattern library to create content in Umbraco.
- Bespoke mapping : Sitting in from of the generated viewmodels, this section configures mappings to route data into grouped or grouped document type definitions to prevent deplication and reduce complexity of CMS schema.

This package is licenced on a subscription per developer basis as part of the Yuzu Developer package. 

Yuzu Delivery Import is found in the **Settings** section of Umbraco under the **Yuzu Import** dashboard.

## Configuration

Setup

| Property    			    	 | Purpose 			                 			             |
| -------------------------------| ----------------------------------------------------------|
| IsActive		                 | Turns off Yuzu Import*                                    |                  
| DocumentTypeAssemblies         | Define assembly to find Umbraco models                    |                 
| ViewModelQualifiedTypeName	 |                                                           |                        
| UmbracoModelsQualifiedTypeName |                                                           |                   
| DataTypeFolder		         | Umbraco data type folder that new data types are added to |
| DataLocations                  | Location of json data states for content import           |
| ImageLocations                 | Location of images for content import                     |
| CustomConfigFileLocation       | Location of the custom mapping config file                | 

