# Viewmodel Generation

Generation of C# pocos uses the schema sent from definition during a release. The core logic can be run independently or integrated into a CMS as we have done with Umbraco.

Setup of the Viewmodel generation again is very similar to the way that Models Builder works for Umbraco in DataApp mode. 

- run the Viewmodels builder process in the application
- manually include the generated files into Visual Studio
- build the solution

## Config

| Property    			    	    | Purpose 			                        | Default Value             |
| ----------------------------------| ------------------------------------------|---------------------------|
| EnableViewmodelsBuilder      	    |                                           | false                     |
| GeneratedViewmodelsNamespace 	    | Namespace for the generated pocos         | YuzuDelivery.ViewModels   |
| GeneratedViewmodelsOutputFolder 	| Folder the viewmodels are generated in    |                           |
| AddNamespacesAtGeneration			| Namespaces add to generated file          |                           |
| ExcludeViewmodelsAtGeneration     | Doesn't generate these viewmodels         |                           |