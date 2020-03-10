# Data/States
Data is a big part of the definition side of Yuzu projects: while it's true that you could get by with only creating schemas, markup, styling and Javascript, creating states is often a critical component of the Yuzu process.

They enable the front-end developers to work in complete isolation, without the need for the delivery side, to check the flexibility of the UI with different data situations. It allows the Handlebars partials to be rigorously tested and styling decisions to be made using extreme data, as well as allowing initial content data to be lifted from the designs and used to hydrate the CMS when being integrated.

---

# What is a state?
A state is quite simply a JSON file for a specific block/page which validates against its schema. It usually uses a specific/extreme set of property values to test one/multiple of the block/page's:

1.  **Handlebars partial**:<br>
    By changing values from truthy to falsy and vice versa, all avenues of the Handlebars conditions outlined in the block/page partial can be explored using multiple states. The includes boolean properties and the absence/presence of data (e.g. empty vs populated array)

2.  **Styling (SCSS/CSS)**:<br> 
    Different amounts and/or combinations of data conditions can cause designs/layouts to break. For example with a string value you could test the extremes over several states: a long sentence, a very long word, a little amount of text etc. to see how it responds across all screen sizes. You can also test layouts: for example if there are two bullet point lists which are equal height in the design, you would purposefully umbalance them by assigning more items to one list than the other in a new state. Designers often use perfect or duplicated dummy data (e.g. Lorem Ipsum fragments) when creating their designs. Authors or users however are not likely to do the same and so your solution should be flexible enough to cope.

3.  **Javascript**:<br>
    States can be useful for when settings are being passed to your scripts via data properties in the markup: for example passing latitude and longitude values into a Google map. It can also be useful when mimicking backend responses: e.g. rendered HTML from the delivery side in AJAX requests ([see Javascript section's API explanation](definition/javascript?id=API)).

States are critical during the creation of the definition side of Yuzu projects: it allows the developer to test all the situations that a block/page has to respond to and as such effectively helps to document the project. They can be range from as basic as a single property, to complex nested objects and arrays.

?> Before you read on, you should already be comfortable with basic states, if not [see the Markup section's JSON files](definition/markup) for examples.

Our state names follow a pattern: the default state which will contain the most "typical" data simply is named after the block e.g. `parExample.json`. Any additional states simply add an underscore followed by a kebab-cased name describing its purpose succinctly e.g. `parExample_no-name.json` (notice its similarity to the BEM naming structure). 

---

# Sub-blocks ($ref)
As explained in the markup section, partials are able to contain other partials; in other words blocks can contain sub-blocks and pages can contain blocks. 

It follows, therefore, that a block's state must also contain any data its sublock(s) require and a page the relevant data for its block(s).

For example, say we are working on a simple profile block (`parProfile`), which uses a social links block (`parSocialLinks`). We have a social links block state (`parSocialLinks_one-link`) which we want to use in our profile block state with a long name (`parProfile_long-name`).

We'll have to copy the social link data into the parent profile block like so:

<!-- tabs:start -->
#### ** parSocialLinks_one-link.json **
```json
{
    "links": [
        {
            "logo": {
                "src: "/images/logos/facebook.svg",
                "alt": "Facebook Logo"
            },
            "link": {
                "title": "",
                "href": "www.facebook.com",
                "label": "View Facebook profile"
            }
        }
    ]
}
```
#### ** parProfile_long-name.json **
```json
{
    "name": "Apu Nahasapeemapetilon",
    "social": {
        "links": [
            {
                "logo": {
                    "src: "/images/logos/facebook.svg",
                    "alt": "Facebook Logo"
                },
                "link": {
                    "title": "",
                    "href": "www.facebook.com",
                    "label": "View Facebook profile"
                }
            }
        ]
    }
}
```
#### ** parProfile.hbs **
```handlebars
<div class="profile">
    <h1>{{name}}</h1>
    {{> parSocialLinks social}}
</div>
```
<!-- tabs:end -->


In this example, this doesn't seem too bad: what's the harm in quickly duplicating one state and adding it to another wrapping block's state. It allows for states to be viewed in isolation doesn't it?

Well yes, but this quickly gets out of hand: imagine a page with multiple blocks: that alone could result in JSON files hundreds of lines long to create a single page state it if included complex underlying blocks. Even worse, those blocks could contain sub-blocks and even those sub-blocks could contain nested blocks of their own. What happens if multiple blocks/pages relied on the same block/sub-block state when testing different aspects of that block/page?

All this unnecessary duplication of data can be avoided by using `$ref`. It allows for the referencing of other blocks' states to form page/wrapping block states. It effectively does the exact same thing when rendering the markup, but is obviously far cleaner and allows for smaller, more understandable states.

The above example would become:
`parProfile_long-name.json`:
```json
{
    "name": "Apu Nahasapeemapetilon",
    "social": {
        "$ref": "/parSocialLinks_one-link"
    }
}
```

---

# Inline vs sub-block state
Although everything said so far may seem to be indicating otherwise, using `$ref` to inject other block states into the current context is not always correct (although most of the time it will be).

You do not want to fall into the trap of, when trying to cleanup your ancestor states (like pages or wrapping blocks), ending up cluttering your solution in the form of too many states for sub-blocks.

An extreme example would be, say, a `parLink` block, used for all the links in the site. Say you wanted the text to be true to the designs everywhere throughout the site. Should you be going and creating 20+ states in `parLink` purely for this reason? No, it means that the states actually testing the block are effectively being diluted by these new states (which are only used for wording changes). It goes away from the self-documenting nature of states.

Inline data (nesting sub-block states within ancestor states), without using `$ref` therefore has its place, but should mainly be limited to simple blocks, which will not bloat the JSON too much. Another thing to consider is that inline data does detract from the self-contained aspect of blocks: now a block's state can be spread throughout your solution which needs to be considered if updating the block's schema (e.g. removing/renaming/restructuring properties). Common sense and an appreciation for balance is required.

A good example in our projects would be form element blocks: while they are blocked and thus have a global schema and markup (as it can be used as a sub-block within many other blocks), their states are virtually all inline within their parent/ancestor blocks. If it was not done this way each form element could end up with a lot of unnecessary JSON files, with the only difference being label/placeholder changes.