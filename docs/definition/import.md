# Import
Yuzu Definition Import is a package designed to rapidly scaffold a project's blocks and pages, vastly reducing the time spent on the more mundane and repetitive tasks associated with setting up components for a new piece of work.

Using either individual Trello cards or .txt files for every block and/or page to record schema definitions in a shorthand the tool creates for every component:
1.  A fully fleshed out schema file
2.  A pre-prepared handlebars file, complete with property rendering, conditions, loops and a rough BEM class for every property (by combining the block/page and property names)
3.  Based on the generated handlebars markup, a SCSS file complete with class stubs
4.  An empty, "clean" JSON block/page state

?> We may look to support sources other than Trello and local `.txt` files in future, but we have no definite plans currently

It's important to note that while this tool can achieve a lot it is not by any means "perfect"- while it has it's limitations it's still a great starting point as it allows you to get to the meat of the project far faster.

For example, some more complex schema structures are currently not possible to create via the shorthand (`anyOf`, `enums` etc.), the only markup element generated are `div`s, the BEM classes are not really standard and may get overly long/verbose. It basically requires manual fine-tuning after initially being run.

---

# Installation
To install it globally, for use anywhere on your machine, run:

```npm install -g yuzu-definition-cli```

---

# Getting started
1. Within the root of your Yuzu project's Definition directory (e.g. `definition.src`) you should add a new folder called `config`.
2. Within `definition.src/config` create a `.json` file called `default.json`

