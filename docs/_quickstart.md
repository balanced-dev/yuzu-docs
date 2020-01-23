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

As Yuzu sites are defined by the frontend this new site won't do anything until they create a realse for us to integrate. 

We have a pre-packaged release from our Lambda workshop available for you to download and integrate.

