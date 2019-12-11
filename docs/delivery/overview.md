# Overview

The delivery side of Yuzu involves integrating releases from UI definition to deliver a site. Because delivery developers are further abstracting from the details of the UI working against an interface then the complexity of their role is greatly reduced. They don't need to worry about integrating any of the markup details, only on mapping model data into the viewmodel definitions.

At runtime the template created in definition is rendered server-side by delivery and sent to the client. 

## Granularity of blocks

The definition application of Yuzu is a pattern library and, as a delivery developer, it's important to remember that any block can be rendered from anywhere within that library. It doesn't matter if it's the main page or a form element they all treated in the same way. 

We use this feature all the time to easily render ajax partials for our sites. When combined with progressive frameworks like Stimulus we can easily create simple ajax interactions. It may seem old fashioned but sometimes having the choice to work in multiple different ways gives you the greatest control whilst being maintainable and simple to implement.