# A Simple Solution

at its heart Yuzu is a simple solution to the complex problem of integration caused by the separation of frontend and backend developers. There are many different cogs to Yuzu but most of these are tools we have created to make the process of creating sites the Yuzu way as painless as possible.

The first main differece is that there are two separate production environments for UI definition and delivery. The base structures for each are outlined below.

#### Definition

- UI is split in descrete UI components called blocks
- each block is self contained
- each block contains a minumum of template, style, schema and data examples
- block schemas describe all the data posibilities and acts as a interface to the block so that it can be rendered without any inner understanding of the template
- block schema, data and templates can reference any other block schema, data and templates
- block references between schema, data and template needn't be in sync, a schema can reference a sub schema but the data can be inline
- previews are created for each block on change
- UI definition is released to Delivery as a collection of template and schema files

NPM package - source [yuzu-definition-core](https://github.com/balanced-dev/yuzu-definition-core)

#### Delivery

- templates are loaded and precompiled
- template schemas are used to define viewmodels
- model data is mapped to viewmodel objects
- template files are rendered natively at runtime by the delivery solution using hydrated viewmodela

CMS agnostic Nuget module - source [yuzudelivery-core](https://github.com/balanced-dev/yuzudelivery.core)  

In conclusion this solution allows UI Definition developers to work in isolation creating self-contained, interlinking blocks whilst generating a managed interface that is synonymous to the DNA of the content led site.

Delivery developers also work in isolation mapping model data to the managed interface and rendering content when required at runtime. 
