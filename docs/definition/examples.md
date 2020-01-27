# Examples
In this section, we'll be building upon concepts previously introduced (namely using [dynamicBlocks](definition/dynamicblocks) as well as the ['anyOf' property](definition/schema?id=anyOf), the ['dynPartial' helper](definition/markup?id=dynPartial) etc.) to create simple versions of two common patterns when constructing websites: forms and grids.

Building key, repeating structures in a way similar to the below examples will help with consistent implementations within a site and indeed throughout all your sites if reused. It therefore allows for easy integration into the Umbraco grid and Umbraco forms.

---

# Building a grid
In this example, we'll build a simple "row only" grid, with no columns for simplicity's sake: each block is effectively a row. This can easily be changed by just creating a new data structure with a slightly different schema to incorporate the idea of columns.

Speaking of data structures, let's begin by writing one for the rows to be used in all of our grids.

<!-- tabs:start -->
#### **parDataGridRows.schema**
```json
{
    "id": "/parDataGridRows",
    "type": "object",
    "additionalProperties": false,
    "anyOfTypes": [
        "parSectionGrid"
    ],
    "properties": {
        "rows": {
            "type": "array",
            "items": {
                "type": "object",
                "additionalProperties": false,
                "properties": {
                    "config": {
                        "anyOf": [
                            {
                                "$ref": "/parSectionGridConfig"
                            }
                        ]
                    },
                    "items": {
                        "anyOf": [
                            {
                                "$ref": "/parSectionGridItems"
                            }
                        ]
                    }
                }
            }
        }
    }
}
```
<!-- tabs:end -->

By leveraging "anyOfTypes" and "anyOf" we can have seperate grids, with their own Handlebars rendering, configuration settings and blocks allowed in them. In `parDataGridRows` we've already set it up to connect to a sub-block of `parSectionGrid` so we'll create this now:

<!-- tabs:start -->
#### **parSectionGrid.hbs**
```handlebars
<div class="section-grid">
{{#each rows}}
    {{#each items}}
        {{{ dynPartial _ref this }}}
    {{/each}}
{{/each}}
</div>
```
#### **parSectionGridConfig.schema**
```json
{
    "id": "/parSectionGridConfig",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "additionalProperties": false,
    "properties": {}
}
```
#### **parSectionGridItems.schema**
```json
{
    "id": "/parSectionGridItems",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "array",
    "items": {
        "anyOf": []
    }
}
```
<!-- tabs:end -->

At the moment though these schemas are empty- let's populate the config and add a few example blocks to our grid items.

Adding a background colour to the rows (in the markup & config)
<!-- tabs:start -->
#### **parSectionGrid.hbs**
```handlebars
<div class="section-grid">
{{#each rows}}
    <section class="section-grid__row{{#if config.backgroundColour}} section-grid__row--{{config.backgroundColour}}{{/if}}">
    {{#each items}}
        {{{ dynPartial _ref this }}}
    {{/each}}
{{/each}}
</div>
```
#### **parSectionGridConfig.schema**
```json
{
    "id": "/parSectionGridConfig",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "additionalProperties": false,
    "properties": {
        "backgroundColour": {
            "type": "string",
            "enum": ["primary","secondary","tertiary"]
        }
    }
}
```
<!-- tabs:end -->

And now to add a few examples sub-blocks into our grid items schema:
<!-- tabs:start -->
#### **parSectionGridItems.schema**
```json
{
    "id": "/parSectionGridItems",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "array",
    "items": {
        "anyOf": [
            {
                "$ref": "/parHero"
            },
            {
                "$ref": "/parImage"
            }
        ]
    }
}
```
<!-- tabs:end -->

We're then free to use the "section grid" block in a parent block/page.

Let's do it now by adding it to our home page
<!-- tabs:start -->
#### **home.hbs**
```handlebars
{{#if content}}
    {{> parSectionGrid content }}
{{/if}}
```
#### **home.schema**
```json
{
    "id": "/home",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "additionalProperties": false,
    "properties": {
        "content": {
            "$ref": "/parDataGridRows",
            "anyOfType": "parSectionGrid"
        }
    }
}
```
#### **home.json**
```json
{
    "content": {
        "rows": [
            {
                "items": [
                    {
                        "$ref": "/parHero"
                    }
                ]
            },
            {
                "items": [
                    {
                        "$ref": "/parImage_large"
                    }
                ]
            },
            {
                "items": [
                    {
                        "$ref": "/parImage"
                    }
                ]
            }
        ]
    },
```
<!-- tabs:end -->

---

# Building a form
Like grids, we found standardisation through using data structures helps to keep things consistent throughout a site, as well as between projects. Though not maybe applicable to all forms used on a site, we've found that they can be used in conjunction with Umbraco forms quite easily.

