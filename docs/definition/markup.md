# Markup
As previously discussed, we choose to use and recommend Handlebars for all our projects purely because of its speed and simplicity to implement, use and learn.

It also allows additional helpers to quickly and easily be added and used (however if you choose to add your own you'll need to replicate the functionality for server-side rendering and we will not be covering it in this documentation).

---

# Handlebars Basics
The best place to start may be the official [Handlebars Docs](https://handlebarsjs.com/ "Read official Handlebars documentation"), but they are not (in our opinion) the best and seem to overcomplicate/not fully explain its features, so this will be a quick get-started guide. 

You should rarely have any real reason to refer to anything but this documentation in most cases, but the Handlebars Documentation is improving ([new draft docs at time of writing](https://handlebars-draft.knappi.org/guide/ "Read new draft Handlebars documentation")) and will dive deeper than this guide will.

?> If you want to quickly try out the examples in this guide yourself or just play around in Handlebars, [tryhandlebarsjs.com](http://tryhandlebarsjs.com/) is a fantastic resource (just note that the Yuzu helpers obviously will not work).

## Basic Expressions
Handlebars expressions are contents wrapped in double curly braces `{{}}` (apart from when not escaping HTML, as double braces escape by default).

### Render properties
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

### Render nested properties
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

### Render property without HTML escaping
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

### Escaping Handlebars expressions
If you don't want to run a Handlebars expression then either add a leading '\\'so:

``` handlebars
\{{escaped}}
```
or wrap it in a "raw" block, which also prevents the processing of mustache blocks:
``` handlebars
{{{{raw}}}}
  {{escaped}}
{{{{/raw}}}}
```

This is especially useful when using a Javascript framework such as Angular.js which also uses braces without them conflicting.

### Comments
For if you want to document your templates:
``` handlebars
{{!-- This is a comment and will not be included in the HTML output --}}
```

---

# Handlebars Helpers

## Block helpers
?> Both the `#each` and `#with` helper change the scope/context for a section of the template

### #each (iterator)
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

{{!-- "this" is available in #each to access the current context --}}
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
?> There are several built-in properties that you can use within an `#each` block, but for simplicity's sake we've omitted them from this section.
To learn more about these properties, and to read about iterating over objects, read the [advanced handlebars section](definition/markup?id=block-helpers-advanced)

### #with (context)
Although we personally haven't had much need for it, the `#with` helper may be useful in some situations. It shifts the current context (similar to the `#each`).

The reason we rarely use it is that we find the way we structure data infrequently leads us to nest properties in more than 2 consecutive objects, uninterrupted by an array. An array will almost certainly lead to an each being employed to iterate over it, which already changes the context.

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

## Conditionals
Use `#if`, `#unless`, and `else` to conditionally render sections of your templates

### #if
The `#if` helper renders its contents if the argument is truthy (not equal to `false`, `undefined`, `null`, `""`, `0`, or `[]`).
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


### #unless
The `#unless` helper renders its contents if the argument is falsy (equal to `false`, `undefined`, `null`, `""`, `0`, or `[]`), in other words the opposite of the `#if` helper.

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
{{#unless undefinedVal}}
    <p>undefinedVal is falsy!</p>
{{/unless}}
```

```html
    <p>boolVal is falsy!</p>
    <p>stringVal is falsy!</p>
    <p>intVal is falsy!</p>
    <p>arrVal is falsy!</p>
    <p>undefinedVal is falsy!</p>
```

### else
When writing an `#if` or `#unless`, often you will want to output a section of your template if the condition evaluates to false, without writing another using the inverse helper (an `#unless` paired with an `#if` and vice versa). 

Handlebars deals with this case by providing `else` functionality, which can also be chained with the conditionals to create `else if` or `else unless`.

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

?> More complex conditions are possible with Yuzu's `#ifCond` helper (see the [Yuzu's handlebars helpers](definition/markup?id=ifCond) section for more).

## Partials

Using partials is a fundemental part of the way Yuzu works: it allows for templates to be used within other templates- in other words blocking!

Simply by passing the Handlebars partial name as the first parameter and a data context for as the second allows for the injection of blocks into layouts, pages and even other blocks!

In basic terms you pass a section of data and delegate its rendering to another block.

For example say you have 2 blocks `parPerson` and `parHeadshot` and you want to include `parHeadshot` within `parPerson`. The code would look something like this:

`parPerson.json`:
```json
{
    "name": "Henry Jones Jr.",
    "image": {
        "src": "https://upload.wikimedia.org/wikipedia/en/8/8e/Indiana_Jones_in_Raiders_of_the_Lost_Ark.jpg",
        "alt": "Archeologist at work"
    }
}
```

`parPerson.hbs`
```handlebars
<h1>{{name}}</h1>
{{> parHeadshot image }}
```

`parHeadshot.hbs`
```handlebars
<img src="{{src}}" alt="{{alt}}"/>
```

`html`
```html
<h1>Henry Jones Jr.</h1>
<img src="https://upload.wikimedia.org/wikipedia/en/8/8e/Indiana_Jones_in_Raiders_of_the_Lost_Ark.jpg" alt="Archeologist at work"/>
```

---

# Advanced Handlebars
You should by now have got to grips with the basics of Handlebars and are prepared to delve a little deeper, learning a bit more about rendering properties than in the basics- including values which Handlebars exposes to you in helpers.

## Rendering properties
### Ancestor properties
There will be situations where you will want to access values outside your current context within helpers which alter the context (`#each`, `#with` etc.).

This can be achieved by simply prefixing the expression with a `../` segment for every level you want to traverse up the tree from your current context.

For example:
```json
{
    "names": {
        "first": "Terry",
        "last": "Jeffords"
    },
    "links": [
        {
            "href": "www.madeuplink.com/photos",
            "label": "photos"
        },
        {
            "href": "www.madeuplink.com/certificates",
            "label": "certificates"
        },
        {
            "href": "www.madeuplink.com/history",
            "label": "history"
        }
    ],
    "personal": {
        "hobbies": [
            "eating yoghurt",
            "painting",
            "working out",
            "referring to himself in 3rd person"
        ]
    }
}
```

```handlebars
<nav>
    {{#each links}}
        <a href="{{href}}">
            View {{../names.first}} {{../names.last}}'s {{label}}
        </a>
    {{/each}}
</nav>
<h2>Hobbies</h2>
<ul>
{{#with personal}}
    {{#each hobbies}}
        <li>{{../../names.first}} loves {{this}}</li>
    {{/each}}
{{/with}}
</ul>
```

```html
<nav>
        <a href="www.madeuplink.com/photos">
            View Terry Jeffords's photos
        </a>
        <a href="www.madeuplink.com/certificates">
            View Terry Jeffords's certificates
        </a>
        <a href="www.madeuplink.com/history">
            View Terry Jeffords's history
        </a>
</nav>
<h2>Hobbies</h2>
<ul>
        <li>Terry loves eating yoghurt</li>
        <li>Terry loves painting</li>
        <li>Terry loves working out</li>
        <li>Terry loves referring to himself in 3rd person</li>
</ul>
```
!> When trying to access ancestor `@data` values ([discussed here](definition/markup?id=data-values) ) found within `#each` helpers. When nesting iterating over nested arrays/objects, you need to prefix the entire path with `@` e.g. `@../../index` NOT `../../@index` when accessing the `@data` values of outer loops.
### Literal segments
To use a property that is not a valid identifier, segment-literal notation can be used.

This allows for the rendering of specific array indexes and property identifiers that, while valid in JSON, may cause problems when Handlebars attempts to process it.

!> We rarely need to use segment-literal notation in our experience. The below example is an extreme example with horribly named properties. However the ability to target a specific index of an array can be very useful, as explained in the [advanced #each section](definition/markup?id=block-helpers-advanced) 

```json
{
    "availableClasses": [
        "test-class",
        "secondary-class"
    ],
    "elements": [
        {
            "#id": "id-1",
            "@elementTag": "span",
            "(contents)": "Hello"
        },
        {
            "#id": "id-2",
            "@elementTag": "p",
            "(contents)": "Goodbye"
        }
    ]    
}
```

```handlebars
<p>
    First available class:
    {{availableClasses.[0]}}
</p>

{{#each elements}}
    <{{[@elementTag]}} class="{{[#id]}}">
        {{[(contents)]}}
    </{{[@elementTag]}}>
{{/each}}
```

```html
<p>
    First available class:
    test-class
</p>

    <span class="id-1">
        Hello
    </span>
    <p class="id-2">
        Goodbye
    </p>

```

## Block helpers (advanced)
### Using else with block helpers
Although `else` is usually seen paired with an `#if` or `#unless` it is also possible to use it in conjunction with other helpers such as `#each`, to conditionally render a fallback to empty arrays.

```json
{
    "subjects": [],
    "student": {}
}
```
```handlebars
<h2>Your subjects:</h2>
<p>
{{#each subjects}}
    {{this}},
{{else}}
    No subjects found!
{{/each}}
</p>

{{#with student.image}}
    <img src="{{src}}" alt="{{alt}}">
{{else}}
    <p>No student image uploaded</p>
{{/with}}
```
```html
<h2>Your subjects:</h2>
<p>
    No subjects found!
</p>

    <p>No student image uploaded</p>
```

### Using else vs #if with an #each
Although using `else` within an `#each` is a clean way of conditionally rendering a section of your template for when your array is empty (e.g. a fallback message) it is not always an ideal choice.

For example, what if you wanted to conditionally render an element which wraps the `#each`? 

The prime example would be in a `<ul>` or `<ol>` which contains `<li>`s. Perhaps you don't want to render the list element at all if the array is empty and would prefer to just render a message instead. This is not possible using `else` within the `#each` approach outline above.

What we have found works is using an `#if` to check for the presence of the first element in the array (i.e. not empty) via segment-literal notation.

```json
{
    "toDo": []
}
```
```handlebars
{{#if toDo.[0]}}
    <ol>
        {{#each toDo}}
            <li>{{this}}</li>
        {{/each}}
    </ol>
{{else}}
    <p>Nothing is on your to-do list!</p>
{{/if}}
```
```html
    <p>Nothing is on your to-do list!</p>
```

### #each - iterating over objects
Although in our opinion this is rarely useful, the `#each` helper is capable of looping through objects, like so:
```json
{
    "obj": {
        "val1": "A",
        "val2": "B",
        "val3": "C"
    }
}
```
```handlebars
<ul>
{{#each obj}}
    <li>{{this}}</li>
{{/each}}
</ul>
```
```html
<ul>
    <li>A</li>
    <li>B</li>
    <li>C</li>
</ul>
```
?> Read on into the @data values section to learn how to render the object key

### @data values
Handlebars has plenty of little properties crammed into the `#each` helper which help solve common problems:
- `@index` - a zero-based index for the current step
- `@key` - the name of the key when looping over objects
- `@first` - boolean value, set to true on the first step of the `#each`
- `@last` - boolean value, set to true on the last step of the `#each`

Here's an example of them all used at once (not perfect, but gets the point across!):
```json
{
    "timesTables": {
        "1": [0,1,2,3,4,5,6,7,8,9,10],
        "2": [0,2,4,6,8,10,12,14,16,18,20],
        "3": [0,3,6,9,12,15,18,21,24,27,30]
    }
}
```
```handlebars
{{#each timesTables}}
    <h2>{{@key}} times table</h2>
    <ul>
        {{#each this}}
            {{!-- Ignore first item where both {{this}} && {{@index}} == 0  --}}
            {{#unless @first}}
                <li>
                    {{@../key}} x {{@index}} = {{this}}
                </li>
            {{/unless}}            
        {{/each}}
    </ul>
    {{!-- Draw divider between all tables --}}
    {{#unless @last}}
        <hr/>
    {{/unless}}
{{/each}}
```
```html
 <h2>1 times table</h2>
    <ul>
                <li>
                    1 x 1 = 1
                </li>
                <li>
                    1 x 2 = 2
                </li>
                <li>
                    1 x 3 = 3
                </li>
                <li>
                    1 x 4 = 4
                </li>
                <li>
                    1 x 5 = 5
                </li>
                <li>
                    1 x 6 = 6
                </li>
                <li>
                    1 x 7 = 7
                </li>
                <li>
                    1 x 8 = 8
                </li>
                <li>
                    1 x 9 = 9
                </li>
                <li>
                    1 x 10 = 10
                </li>
    </ul>
        <hr/>
    <h2>2 times table</h2>
    <ul>
                <li>
                    2 x 1 = 2
                </li>
                <li>
                    2 x 2 = 4
                </li>
                <li>
                    2 x 3 = 6
                </li>
                <li>
                    2 x 4 = 8
                </li>
                <li>
                    2 x 5 = 10
                </li>
                <li>
                    2 x 6 = 12
                </li>
                <li>
                    2 x 7 = 14
                </li>
                <li>
                    2 x 8 = 16
                </li>
                <li>
                    2 x 9 = 18
                </li>
                <li>
                    2 x 10 = 20
                </li>
    </ul>
        <hr/>
    <h2>3 times table</h2>
    <ul>
                <li>
                    3 x 1 = 3
                </li>
                <li>
                    3 x 2 = 6
                </li>
                <li>
                    3 x 3 = 9
                </li>
                <li>
                    3 x 4 = 12
                </li>
                <li>
                    3 x 5 = 15
                </li>
                <li>
                    3 x 6 = 18
                </li>
                <li>
                    3 x 7 = 21
                </li>
                <li>
                    3 x 8 = 24
                </li>
                <li>
                    3 x 9 = 27
                </li>
                <li>
                    3 x 10 = 30
                </li>
    </ul>

```

### Block parameters
If you thought that the example in the previous example was messy (@data values) then you'd be correct! You'd also be right in thinking there must be a better, more understandable and readable way of achieving the same output: Handlebars actually supports block parameters in `#each`. 

It allows for named references to be used anywhere within the helper, reducing the need for utilising `../` to gain access to `@data` values of outer loops. It can also be used to name "this", potentially making your code understandable.

For example:
- `#each {array-value-here} as |{context-name-here}|` allows you to name the context within the `#each`
- `#each {array-value-here} as |{context-name-here} {index-name-here}|` allows you to not only name the context and the index
- `#each {object-value-here} as |{context-name-here} {key-name-here}|` similarly allows the naming of the context and they key

Basically it allows aliases for the `@key`, `@index` and `this` values within `#each` and, so long as there are no naming clashes, removes the need for using `../` to access ancestor properties.

Using the [@data values](definition/markup?id=data-values) example, here is an improved handlebars file, achieving identical html output:
```handlebars
{{#each timesTables as |table tableNo|}}
    <h2>{{tableNo}} times table</h2>
    <ul>
        {{#each table as |product factor|}}
            {{!-- Ignore first item where both {{product}} && {{factor}} == 0  --}}
            {{#unless @first}}
                <li>
                    {{tableNo}} x {{factor}} = {{product}}
                </li>
            {{/unless}}            
        {{/each}}
    </ul>
    {{!-- Draw divider between all tables --}}
    {{#unless @last}}
        <hr/>
    {{/unless}}
{{/each}}
```

---

# Yuzu's Handlebars Helpers
Most markup in a project can be achieved without touching any of these helpers, which is what we aimed for. We strive to make Yuzu as accessible as possible to newcomers and that meant trying to lean on vanilla Handlebars.js as much as possible. Adding a whole raft of helpers just means there's more to learn and adds complexity to the project.

Repeating markup patterns? Add another block! Need a calculated value (e.g. an array count, a value in multiple formats like a date etc.) or you can't easily solve a condition using the helpers? Request it from the delivery (back-end) side of the project by adding it to the block's schema and JSON: e.g. `arrCount` integer for the array length, separate values for multiple formats, or a boolean flag for complex conditions.

That said, we have sprinkled some very basic helpers that add some quality of life functionality, solve some issues which are definition (front-end) specific and, in the case of `pictureSource` helper, reduce the amount of complexity/bloat within blocks.

## Yuzu '_' properties
Before we get into looking at the helpers, there are automatically added data properties attached to all blocks specifically when using Yuzu which you should be aware of.

These are purely appended so that they do not clutter the JSON/Schema files in a block as they are likely to be useful in several situations in larger projects. By not relying on the data contained within the block also has the added benefit of both standardising the property names and decoupling the definition and delivery sides of the projects. This is desirable as the properties we have added are to fix issues which are purely concerns of the front-end.

These properties are used in a select few of the following helpers and will be described further in their respective section.

| Property Name | Helper        | Type              | Purpose                                                                                                                       |
| ------------- | ------------- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------- |  
| `_ref`        | `dynPartial`  | `string`          | Contains block reference for the block e.g. `parLink`                                                                         |
| `_modifiers`  | `modPartial`  | `array[string]`   | An array of class names passed down to the block, from the parent Handlebars template, thereby allowing contextual styling    |

## #ifCond
This helper basically just adds a wider range of possibilites for conditional rendering. Where `#if` and `#unless` fall down is that you can only evaluate a singular value and not a condition.

It's limited to 3 parameters to form the most simple of condtions, `value-1`, `operator`, `value-2` in that order.

The values can be a variable or just a string, integer etc. while the operator is limited to a string representation of a collection of comparison/logical operators listed below:


| Operator  | Description 			    |
| --------- | ------------------------- | 
| `'==='`   | Equal to			        |
| `'!=='`   | Not equal to		        |
| `'<'`     | Less than			        |
| `'<='`    | Less than or equal to     |
| `'>'`     | Greater than		        |
| `'>='`    | Greater than or equal to  |
| `'&&'`    | And				        |
| `'\|\|'`  | Or    			        |

As with `#if` and `#unless` they can be paired with an `else` or even joined to create a `else ifCond`. It can even be mixed with `#if` and `#unless` and thus can create fairly complex chains of conditions:

```json
{
    "engineeringProblemCount": 4,
    "engineeringProblems": [
        {
            "isMoving": true,
            "shouldBeMoving": false
        },
        {
            "isMoving": false,
            "shouldBeMoving": false
        },
        {
            "isMoving": true,
            "shouldBeMoving": true
        },
        {
            "isMoving": false,
            "shouldBeMoving": true
        }
    ]
}
```
```handlebars
<p>
    {{#unless engineeringProblemCount}}
        No questions?!? Why did you bother me?
    {{else ifCond engineeringProblemCount '===' 1}}
        Just the one question? Ok!
    {{else ifCond engineeringProblemCount '<=' 3}}
        Ok, let's see...
    {{else ifCond engineeringProblemCount '>' 3}}
        Wow! A lot of questions!
    {{/unless}}
</p>
<ul>
    {{#each engineeringProblems}}
        <li>
            Solution to problem {{@index}}:
            {{#ifCond isMoving '&&' shouldBeMoving }}
                There's nothing to fix! It's working perfectly as it should be moving and it is!
            {{else ifCond isMoving '||' shouldBeMoving}}
                {{#if isMoving}}
                    Duct tape is your friend!
                {{else}}
                    Apply WD-40 generously.
                {{/if}}
            {{else}}
                What are you worried about? It's not meant to be moving and it's not.
            {{/ifCond}}
        </li>
    {{/each}}
</ul>
```
```html
<p>
        Wow! A lot of questions!
</p>
<ul>
        <li>
            Solution to problem 0:
                    Duct tape is your friend!
        </li>
        <li>
            Solution to problem 1:
                What are you worried about? It's not meant to be moving and it's not.
        </li>
        <li>
            Solution to problem 2:
                There's nothing to fix! It's working perfectly as it should be moving and it is!
        </li>
        <li>
            Solution to problem 3:
                    Apply WD-40 generously
        </li>
</ul>
```
`#ifCond`, although not perfect, can solve some more complex issues than purely using `#if` or `#unless` (namely when dealing with numerical values- `<`, `>`, etc. and string values `===`). It is flawed in the sense that you cannot invert boolean values in the condition (e.g. `{{#ifCond !value1 '&&' !value2}}`), it's also impossible to do complex conditions like `{{#ifCond value1 '&&' (value2 '||' value3)}}` etc. and have to resort to using nested conditions (as with the above example) to achieve the desired output.

Rarely would such a complex set of conditions be used in the way they were when looping over the `engineeringProblems` array- if this was a proper project, the logic should be moved over to the delivery (back-end) side of the project with a string property being passed instead of a series of boolean values. This would either contain what type of message should be rendered (by using a series of `#ifCond`, `else ifCond`s to effectively form a switch statement) or just pass the actual message itself.

In this way `#ifCond` is actually perfect- it does as much as it reasonably can within the constraints of Handlebars, while encouraging the more complex logic to be kept out of the the templates, thus making them easier to understand. It also allows the front-end team to concentrate on what actually matters: good markup, styling and Javascript interactions.

## dynPartial
To explain the need for `dynPartial` (dynamic partial) it is best to describe the problem we were experiencing.

When we were creating sites which used grids the same pattern kept emerging: we would effectively have to create a long switch statement (using a string parameter and `#ifCond` & `else ifCond`) with all the blocks available to be rendered in a rows/column. If there were many options, it could quickly bloat the Handlebars file making it difficult to maintain. 

For example:
```json
{
    "rows": [
        {
            "blockName": "image",
            "data": {
                ...
            }
        },
        {
            "blockName": "rte",
            "data": {
                ...
            }
        }
        {
            "blockName": "heading",
            "data": {
                ...
            }
        }
    ]
}
```
```handlebars
{{#each rows}}
    {{#ifCond blockName '===' 'image'}}
        {{> parImage data}}
    {{else ifCond blockName '===' 'rte'}}
        {{> parRichTextEditor data}}
    {{else ifCond blockName '===' 'heading'}}
        {{> parSectionHeading data}}
    {{/ifCond}}
{{/each}}
```
As you can imagine, templates (such as grids) where we needed to conditionally render blocks depending on which block was represented in the data was a fairly common occurance and so we created dynamic partial helper to solve this, where a parameter could be a string variable.

This allowed for the previous example to become:
```json
{
    "rows": [
        {
            "blockName": "parImage",
            "data": {
                ...
            }
        },
        {
            "blockName": "parRichTextEditor",
            "data": {
                ...
            }
        }
        {
            "blockName": "parSectionHeading",
            "data": {
                ...
            }
        }
    ]
}
```
```handlebars
{{#each rows}
    {{{ dynPartial blockName data}}}
{{/each}}
```
?> Please note that as `dynPartial` returns HTML it is necessary to not escape the helper's output (i.e. use triple braces: `{{{}}}`)

Already that's a huge improvement!

Now you may notice that we had to change the `blockName` property from containing any string value to render the correct block through the conditions to the name of the Handlebars partial (e.g. sectionHeading block will have a `parSectionHeading.hbs` file, so the string has to equal `parSectionHeading`) so it can use the correct one.

This however creates an issue: it ties the delivery and definition sides of the project together as renaming a block on the definition side could cause it not to render using `dynPartial` if the change wasn't reflected in the data being sent from the delivery side. 

To remidy this we decided to add a property called `_ref` which is automatically added to the data before it's rendered. It holds the string representaiton of its block name (e.g. `parImage`), removing this link between the two sides of the project and cleaning up the JSON simultaneously.

This makes the example even cleaner:
```json
{
    "rows": [
        {
            "data": {
                ...
            }
        },
        {
            "data": {
                ...
            }
        }
        {
            "data": {
                ...
            }
        }
    ]
}
```
```handlebars
{{#each rows}
    {{{ dynPartial data._ref data}}}
{{/each}}
```

## modPartial
Again, this may be easier to describe the problem first, both making it easier to justify the reason behind the helper and understand.

Imagine you have a header block (`parSectionHeading`) which, when in different contexts, needs to change its appearance. An easy example would be changing colours depending on what the background colour of the parent block. How would you tackle this problem?

If you want to skip a lot of explanation behind how we decided on this solution and why it's needed [skip ahead to modPartial usage](definition/markup?id=modpartial-usage).

### The scenario
Let's begin by outlining some markup to set the scene. Say this block `parSectionHeading` is used in two places:

`parBlog.hbs`
```handlebars
{{!-- Blog has a black background --}}
<section class="blog">
    {{> parSectionHeading heading}}
    ...
</section>
```
`parArticle.hbs`
```handlebars
{{!-- Article has a white background --}}
<section class="article">
    {{> parSectionHeading articleHeading}}
    ...
</section>
```
`parSectionHeading.hbs`
```handlebars
<header class="section-heading">
    {{#if title}}
        <h2 class="section-heading__title">{{title}}</h2>
    {{#if}}
    {{#if subTitle}}
        <h3 class="section-heading__subtitle">{{title}}</h3>
    {{/if}}
</header>
```
Pretty basic stuff right?

Not so much when we try and style the `parSectionHeading` in different ways depending on whether it is contained in the `blog` block or the `article` block.

With the risk of this turning into a delve into an explanation of cascading styles, let's say we have 2 options:
1.  We could simply inherit the `color` CSS property from the `.blog` and `.article` classes in their respective stylesheets e.g.
    ```scss
    .blog {
        background-color: #000000;
        color: #FFFFFF; // This would be inherited by the sectionHeading block (so long as it doesn't overwrite it)
    }
    ```
2.  We could use the SCSS parent selector (`&`) to create styles when the `.section-heading` class is nested within either `.article` or `.article` like so:
    ```scss
    .section-heading {

        .blog & {
            color: #FFFFFF;
        }

        .article & {
            color: #000000;
        }        
    }
    ```
    Which would compile to:
    ```css
    .blog .section-heading {
        color: #FFFFFF
    }
    .article .section-heading {
        color: #000000
    }
    ```


So we have two ways of effecting the colour of the text, which to their credit would work perfectly well. If they work why do we even need `modPartial` then I hear you ask! The problem is they are not at all clean and have inherent problems.

Take solution 1- what happens if we want the headings in `parSectionHeading` to not be one single text colour? For arguments sake, say the title needs to be white and the subtitle a light grey when used in blog, whereas when used in the article block (which uses a white background) the title should be black and have a dark grey subtitle. This fix instantly falls down when there is greater control needed.

Solution 2 however is viable even when styling the titles independently, succeeding where solution 1 fails:
```scss
.section-heading {
    $this: '.section-heading';

    .blog & {
        #{$this} {
            &__title {
                color: #FFFFFF;
            }

            &__subtitle {
                color: #D3D3D3;
            }
        }
    }

    .article & {
        #{$this} {
            &__title {
                color: #000000;
            }

            &__subtitle {
                color: #3F3F3F;
            }
        }
    }        
}
```
```css
.blog .section-heading .section-heading__title {
  color: #FFFFFF;
}
.blog .section-heading .section-heading__subtitle {
  color: #D3D3D3;
}
.article .section-heading .section-heading__title {
  color: #000000;
}
.article .section-heading .section-heading__subtitle {
  color: #3F3F3F;
}
```
?> Confused by/want to know more about the `.scss` file? Head on over to the ["Style"](definition/style) section to learn more

But again this solution is not without its issues. What happens when another block (say `parPersonProfile`) is added which also needs to use `parSectionHeading` and it too has a dark background like `parBlog`. Do you change the selector within the sectionHeading SCSS file to `.person-profile &, .blog &`?

Perhaps now you may evolve it to be more of a generic class and move away from using the outer blocks' classes as selectors within the `_parSectionHeading` stylesheet. Something like `.black-background` or maybe to cover both instances to avoid having to add to a selector everytime a block needs to use `parSectionHeading`.

But where would you put it? At the root of the parent block? No, that wouldn't make sense as it purely relates to `parSectionHeading`. Introducing a new element to wrap `parSectionHeading` like so would surely be cleanest?:
```handlebars
<section class="blog">
    <div class="black-background">
        {{> parSectionHeading heading}}
    </div>
    ...
</section>
```
But the issue with all of these solutions is that not one so far is without a problem:
- Relying on inherited styles is very limited and should be relied upon as little as possible as it's very fragile and is very easily overwritten by mistake
- All styles relating to a block should be kept within the block itself, not relying on styles from outside of its directory stylesheet or classes in markup outside of its own handlebars file
- Block names should not be used: adding the parent block class into the sub-block stylesheet (e.g. `.blog` in above examples) is messy and creates a link between them which needs updating if the blog block or `.blog` is renamed
- You shouldn't need to add classes/wrapping elements with classes around partial helpers purely to apply contextual styles to the sub-block. This extra markup or classes just pollutes your code and may not have clear intent to somebody looking at it for the first time
- None are using BEM's modifier convention, but modifier classes require both the knowledge of the sub-block's root class name (e.g. `.section-heading` to create the modifier class `.section-heading--black-background`, which again creates a link which needs updating if the class name is renamed) and we need access to the root of the sub-block markup to add this class (which do not have from the parent).

The answer: Yuzu's "\_" property `_modifiers` and the `modPartial` helper.

### modPartial Usage
`modPartial` simply uses an array called `_modifiers` which is automatically added to each block by Yuzu. `modPartial` allows for the addition of modifier classes to the sub-block, without knowing the block's base class. By using this value in a block's markup, we can cleanly add contextual styles through inserting modifier classes to the root of the sub-block markup.

To use `modPartial` simply call pass the name of the partial (as a string), the context and then as many modifiers as you want as string parameters to the end of the helper like so:

`parBlog.hbs`
```handlebars
<section class="blog">
    {{{ modPartial 'parSectionHeading' heading 'dark-background' 'larger-title' }}}
    ...
</section>
```


Then in the sub-block we can add these modifiers held in `_modifiers` like this:

`parSectionHeading.hbs`
```handlebars
<header class="section-heading {{#each _modifiers}} section-heading--{{this}} {{/each}}">
    ...
</header>
```
`parBlog` html output
```html
<section class="blog">
    <header class="section-heading  section-heading--dark-background  section-heading--larger-title ">
        ...
    </header>
    ...
</section>
```
This enables us to cleanly separate styling as much as possible, without having to know the root class the block is using from the parent block, it doesn't require extra markup, doesn't need the delivery side to pass us properties so that the block knows its context, and uses BEM's modifier concept. This all makes for the best solution we could envisage and works very well for us!

?> Please note that as `modPartial` returns HTML it is necessary to not escape the helper's output (i.e. use triple braces: `{{{}}}`)

?> The eagle-eyed among you may have realised that there is very little difference between the two helpers `dynPartial` and `modPartial`: after all they both take a string parameter of the name of the partial they render, the data context and return the compiled template in HTML. While it's true that they basically do the same thing (apart from `modPartial` pushing classes into the `_modifiers` property) and could be easily combined, we believe that having two separate helpers is better: it better illustrates intent, with one being purely for rendering the correct partial given a context (`dynPartial`) while the other should just be used when adding contextual classes to the partial (`modPartial`).

## pictureSource
!> Only designed to work with [ImageProcessor](https://imageprocessor.org/)

The `pictureSource` helper was borne out of frustration of maintaining `<picture>` element contents.

While building sites we'd often try to optimise our images for all screen sizes, using the picture element and source tags in combination with ImageProcessor. We started using only the basic functionality of ImageProcessor and the `<picture>` element until we started getting comfortable with both and digging deeper and finding more strategies and patterns we could take advantage of to really reduce the file sizes of image-heavy pages:
- media queries to serve different sized images for different devices
- requesting double sized images at a lower quality for higher density displays (e.g. retina), which produced smaller file sizes, with no percieved quality loss
- using WebP with a fallback source for those browsers where WebP is unsupported

However this came at a cost of maintainabilty. Yes, we could drastically increase the page load speed by serving images of smaller file sizes (especially with WebP), but we'd have to wrestle with unsightly code and we'd keep making mistakes like forgetting to change a value when making tweaks. This meant little issues would creep into blocks using picture elements and go unnoticed for a while as it would only effect a certain screen density at a certain screen size.

Take a look below at an example from one of our projects:
```handlebars
<picture>
    <source media="(min-width: 1367px)" srcset="{{image.src}}?width=794&height=640&mode=crop&quality=85&format=webp, {{image.src}}?width=1588&height=1280&mode=crop&quality=60&format=webp 1.5x" type="image/webp">
    <source media="(min-width: 1367px)" srcset="{{image.src}}?width=794&height=640&mode=crop&quality=80, {{image.src}}?width=1588&height=1280&mode=crop&quality=50 1.5x">

    <source media="(min-width: 1024px)" srcset="{{image.src}}?width=550&height=422&mode=crop&quality=85&format=webp, {{image.src}}?width=1100&height=844&mode=crop&quality=60&format=webp 1.5x" type="image/webp">
    <source media="(min-width: 1024px)" srcset="{{image.src}}?width=550&height=422&mode=crop&quality=80, {{image.src}}?width=1100&height=844&mode=crop&quality=50 1.5x">

    <source media="(min-width: 768px)" srcset="{{image.src}}?width=460&height=370&mode=crop&quality=85&format=webp, {{image.src}}?width=920&height=740&mode=crop&quality=60&format=webp 1.5x" type="image/webp">
    <source media="(min-width: 768px)" srcset="{{image.src}}?width=460&height=370&mode=crop&quality=80, {{image.src}}??width=920&height=740&mode=crop&quality=50 1.5x">

    <source media="(min-width: 601px)" srcset="{{image.src}}?width=690&height=556&mode=crop&quality=85&format=webp, {{image.src}}?width=1380&740&height=1112&mode=crop&quality=60&format=webp 1.5x" type="image/webp">
    <source media="(min-width: 601px)" srcset="{{image.src}}?width=690&height=556&mode=crop&quality=80, {{image.src}}?width=1380&740&height=1112&mode=crop&quality=50 1.5x">
    
    <source media="(min-width: 0px)" srcset="{{image.src}}?width=535&height=430&mode=crop&quality=85&format=webp, {{image.src}}?width=1070&height=860&mode=crop&quality=60&format=webp 1.5x" type="image/webp">
    <source media="(min-width: 0px)" srcset="{{image.src}}?width=535&height=430&mode=crop&quality=80, {{image.src}}?width=1070&height=860&mode=crop&quality=50 1.5x">
    <img src="{{image.src}}?width=1100&height=845&mode=crop&quality=80" alt="{{image.alt}}">
</picture>
```
So for each breakpoint we have:
- 2 sources (one for WebP, one for a fallback- uses whatever type the image is already): notice the `&format=webp` URL parameter and `type="image/webp"` attribute
- For each source (both WebP and non-WebP) we have a normal density crop using `width=[width]&height=[height]&mode=crop` URL parameters and a higher density version using `width=[width * 2]&height=[height * 2]&mode=crop`. Also notice the `1.5x` at the end of the higher density version.

So imagine if multiple tweaks needed to be made after the initial version had been built. Calculating double widths and height values, making sure you've updated both the WebP and non-WebP version etc. became very tedious, especially with multiple `picture` elements throughout the project.

Meanwhile with `pictureSource` the above example can be boiled down to:
```handlebars
<picture>
    {{{ pictureSource 'min-width: 1367px' image.src width=794 height=640 mode='crop'}}}
    {{{ pictureSource 'min-width: 1024px' image.src width=550 height=422 mode='crop'}}}
    {{{ pictureSource 'min-width: 768px' image.src width=460 height=370 mode='crop'}}}
    {{{ pictureSource 'min-width: 601px' image.src width=690 height=556 mode='crop'}}}
    {{{ pictureSource 'min-width: 0px' image.src width=535 height=430 mode='crop'}}}
    <img src="{{image.src}}?width=1100&height=845&mode=crop&quality=80" alt="{{image.alt}}">
</picture>
```

Which is much simpler and makes it far easier to understand what is going on. No more multiple sources for the same breakpoint, no long, difficult to read string of parameters, and no calculations required.

Simply by passing the necessary initial 2 required parameters: `{{{ pictureSource [media query string] [image url] }}}` we can begin to use the `pictureSource` helper.

Any additional optional parameters for the ImageProcessor are hash parameters and must be named e.g. `width=500`. Additional parameters are also used to control the `pictureSource` defaults which are as follows:

| pictureSource additional parameter    | Default Value | Description 			                                                                                        |
| ------------------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------- | 
| createHighDensityDisplay              | true          | Toggles creation of the higher resolution source for higher density displays (including for webP, if enabled) |
| createWebP                            | true          | Toggles creation of a separate source tag for a webP version of the image                                     |
| highDensityDisplayDensity             | '1.5x'        | srcset pixel density descriptor for higher density display version                                            |
| highDensityDimensionMultiplier        | 2             | Factor by which image dimensions are increased by for higher density displays                                 |
| quality                               | 80            | Quality of non-webP image for standard density displays                                                       |
| webPQuality                           | 85            | Quality of webP image for standard density displays                                                           |
| highDensityQuality                    | 50            | Quality of non-webP images on higher density displays                                                         |
| highDensityWebPQuality                | 60            | Quality of webP images on higher density displays                                                             |

So for example, you can mix and match as many ImageProcessor/pictureSource hash parameters as you like so long as they're named appropriately:
```handlebars
{{{ pictureSource 'min-width: 0px' image.src width=535 createHighDensityDisplay=false webPQuality=70}}}
or
{{{ pictureSource 'max-width: 600px' image.src quality=10 contrast=25 createWebP=false}}}
```

Further ImageProcessor options can be found [here](https://imageprocessor.org/imageprocessor-web/imageprocessingmodule/) under 'methods'
## toString

---

# Layouts
A layout is basically a wrapper around pages, generally containing common components which are included within the HTML document outside of the main page contents. This can range from just including things like links to stylesheets and Javascript files, to including common 'layout' blocks which appear on multiple pages: for example the site header/footer, cookie messages, modals etc.

It is also important to note that if you want to use the Yuzu Definition UI that you must attach some classes within your layout document to indicate where they root of the layout content (either the `<body>` itself or nested within `<body>`) and the root of the content is (nested within the `<body>`) by using the `.yuzu-layout-root` and `.yuzu-content-root` classes respectively. The Yuzu Definition UI script should also be included.

Below is a simple page layout illustrating what we've discussed so far:
```handlebars
<!doctype html>
<html>
	<head>
		<meta charset="utf-8">
		<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1.0, user-scalable=0">
		
        
        {{!-- Further meta/stylesheets here --}}
	</head>
	<body>
        {{!-- Yuzu Definition UI Script --}}
		<script src="/yuzu-def-ui/websocketClient.js" defer></script>
        
		<div class="yuzu-layout-root">
            {{> parSiteHeader header}}

			<div class="yuzu-content-root">
                {{!-- Page content is injected into {{{contents}}} --}}
				{{{ contents }}}
			</div>

            {{> parSiteFooter footer }}
		</div>

	</body>
</html>
```
```json
{
    "header": {
        "$ref": "/parSiteHeader"
    },
    "footer": {
        "$ref": "/parSiteFooter"
    }
}
```

---

# Best Practices
