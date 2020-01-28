# Getting Started

The quickest way to get Yuzu up and running is to run our quick-start setup. We will make assumptions about the install that you can tweak to your own preferences as you gain confidence. As we'll see later on Yuzu is made up of many modules that can be swapped out or removed completey. This quickstart installs all the standard and automation modules, especially in the Umbraco implementation.

## Definition (Frontend)

## Umbraco Delivery (Backend)

In this section we'll assume that you have some experience of Visual Studio, installing nuget packages and Umbraco. 

### Visual Studio

In Visual Studio create a new solution that has 2 projects both using using .NET framework 4.72

- **Web project** - ASP.NET Web Application (.Net Framework) 
- **Core project** - Class Library (.Net framework)

For the web application project install this nuget package

```
npm Install-Package YuzuDelivery.Umbraco.QuickStart.Web
```

For the class library project install this nuget package

```
npm Install-Package YuzuDelivery.Umbraco.QuickStart.Core
```
?> **Make sure that the web project references the core project**!

### Umbraco

Build the project and run Debugging > Start without debugging to run the site, then install Umbraco without a starter kit.

### Models builders

Install the visual studio extensions Umbraco ModelsBuilder Custom Tool and Yuzu Delivery ViewModel Generator. 

Both of these extensions are configured from Tools > Options (Yuzu and Umbraco) and require a login to the Umbraco website created earlier.

That's it! 

A new implementation of Yuzu Delivery for Umbraco is now installed and ready for use!

### Integration

As Yuzu sites are defined by the frontend this new site won't do anything until definition push a new release for integration. 

Here is a list of our full working examples

[Lambda (with workshop)](https://github.com/balanced-dev/yuzu-example-lambda) - A single page site example for Yuzu using a theme called Lambda. There is a workshop available on the workshop branch.   

[Logistics](https://github.com/balanced-dev/yuzu-example-logistics) - A multiple page site example with bespoke mapping for Yuzu using a theme called Logistics.

[One School](https://github.com/balanced-dev/yuzu-example-oneschool) - A multiple page site example with bespoke mapping for Yuzu using a theme called One School. Includes multiple grid types, four forms, bespoke mapping, stimulus js integration for ajax rendered partials and vue integration for more complect progressive js.with   