We begin with 2 data structures:
<!-- tabs:start -->
#### **parDataForm.schema**
```json
{
    "id": "/parDataForm",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "additionalProperties": false,
    "properties": {
        "testForm": {
            "anyOf": [
                {
                    "$ref": "/parDataFormBuilder"
                }
            ]
        },
        "liveForm": {
            "type": "string"
        }
    }
}
```
#### **parDataFormBuilder.schema**
```json
{
    "id": "/parDataFormBuilder",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "additionalProperties": false,
    "anyOfTypes": [
        "parFormBuilder"
    ],
    "properties": {
        "title": {
            "type": "string"
        },      
        "isSuccess": {
            "type": "boolean"
        },      
        "successMessage": {
            "type": "string"
        },        
        "validation": {
            "type": "object",
            "additionalProperties": false,
            "properties": {
                "title": {
                    "type": "string"
                },
                "message": {
                    "type": "string"
                }
            }
        },
        "fieldsets": {
            "type": "array",
            "items": {
                "type": "object",
                "additionalProperties": false,
                "properties": {
                    "legend": {
                        "type": "string"
                    },
                    "fields": {
                        "anyOf": [
                            {
                                "$ref": "/parFormBuilderFields"
                            }
                        ]
                    }
                }
            }
        },
        "submitButtonText": {
            "type": "string"
        },
        "actionLinks": {
            "type": "array",
            "items": {
                "$ref": "/parDataLink"
            }
        }
    }
}
```
<!-- tabs:end -->
`parDataForm` is a very simple block used as a "switch", with two properties: `testForm` is just standard sub-block JSON (properties, objects, arrays etc.) whereas `liveForm` will only contain raw HTML. This is because the backend may require direct control of the `<form>` tag and thus it's easier to just render the entire form's HTML on the delivery side. However we still need to test and mimic forms on the definition side, which is why `testForm` property exists.

`parDataFormBuilder` meanwhile is more complex: it's a basic framework for what we believe most forms are structured like and covers most bases, while not overcomplicating things. We make use again of "anyOfTypes" and "anyOf" to link to our example form block of `parFormBuilder` to render our `parDataFormBuilder` data structure and also outline our selection of form element sub-blocks in a `parFormBuilderFields` schema.

So, to give the delivery side the ability to render forms and thus access to the `<form>` tag, all forms should be rendered by a "schema-less" block: a basic Handlebars partial like the one below.

<!-- tabs:start -->
#### **parForm.hbs**
```handlebars
{{#if testForm}}
    <form action="#">
        {{{ dynPartial testForm._ref testForm }}}
    </form>
{{else if liveForm}}
    {{{liveForm}}}
{{/if}}
```
<!-- tabs:end -->

We then setup the block for the contents of our form: `parFormBuilder` which consists of a Handlebars partial and the form elements available via a schema (`parFormBuilderFields`).
<!-- tabs:start -->
#### **parForm.hbs**
```handlebars
<div class="form{{#if modifier}} {{modifier}}{{/if}}">
    <header class="form__header">
        <div class="form__header__main">
            {{#if title}}
                <h2 class="form__title">{{title}}</h2>
            {{/if}}
        </div>
    </header>
    <div class="form__main">
        {{#if isSuccess}}
            <p class="form__success">{{successMessage}}</p>
        {{else}}
            {{#if validation.message}}
                <div class="form__errors">
                    {{#if validation.title}}
                        <h3 class="form__errors__title">{{validation.title}}</h3>
                    {{/if}}
                    <div class="form__errors__content">
                        {{{validation.message}}}
                    </div>
                </div>
            {{/if}}
            {{#each fieldsets}}
                <fieldset class="form__fieldset">
                    {{#if legend}}
                        <legend class="form__fieldset__legend">{{legend}}</legend class="form__fieldset">
                    {{/if}}
                    {{#each fields}}
                        {{#if _ref }}
                            {{{ modPartial _ref this 'form' }}}
                        {{/if}}
                    {{/each}}
                </fieldset>
            {{/each}}
        {{/if}}            
        <div class="form__actions">
            {{#if submitButtonText}}
                <button class="default__form-button" type="submit">
                    {{submitButtonText}}
                </button> 
            {{/if}}

            {{#if actionLinks.[0]}}
                <div class="form__links">
                    {{#each actionLinks}}
                        <a class="form__links__link" href="{{href}}" title="{{title}}">{{label}}</a>
                    {{/each}}
                </div>
            {{/if}}
        </div>
    </div>
</div>
```
#### **parFormBuilderFields.schema**
```json
{
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "array",
    "items": {
        "anyOf": [
            {
                "$ref": "/parFormCheckboxRadio"
            },
            {
                "$ref": "/parFormCheckboxRadioList"
            },
            {
                "$ref": "/parFormTextArea"
            },
            {
                "$ref": "/parFormTextInput"
            },            
            {
                "$ref": "/parFormSelect"
            },            
            {
                "$ref": "/parFormHidden"
            },            
            {
                "$ref": "/parFormRecaptcha"
            },            
            {
                "$ref": "/parFormTitleAndDescription"
            }
        ]
    },
    "additionalProperties": false,
    "id": "/parFormBuilderFields"
}
```
<!-- tabs:end -->

From there, we can define our form element blocks and then go on to use all of what we've built in a parent block/page, like so:
<!-- tabs:start -->
#### **formPage.hbs**
```handlebars
{{#if form}}
    {{> parForm form }}
{{/if}}
```
#### **formPage.schema**
```json
{
    "id": "/formPage",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "additionalProperties": false,
    "properties": {
        "form": {
            "$ref": "/parDataForm",
            "anyOfType": "parFormBuilder"
        }
    }
}
```
#### **formPage.json**
```json
{
     "form": {
        "testForm": {
            "title": "Login",
            "fieldsets": [
                {
                    "fields": [
                        {
                            "$ref": "/parFormTextInput"
                        }
                    ]
                },
                {
                    "fields": [
                        {
                            "$ref": "/parFormSelect"
                        }
                    ]
                }
            ],
            "submitButtonText": "Example submit text",
            "actionLinks": [
                {
                    "label": "Go back",
                    "href": "#",
                    "title": ""
                }
            ],
        }
     }
}
```
<!-- tabs:end -->