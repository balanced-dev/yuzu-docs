# Definition release structure

Overview of the files sent from definition when the dist function is run. This is all the information that the CMS needs to create and run the site. 

Yuzu adds two directories to the CMS directory, 

- _client : production releases of css and js and static support files (fonts, images etc)
- _templates :  everything needed by Yuzu to generate the ViewModels render blocks. Yuzu Umbraco Import also use these files to create the CMS site definition and import content.

This templates directory is further split into directories

- data : json files of page state state used to import content
- html : rendered examples of all pages and blocks
- paths : schema meta data; relationships maps between blocks including anyOf definitions, ofType details
- schema : all schema files, separated into pages and blocks
- src : all  handlebars files, separated into pages and blocks 

