# Manual Mapping

Whilst it is possible to create manual mapping in code, the Umbraco Import plugin is the perfect place to define where and how manual maps are applied to the system. The developer can add manual mapping to any viewmodel or property and it decides which mapping types are available given the context of the current item. 

### Code generation

The interfaces that define all the types of available manual maps are not that complex but it is difficult to see their context and understand what various parts are required to set up a manual map. We introduced manual map code generation to take the guess work out of this process. 

- choose the type of manual mapping and click generate code
- the process uses a template to create a code stub with the correct interface generics and method signature
- the process writes the code into the `Yuzu/Mappings` file directory. This directory is defined in the `YuzuImportManualMappingDirectory` app setting in the web config.
- the developer adds the code stub into visual studio, writes the manual mapping code and builds the solution
- in the Umbraco plugin the newly added manual map is available in valid types drop down and can be applied to the system.

?> Applying manual maps restarts the Umbraco process

?> There is no need to add any manual mapping definitions in code when applying manual maps in this way.  

