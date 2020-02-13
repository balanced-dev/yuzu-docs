# A Simple Solution

At its heart Yuzu is a simple solution to the complex problem of integration that is always caused by the separation of definition and delivery. We have created ancillary services and tools for Yuzu that make the process easier and pave the way to automation.

Yuzu is actually two separate production environments, one for each of definition and delivery. The basic rules and processes for each are outlined below.

---

#### Definition

- UI is split into discrete UI components called blocks
- Each block is self contained
- Each block contains a minimum of template, style, schema and data examples
- Block schemas describe the block dynamicity and act as a interface so that it can be rendered without any inner understanding
- Block data, schema and templates can reference any other block schema, data and templates
- Block references between schema, data and template needn't be in sync, a schema can reference a sub schema but the data can be inline
- Previews are created for each block on change
- UI definition is released to Delivery as a collection of template and schema files

    ```
    npm install yuzu-definition-core
    ```
    [docs](/definition/overview.md) - 
    [source](https://github.com/balanced-dev/yuzu-definition-core)

---

#### Delivery

- CMS agnostic
- Templates are loaded and precompiled
- Template schemas are used to generate viewmodels
- Model data is (auto)mapped to viewmodel objects
- Template files are rendered natively at runtime by a delivery solution using viewmodels hydrated from model data

    ```
    PM> Install-Package YuzuDelivery.Core
    ```
    [docs](/delivery/overview.md) - 
    [source](https://github.com/balanced-dev/yuzu-definition-core)

In conclusion this solution allows UI Definition developers to work in isolation creating self-contained, interlinking blocks whilst generating a managed interface that is synonymous to the DNA of the content led site.

Delivery developers also work in isolation mapping model data to the managed interface and rendering content when required at runtime. 
