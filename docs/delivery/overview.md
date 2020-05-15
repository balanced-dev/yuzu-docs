# Overview

Delivery in Yuzu is *integrating releases* from UI definition to deliver a site. Because delivery developers are abstracted from the details of the UI, only working against the UI interface the complexity of the task is greatly reduced. No need to worry about integrating  markup, only on mapping the model data to viewmodel definitions.

On startup all the blocks and pages are pre-compiled into memory.

At runtime the templates created by definition are rendered server-side by delivery and sent to the client. 

## Release from definition

When the Yuzu package is first installed it contains no templates or data from definition. As a delivery developer your first action may be to push a release to delivery. 

```
npm run dist
```

## Granularity of blocks

The definition side of Yuzu is a pattern library and, as a delivery developer, it's important to realise that *any* block from the pattern library can be used when designing how the site is delivered. It doesn't matter if it's the main page or a form element they all behave in the same way. It is...

**Inversion of control for the UI**

We use this feature all the time to easily render ajax partials for our sites. When combined with progressive frameworks like Vue or Stimulus we can easily create simple ajax interactions.