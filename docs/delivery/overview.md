# Overview

Delivery in Yuzu is all about integrating releases from UI definition to deliver a site. Because delivery developers are abstracted from the details of the UI, only working against an interface, then the complexity of the role is greatly reduced. No need to worry about integrating any of the markup, only on mapping the model data into the viewmodel definitions.

On startup all the blocks and pages are pre-compiled into memory.

At runtime definition created templates are rendered server-side by delivery and sent to the client. 

## Granularity of blocks

The definition side of Yuzu is a pattern library and, as a delivery developer, it's important to realise that *any* block from the pattern library can be used when designed how the site is delivered. It doesn't matter if it's the main page or a form element they all behave in the same way. It is...

**Inversion of control for the UI**

We use this feature all the time to easily render ajax partials for our sites. When combined with progressive frameworks like Stimulus we can easily create simple ajax interactions. It may seem old fashioned but sometimes having the choice to work in multiple different ways gives the greatest control whilst being maintainable and easy to implement.