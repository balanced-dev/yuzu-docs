# Umbraco Core

This package has no configuration. It is used to bootstrap the CMS agnostic core package into Umbraco dependency injection container.

It also contains mappings for Link and Image viewmodels to the Umbraco default objects.

## Umbraco Import Config

Ignore the DataImage and DataLink from IgnoreViewmodels.

## ToModel

When building Umbraco sites working with teh model builder it is common to end up with single standard Umbraco nodes objects (IPublishedContent and IPublishedElement). It can be difficult to safely convert these into strongly typed object. We have created an extension method safely makes this conversion.

``` c#
content.ToModel<HomePage>();
content.ToElement<CTA>();
```

