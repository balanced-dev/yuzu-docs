# Site Creation Procedure

+ Generate all viewmodels, include in project and rebuild
+ Decide which viewmodel properties are not stored in the CMS. Examples include breadcrumbs, navigation or anything that is filled using external data.  
+ Define any global referenced data   
--* Generate any parent document types
--* Add global setting for each viewmodel
+ Add any groups using the automated tool (tools > add globals)
+ Run Map All Viewmodels
+ Generate Umbraco Models, include in project and rebuild
+ Create and attach all Manual Mappings  
+ Create the master template and templates for each page
+ Import content for each page