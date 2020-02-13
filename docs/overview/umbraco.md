# Umbraco Integration

Although the core of Yuzu is delivery application (CMS) agnostic we have released a series of modules that integrate with the Umbraco CMS. They show how a CMS can be integrated at the core level and with specific editors that CMS's uses to define common UI structures.

Below is an introduction to each module that integrates with Umbraco

## Core umbraco

umbraco specific wrapper for yuzu.definition.core. Bootstraps Yuzu by registering components in the IOC container and adding hbs helpers. Also contains the default viewmodels for Links and Images and their mapping. 

```
PM> Install-Package YuzuDelivery.Umbraco.Blocks
```
[docs](/delivery/umbraco/core) - 
[source](https://github.com/balanced-dev/yuzudelivery.umbraco.blocks)

## Grids

integration and mapping tools for working with the Grid Layout property type in Umbraco

```
PM> Install-Package YuzuDelivery.Umbraco.Grids
```
[docs](/delivery/umbraco/grid) - 
[source](https://github.com/balanced-dev/yuzudelivery.umbraco.grids)

## Forms 

integration and mapping tools for working with Umbraco Forms.

```
PM> Install-Package YuzuDelivery.Umbraco.Forms
```
[docs](/delivery/umbraco/forms) - 
[source](https://github.com/balanced-dev/yuzudelivery.umbraco.forms)

## Members 

integration and controllers for working with Umbraco Membership, extending core Umbraco functionality. Easily add simple Login, Forgotten Password, Register, Change Password and Change name/email

```
PM> Install-Package YuzuDelivery.Umbraco.Members
```
[docs](/delivery/umbraco/members) - 
[source](https://github.com/balanced-dev/yuzudelivery.umbraco.members)