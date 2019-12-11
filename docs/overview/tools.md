# Tools

on top of this simple interface we have created a collection of CMS agnostic tools that make the production of Yuzu sites easier. 

## Definition

- template overlay containing
    - Pattern library - allows the definition developer to easily move between blocks that are organised into different types and areas.  
    - State management - State data changes can be made in real time and saved to the solution within the template overlay so that definition developers can see the effects that state has on the design.

```
npm install yuzu-definition-ui
```
[docs]() - 
[source](https://github.com/balanced-dev/yuzu-definition-api)

- CLI - a simple command line interface to add new blocks, rename blocks or add new states.  

```
npm install -g yuzu-definition-cli
```
[docs]() - 
[source](https://github.com/balanced-dev/yuzu-definition-cli)

## Delivery

- schema to viewmodel generation 
    - converts json schema files (released from Definition) into to C# Poco viewmodels. 
    - when used together with yuzu-delivery-import, assigns automapper attributes to all viewmodels.

Search in Visual Studio Extensions under 'yuzu delivery viewmodel generator' in VS 2019/7 extensions or 

[vs marketplace](https://marketplace.visualstudio.com/items?itemName=BalancedDev.yuzudeliveryviewmodelgenerator) - [docs]() - 
[source](https://github.com/balanced-dev/yuzudelivery.viewmodelgenerator)
