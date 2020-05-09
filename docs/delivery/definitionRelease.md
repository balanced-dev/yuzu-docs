# Definition release structure

An overview of the files that are sent from definition when the dist function is run. This is all the information that the CMS needs to create the integration and run the site. 

Yuzu adds two directories to the CMS directory, 

- _client : production releases of css and js, static support files (fonts, images etc) and pattern library preview files.
- _templates :  everything needed by Yuzu to render pages from blocks. Yuzu Umbraco Import also use these files to create the CMS site definition and import content.

The client directory is further split into directories

- styles : the css output files
- js : the js output files
- sassdoc : sass documentation files
- html : html previews from definition
- image : any images that the preview use

additionally any ancillary directories; font etc will also be sent over

This templates directory is further split into directories

- data : json files of page state state used to import content
- paths : schema meta data; relationships maps between blocks including anyOf definitions, ofType details
- schema : all schema files, separated into layouts, pages, blocks
- src : all handlebars files, separated into layouts, pages and blocks 

