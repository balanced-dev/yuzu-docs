# Tools

on top of this simple interface we have created a collection of CMS agnostic tools that make the production of Yuzu sites easier. 

### Definition

- template preview overlay
    - Pattern library - an overlay to the template preview that allows the definitions developer to easily move between blocks that are organised into different types and areas.  
    - State management - State data changes can be made in real time and saved to the solution within the template overlay so that definition developers can see the effects that state will have on the design.

```
npm install yuzu-definition-ui
```
[docs]() - 
[source](https://github.com/balanced-dev/yuzu-definition-api)

- CLI - a simple command line interface to simply add new blocks, rename blocks or add new states.  

```
npm install -g yuzu-definition-cli
```
[docs]() - 
[source](https://github.com/balanced-dev/yuzu-definition-cli)

### Delivery

- schema to viewmodel generation - converts json schema files created for a release from Definition into to C# Poco viewmodels. When used with yuzu-delivery-import, automaps the mapping process by assigning automapper attributes to all viewmodels.

Search for yuzu delivery viewmodel generator in VS 2019/7 extensions or [VS marketplace](https://marketplace.visualstudio.com/items?itemName=BalancedDev.yuzudeliveryviewmodelgenerator)  
[docs]() - 
[source](https://github.com/balanced-dev/yuzudelivery.viewmodelgenerator)