<!-- tabs:start -->
#### ** Using Trello Cards **
```json
{
    "trello": {
        "board" : "{TRELLO BOARD NAME}",
        "key": "••••••••••••••••••••••••••••••••",
        "secret": "••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••••"
    },
    "generationSource": "trello"
}
```
?> Your developer api key and OAuth secret are available here: [Trello App Keys](https://trello.com/app-key)
#### ** Using .txt files **
1. Add another directory in the Definition root called `blockGeneration`. All `.txt` files will go here
2. At a minimum, add the following to your `definition.src/config/default.json` file:
    ```json
    {
        "generationSource": "localFiles"
    }
    ```
<!-- tabs:end -->

---

# Card/File naming
For every block/page you either create a card (Trello) or a `.txt` file, depending on the method you choose.

These cards/files should be titled (Trello) or named (`.txt`) using the following convention to properly differentiate between blocks and pages:

`Block - {{blockName}}`

`Page - {{pageName}}`

For example:
`Block - siteHeader` or `Page - homePage` on Trello
<br>
vs
<br>
`Block - siteHeader.txt` or `Page - homePage.txt`

---

# Schema shorthand
Regardless of which method you choose to use to record your block/page schema shorthand, the process of writing is near enough identical- the only difference is where it is written: with `.txt` files it's just in the content, with Trello it's just in the cards' descriptions.

## Rules
1.  To be detected and parsed by the generator begin your schema shorthand with the word "Schema", followed by a colon (`Schema:`). If on Trello you're using the description for content other than your schema shorthand, begin writing at the end of the field.
2.  We use a simple format: all properties should be preceded by a type (object/array/string/boolean/integer/number). Sub-blocks are more complicated however, and will be explained further later on
3.  You can add properties in any order with one major caveat: you **MUST** first "initialise" arrays and objects before adding properties to them (in other words you must write the shorthand for the array/object before writing sub-properties for them)

## Examples
Let's start off with the most basic schema: no sub-blocks, arrays or objects:

### Basic example
We'll start off by writing the shorthand for a simple block (parUser), with a flat schema (no nested properties)
<!-- tabs:start -->
#### ** Block - user (Schema shorthand) **
```
Schema:
- string+userName
- integer+age
- number+heightMetres
- boolean+isAdmin
```
#### ** Generated schema (parUser.schema) **
```json
{
    "id": "/parUser",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "properties": {
        "userName": {
            "type": "string"
        },
        "age": {
            "type": "integer"
        },
        "heightMetres": {
            "type": "number"
        },
        "isAdmin": {
            "type": "boolean"
        }
    },
    "additionalProperties": false
}
```
<!-- tabs:end -->

Pretty straight forward isn't it? Remember, this simple shorthand will not only generate your schema, but a whole directory for the block/page compete with some Handlebars markup, some basic SCSS stubs and an empty JSON state:
<!-- tabs:start -->
#### ** parUser.hbs **
```handlebars
<div class="user{{#each _modifiers}} user--{{this}}{{/each}}">

    {{#if userName}}
        <div class="user__user-name">
            {{userName}}
        </div>
    {{/if}}
    {{#if age}}
        <div class="user__age">
            {{age}}
        </div>
    {{/if}}
    {{#if heightMetres}}
        <div class="user__height-metres">
            {{heightMetres}}
        </div>
    {{/if}}
    {{#if isAdmin}}
        <div class="user__is-admin">
            {{isAdmin}}
        </div>
    {{/if}}
</div>
```
#### ** _parUser.scss **
```scss
.user {

    &__user-name {}

    &__age {}

    &__height-metres {}

    &__is-admin {}    
}
```
#### ** parUser.json **
```json
{
    "userName": "",
    "age": 0,
    "heightMetres": 0,
    "isAdmin": false
}
```
<!-- tabs:end -->

### Object/Array examples
Arrays and objects are not really that much more complex than the other properties: all you have to remember is to first initialise the object/array itself before trying to push sub-properties to the object/array's child object.

To append properties to an array child object/object you must simply append the name of the parent object/array property.

#### Simple object:
<!-- tabs:start -->
#### ** Block - quote **
```
Schema:
- string+text
- object+person
- person+string+personName
- person+string+jobTitle
- person+string+company

```

#### ** Generated schema (parQuote.schema) **
```json
{
    "id": "/parQuote",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "properties": {
        "text": {
            "type": "string"
        },
        "person": {
            "type": "object",
            "properties": {
                "personName": {
                    "type": "string"
                },
                "jobTitle": {
                    "type": "string"
                },
                "company": {
                    "type": "string"
                }
            },
            "additionalProperties": false
        }
    },
    "additionalProperties": false
}
```
<!-- tabs:end -->

#### Simple array:
<!-- tabs:start -->
#### ** Block - recipe **
```
Schema:
- array+ingredients
- ingredients+number+quantity
- ingredients+string+unit
- ingredients+string+ingredient

```
#### ** Generated schema (parRecipe.schema) **
```json
{
    "id": "/parRecipe",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "properties": {
        "ingredients": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "quantity": {
                        "type": "number"
                    },
                    "unit": {
                        "type": "string"
                    },
                    "ingredient": {
                        "type": "string"
                    }
                },
                "additionalProperties": false
            }
        }
    },
    "additionalProperties": false
}
```
<!-- tabs:end -->

#### Deep nesting
Of course more all blocks/pages are not going to be so straightforward: they require a combination of both objects and arrays nested within another object or array.

These are still possible via our shorthand language, however.

?> The following example should be taken with a grain of salt- try to avoid nesting too deeply 
<!-- tabs:start -->
#### ** Block - footballTeams **
```
Schema:
- array+teams
- teams+string+teamName
- teams+string+location
- teams+string+stadiumName
- teams+array+players
- teams+players+string+name
- teams+players+integer+age
- teams+players+object+statistics
- teams+players+statistics+object+goals
- teams+players+statistics+goals+integer+thisSeason
- teams+players+statistics+goals+integer+careerTotal
- teams+players+statistics+goals+number+average
- teams+players+statistics+object+appearances
- teams+players+statistics+appearances+integer+thisSeason
- teams+players+statistics+appearances+integer+careerTotal

```
#### ** Generated schema (parFootballTeams.schema) **
```json
{
    "id": "/parFootballTeams",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "properties": {
        "teams": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "teamName": {
                        "type": "string"
                    },
                    "location": {
                        "type": "string"
                    },
                    "stadiumName": {
                        "type": "string"
                    },
                    "players": {
                        "type": "array",
                        "items": {
                            "type": "object",
                            "properties": {
                                "playerName": {
                                    "type": "string"
                                },
                                "age": {
                                    "type": "integer"
                                },
                                "statistics": {
                                    "type": "object",
                                    "properties": {
                                        "goals": {
                                            "type": "object",
                                            "properties": {
                                                "thisSeason": {
                                                    "type": "integer"
                                                },
                                                "careerTotal": {
                                                    "type": "integer"
                                                },
                                                "average": {
                                                    "type": "number"
                                                }
                                            },
                                            "additionalProperties": false
                                        },
                                        "appearances": {
                                            "type": "object",
                                            "properties": {
                                                "thisSeason": {
                                                    "type": "integer"
                                                },
                                                "careerTotal": {
                                                    "type": "integer"
                                                }
                                            },
                                            "additionalProperties": false
                                        }
                                    },
                                    "additionalProperties": false
                                }
                            },
                            "additionalProperties": false
                        }
                    }
                },
                "additionalProperties": false
            }
        }
    },
    "additionalProperties": false
}
```
<!-- tabs:end -->

### Sub-blocks
There are two ways of writing shorthand for a sub-block:
1.  As a property it has 3 components: `- subBlock+{blockName}+{propertyName}`  
    e.g. `- subBlock+parFootballTeams+teams`
2.  As an array child item. For example, imagine the scenario where you have a `parProductGrid` block, which has an array called `products`. We can reference a block (`parProduct`) as a direct child in the schema of that array, without having to add it as a property, wrapped in an object within that array. All we simply have to do is to not append the property name to the end of the shorthand  
    e.g. `- array+products`
    `- products+subBlock+parProduct`


#### As a property

<!-- tabs:start -->
#### ** Page - contentPage **
```
Schema:
- subBlock+parHero+heroSection
- subBlock+parArticle+content
- subBlock+parPageFooter+footer

```
#### ** Generated schema (parRecipe.schema) **
```json
{
    "id": "/contentPage",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "properties": {
        "heroSection": {
            "$ref" "/parHero"
        },
        "content": {
            "$ref" "/parArticle"
        },
        "footer": {
            "$ref" "/parPageFooter"
        }
    },
    "additionalProperties": false
}
```
<!-- tabs:end -->

#### As an array item

<!-- tabs:start -->
#### ** Page - productListingPage **
```
Schema:
- array+products
- products+subBlock+parProduct

```
#### ** Generated schema (parRecipe.schema) **
```json
{
    "id": "/productListingPage",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "properties": {
        "products": {
            "type": "array",
            "items": {
                "$ref": "/parProduct"
            }
        }
    },
    "additionalProperties": false
}
```
<!-- tabs:end -->

Here's what would happen if we incorrectly named the `parProduct` array child item from the above example. **DO NOT** obviously do this as it just adds an unnecessary wrapping object!

<!-- tabs:start -->
#### ** Page - productListingPage **
```
Schema:
- array+products
- products+subBlock+parProduct+product

```
#### ** Generated schema (parRecipe.schema) **
```json
{
    "id": "/productListingPage",
    "$schema": "http://json-schema.org/draft-04/schema#",
    "description": "",
    "type": "object",
    "properties": {
        "products": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "product": {
                        "$ref": "/parProduct"
                    }
                }
            }
        }
    },
    "additionalProperties": false
}
```
<!-- tabs:end -->

---

# Advanced configuration
Below is a complete representation of all the default options available to be overridden in your `config/default.json` file, used by the Definition Import tool. It allows you to tailor what's generated and from what source.

We've not managed to expose all the settings we wanted to yet. If there are alterations to output of the tool you want to adjust, please get in touch.
```json
{
    "markupSettings": {
        "defaultMarkupTag": "div", // Element used for all data (apart from dataStructures)
        "classNameDivider": "__", // Used in BEM class generation e.g. .test-block__property
        "indentSize": 4, // Handlebars indentation size
        "backupRefArrayChildClass": "item" // Fallback class name when one cannot be generated automatically e.g. .test-block__array__item
    },
    "prefixes": {
        "block": {
            "card": "Block - ", // What the Trello card title/.txt file name should begin with if it's a block e.g. "Block - testBlock"
            "fileName": "par" // Prefix for block filenames e.g. "par" = "parTestBlock.hbs"
        },
        "page": {
            "card": "Page - ", // What the Trello card title/.txt file name should begin with if it's a page e.g. "Page - aboutUs"
            "fileName": "" // Prefix for block filenames e.g. "page" = "pageAboutUs.hbs"
        },
        "schema": {
            "card": "Schema" // Indicates the beginning of schema shorthand in the description of Trello cards/contents of .txt files
        },
        "property": {
            "card": "- " // What each property should begin with in the schema shorthand e.g. "- array+students"
        }
    },
    "localFiles": {
        "directoryPath": "./blockGeneration" // Relative path to the root of where your .txt files are stored from the definition directory
    },
    "generationSource": "", // Toggle between either "trello" or "localFiles"
    "trello": {
        "board" : "", // Name of the Trello board
        "key": "", // Your Trello developer api key
        "secret": "" // Your Trello OAuth secret
    },
    "yuzuPro": {
        "key": "" // Your Yuzu licence for use with the Definition Import tool
    }
}
```