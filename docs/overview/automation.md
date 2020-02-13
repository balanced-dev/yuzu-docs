# Automation

When viewed from above the schema that defines the properties of and relationships between each block start to take the form of a map of the dynamicity of the site. For content-led sites this is the DNA of the site.

Because the core to Yuzu is this site DNA we can use it to automate the generation of different parts of the process and accelerate production. 

Currently we have two areas of automation

# Definition Scaffold 

we have created a schema short-hand that makes it easy to define the schema of a block. This text fragment can be stored as files or as Trello cards.  

A node js application takes these text fragments and parses it into a schema. From here the application generates
    - empty json state file with the correct properties
    - scss file with bem stubs for all the properties
    - template with mocks up HTML for all the properties
    - schema file

This scaffolds up most of the site so that, for each block, definition developers only need add the example data from wireframe / design, teak the template and apply styles.

Coming Soon

# Umbraco Import 

Automates the creation of Umbraco schema and content.

- an overview of viewmodels and how they map to umbraco document types and properties
- creates document types from viewmodel definitions
- creates a range of document type properties based on property types
- defines relationships between document types by creating nested content and child document types
- granular control of generation from document type property up to an entire site
- uses page states from definition to generate content in Umbraco
- bespoke mapping of viewmodels to global content for shared content
- bespoke mapping of viewmodels to grouped content to flatten document type structures

```
PM> Install-Package YuzuDelivery.Umbraco.Import
```
[docs]() - 