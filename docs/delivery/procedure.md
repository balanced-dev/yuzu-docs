# Site Creation Procedure

After creating numerous sites using Yuzu we have developed a general procedure we use to get the best results. As we have the power to quickly create lots of document types and data types it's very easy to make mistakes and create entities that are not used or orphan entities that cause issues in model building. Using the following process we have found that we can create clean, easy to understand integrations that minimize any cleanup afterwards. 

Overall planning is the key, understanding how you want the site to integrate before making a move will save time later.

+ Generate all viewmodels, include these in the project and rebuild
+ Ignore any viewmodel or properties are not being stored in the CMS. Examples of these include breadcrumbs, navigation or anything that uses external data.  
+ Define if global referenced data   
    - Generate any parent document types
    - Add global setting for each viewmodel
+ Add any groups from `Map All Groups`
+ Run `Map All Viewmodels`
+ Generate Umbraco Models, include models in the project and rebuild
+ Create and attach all Manual Mappings  
+ Create the master and page templates and assign them to the correct document types
+ Add blank node structure in the content section
+ Import content for each page