# Markup

## Handlebars Basics
As previously discussed, we choose to use and recommend Handlebars for all our projects purely because of its speed and simplicity, both to implement, use and learn.

It also allows additional helpers to quickly and easily be added and used (however if you choose to add your own you'll need to replicate the functionality for server-side rendering).

The best place to start may be the [Handlebars Docs](https://handlebarsjs.com/ "Read official Handlebars documentation"), although they are not (in our opinion) the best and seem to overcomplicate/not fully explain its features, so this will be a quick get-started guide. You should rarely have any real reason to refer to anything but this in most cases, but the Handlebars Documentation is improving ([new draft docs at time of writing](https://handlebars-draft.knappi.org/guide/ "Read new draft Handlebars documentation")) and will dive deeper than this guide will.

?> If you want to follow along with the examples in the basic section of this guide yourself or just play around in Handlebars, [tryhandlebarsjs.com](http://tryhandlebarsjs.com/) is a fantastic resource

### Basic Expressions
Handlebars expressions are contents wrapped in double curly braces `{{}}` (apart from when not escaping HTML).

#### Render properties
To render a property from the current context:
``` json
{
    "name": "John Doe"
}
```

``` handlebars
<h1>{{name}}</h1>
```

``` html
<h1>John Doe</h1>
```

#### Render nested properties
To render a property from the current context, nested in an object:
``` json
{
    "person": {
        "name": "John Doe"
    }
}
```

``` handlebars
<h1>{{person.name}}</h1>
```

``` html
<h1>John Doe</h1>
```

#### Render property without HTML escaping
If you don't want Handlebars to escape a value, you must use triple braces `{{{}}}`. This is very useful for outputting values from RTEs (Rich Text Editors) as illustrated below (double vs triple braces).
``` json
{
    "personBio": "<p>A bit about me!</p>"
}
```

``` handlebars
<div>{{personBio}}</div>
vs
<div>{{{personBio}}}</div>
```

``` html
<div>&lt;p&gt;A bit about me!&lt;/p&gt;</div>
vs
<div><p>A bit about me!</p></div>
```

#### Escaping Handlebars expressions
If you don't want to run a Handlebars expression then either add a leading '\\'so:

``` handlebars
\{{escaped}}
```
or wrap it in a "raw" block:
``` handlebars
{{{{raw}}}}
  {{escaped}}
{{{{/raw}}}}
```

This is especially useful when using a Javascript framework such as Angular.js which also uses braces without them conflicting.

#### Comments
For if you want to document your templates:
``` handlebars
{{!-- This is a comment and will not be included in the HTML output --}}
```


## Handlebars Helpers

### Block helpers
?> Both the `#each` and `#with` helper change the scope/context for a section of the template

#### #each (iterator)
Mostly used to loop over arrays, but can also be used with objects.

```json
{
    "people": [
        {
            "name": "Yohn Royce"
        },
        {
            "name": "Rodrik Cassel"
        },
        {
            "name": "Eddison Tollett"
        }
    ],
    "places": [
        "White Harbour",
        "Deepwood Motte",
        "Barrowtown"
    ]
}
```

```handlebars
<h2>People:</h2>
<ul>
    {{#each people}}
        <li>
            {{name}}
        </li>
    {{/each}}
</ul>

<h2>Places:</h2>
<ul>
    {{#each places}}
        <li>
            {{this}}
        </li>
    {{/each}}
</ul>
```

```html
<h2>People:</h2>
<ul>
        <li>
            Yohn Royce
        </li>
        <li>
            Rodrik Cassel
        </li>
        <li>
            Eddison Tollett
        </li>
</ul>

<h2>Places:</h2>
<ul>
        <li>
            White Harbour
        </li>
        <li>
            Deepwood Motte
        </li>
        <li>
            Barrowtown
        </li>
</ul>
```
!> There are several built-in properties that you can use within an #each block, but for simplicity's sake we've omitted them from this section.
To learn more see the [advanced handlebars section](definition/markup?id=block-helpers-advanced)

#### #with (context)
Althought the `#with` helper shouldn't be needed in the majority of projects (properties shouldn't really be nested within more than 2 or 3 objects, unbroken by arrays, in most cases), it may come in handy with deeply nested properties, as it shifts the current context.

```json
{
    "person": {
        "names": {
            "first": "Anthony",
            "middle": "Edward",
            "last": "Stark"
        }
    }
}
```

```handlebars
{{!-- 
    Instead of:
    <p>{{person.names.first}} {{person.names.middle}} {{person.names.last}}</p>
--}}
{{#with person.names}}
    <p>{{first}} {{middle}} {{last}}</p>
{{/with}}
```

```html
    <p>Antony Edward Stark</p>
```

### Conditionals
Use #if, #unless, and else to conditionally render sections of your templates

#### #if
The #if helper renders its contents if the argument is truthy (not equal to false, undefined, null, "", 0, or []).
```json
{
    "boolVal": true,
    "stringVal": "Hello",
    "intVal": 99,
    "arrVal": ["value"]
}
```

```handlebars
{{#if boolVal}}
    <p>boolVal is truthy!</p>
{{/if}}
{{#if stringVal}}
    <p>stringVal is truthy!</p>
{{/if}}
{{#if intVal}}
    <p>intVal is truthy!</p>
{{/if}}
{{#if arrVal}}
    <p>arrVal is truthy!</p>
{{/if}}
```

```html
    <p>boolVal is truthy!</p>
    <p>stringVal is truthy!</p>
    <p>intVal is truthy!</p>
    <p>arrVal is truthy!</p>
```


#### #unless
The #unless helper renders its contents if the argument is falsy (equal to false, undefined, null, "", 0, or []), in other words the opposite of the #if helper.

```json
{
    "boolVal": false,
    "stringVal": "",
    "intVal": 0,
    "arrVal": []
}
```

```handlebars
{{#unless boolVal}}
    <p>boolVal is falsy!</p>
{{/unless}}
{{#unless stringVal}}
    <p>stringVal is falsy!</p>
{{/unless}}
{{#unless intVal}}
    <p>intVal is falsy!</p>
{{/unless}}
{{#unless arrVal}}
    <p>arrVal is falsy!</p>
{{/unless}}
```

```html
    <p>boolVal is falsy!</p>
    <p>stringVal is falsy!</p>
    <p>intVal is falsy!</p>
    <p>arrVal is falsy!</p>
```

#### else
When writing an `#if` or `#unless`, often you will want to output a section of your template if the condition evaluates to false, without writing another using the inverse helper (an `#unless` paired with an `#if` and vice versa). 

Handlebars deals with this case by providing `else` functionality, which can also be chained with the conditionals to create `else if`s or `else unless`es.

!> Elses are not limited to being paired with `#if` and `#unless`. Read the [block helpers advanced](definition/markup?id=block-helpers-advanced) section for more.


```json
{
    "trueVal": true,
    "falseVal": false
}
```

```handlebars
{{#if falseVal}}
    <p>falseVal = true</p>
{{else}}
    <p>falseVal = false</p>
{{/if}}

{{#unless trueVal}}
    <p>trueVal = false</p>
{{else}}
    <p>trueVal = true</p>
{{/unless}}

<h2>Validation message:
<p>
{{#if falseVal}}
    Error: falseVal does not equal false!
{{else unless trueVal}}
    Error: trueVal does not equal true!
{{else}}
    No errors!
{{/if}}
</p>
```

```html
    <p>falseVal = false</p>

    <p>trueVal = true</p>

<h2>Validation message:
<p>
    No errors!
</p>
```

#### Partials

## Advanced Handlebars

### Block helpers (advanced)

## Yuzu's Handlebars Helpers


## Layouts

## Best Practices
