# Getting Started

The quickest way to get Yuzu up and running is to run our quick-start setup. We will make assumptions about the install that you can tweak to your own preferences as you gain confidence. As we'll see later on, Yuzu is made up of many modules that can be swapped out or removed completely. This quickstart installs all the standard and automation modules, especially in the Umbraco implementation.

---

# Definition (Frontend)
We've tried to make getting up and running with your definition solution as quick and easy as possible: rather than manually having to create your directory structure and populate it manually with a basic, standardised setup yourself, we've automated this scaffolding in our [Yuzu Definition CLI](definition/cli).

1.  If you haven't done so already, install the CLI

```
npm install -g yuzu-definition-cli
```

2.  Open up your terminal in the root directory of your project
3.  Run the `yuzudef create "<Project name here>"` command
4.  Navigate to the newly created "definition.src" folder in your terminal
5.  Run the `npm install` command
6.  After the packages are installed, use the `npm run serve` or `gulp ui` command to build & run the solution

---

# Umbraco Delivery (Backend)
In this section we'll assume that you have some experience of Visual Studio, installing nuget packages and Umbraco. 

## .Net framework

### Visual Studio

There are 2 ways of setting up Yuzu in Visual Studio; as a standalone or two separate projects (web and core)

#### Standalone

In Visual Studio create a new ASP.NET Web Application (.Net Framework) solution using using .NET framework 4.72

```
pm> Install-Package YuzuDelivery.Umbraco.QuickStart
```

This will install a Yuzu directory in the project that contains the startup config. Manual mapping profiles can also be added here. 

#### Separated

In Visual Studio create a new solution that has 2 projects both using using .NET framework 4.72

- **Web project** - ASP.NET Web Application (.Net Framework) 
- **Core project** - Class Library (.Net framework)

For the web application project install this nuget package

```
pm> Install-Package YuzuDelivery.Umbraco.QuickStart.Web
```

For the class library project install this nuget package

```
pm> Install-Package YuzuDelivery.Umbraco.QuickStart.Core
```
?> **Make sure that the web project references the core project**!

## .Net 5 and above
For .Net 5 and above you will need to install the project templates.
```
dotnet new --install YuzuDelivery.Umbraco.Templates
```
There are 2 ways of setting up Yuzu; as a standalone or two separate projects (web and core)

#### Standalone

In Visual Studio create a new ASP.NET Web Application (.Net Framework) solution using using .NET 5 or higher

```
dotnet new yuzu-delivery
```

#### Separated

In Visual Studio create a new solution that has 2 projects both using using .NET 5 or higher
```
dotnet new yuzu-delivery-core
```
```
dotnet new yuzu-delivery-web
```

### Umbraco

Build the project and run Debugging > Start without debugging to run the site, then install Umbraco **without a starter kit**.

### Viewmodels builders and Import

All Yuzu tools are found in the User Interface Integration group of the Settings section in Umbraco.

That's it! 

A new implementation of Yuzu Delivery for Umbraco is now installed and ready for use!

### Integration

As Yuzu sites are defined by the frontend this new site won't do anything until definition [push a new release for integration](/delivery/overview?id=release-from-definition). To show you how it works we have created a standalone example that uses the Lambda definition from the Lambda example below.

```
pm> Install-Package YuzuDelivery.Umbraco.QuickStart.Example
```

### Setup process

1. Install Umbraco without a starter kit.
2. In Umbraco, go to Settings > Yuzu Viewmodels builder > Generate models
3. In Visual Studio add AppData/Viewmodels folder (show all files) to the solution and rebuild
4. In Umbraco, go to Yuzu Integrate and add all groups using `Map All Groups`
5. Click `Map All Viewmodels`
6. Go to settings > Models builder and generate models
7. In Visual Studio add AppData/Models to the solution and rebuild
8. In Umbraco, add 2 new templates, one called master and a child of master called home
9. Assign the home template to the home document type and allow this document type node at root 
10. Add a new content root node of Home
11. Go to Yuzu Import and import content on the Home node 

<div class="video-embed">
    <iframe src="https://www.youtube.com/embed/neigJuG2gp8?modestbranding=1&rel=0" width="720" height="405" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

### Further examples

Here is a list of our full working examples

[Lambda](https://github.com/balanced-dev/yuzu-example-lambda) - A single page site example for Yuzu using a theme called Lambda.  

[Logistics](https://github.com/balanced-dev/yuzu-example-logistics) - A multiple page site example with bespoke mapping for Yuzu using a theme called Logistics.

[One School](https://github.com/balanced-dev/yuzu-example-oneschool) - A multiple page site example with bespoke mapping for Yuzu using a theme called One School. Includes multiple grid types, four forms, bespoke mapping, stimulus js integration for ajax rendered partials and vue integration for more complex progressive js.
