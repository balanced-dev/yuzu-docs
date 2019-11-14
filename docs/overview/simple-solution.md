# A Simple Solution

at its heart Yuzu is a simple solution to the complex problem of integration caused by the separation of frontend and backend developers and how the UI is integrated. There are many different working parts to Yuzu but mostly these are just tools we have created to make the process of creating sites in this way as painless as possible.

We start with two production environments; UI definition and delivery and we have split the structures for each below

#### Definition

- UI is split in descrete UI components called blocks
- each block is self contained
- each block contains a minumum of template, style, schema and data examples
- block schemas describe all the data posibilities and acts as a interface to the block so that it can be rendered without any inner understanding of the template
- block schema, data and templates can reference any other block schema, data and templates
- block refrences betweem schema, data and template don't need to be in sync, a schema can reference a sub schema but the data can be inline
- previews are created for each block
- UI definition is released to Delivery as a collection of template and schema files

source [yuzu-definition-core](https://github.com/balanced-dev/yuzu-definition-core)

#### Delivery

- template files are loaded and precompiled
- template schemas are converted into the native viewodel objects
- model data is (auto)mapped to generated viewmodel objects
- template files are rendered natively at runtime by the delivery solution using hydrated viewmodela

CMS agnostic module - source [yuzudelivery-core](https://github.com/balanced-dev/yuzudelivery.core)  
Umbraco bootstrap - source [yuzudelivery.umbraco.blocks](https://github.com/balanced-dev/yuzudelivery.umbraco.blocks)  
[Viewmodel Generation in Vs marketplace](https://marketplace.visualstudio.com/items?itemName=BalancedDev.yuzudeliveryviewmodelgenerator) - source [yuzudelivery](https://github.com/balanced-dev/yuzudelivery.viewmodelgenerator)

In conclusion this solution allows UI Definition developers to work in isolation creating self-contained, inlinking blocks whilst generating a managed interface that is synonymous to the DNA of the content led site.

Delivery developers also work in isolation mapping model data to the managed interface and rendering content when required at runtime. 
