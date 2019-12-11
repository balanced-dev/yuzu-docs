# Schema
The JSON schemas for all the blocks, pages and layouts are probably the most important components of the overall Yuzu process: it's the very interface between the definition and delivery hemispheres, the glue that holds the project together.

As block/page/layout data is validated against their relevant schemas, it effectively documents the data structure; defining what a component can contain and what its strucutre is. The delivery side then in turn relies on this to structure itself to fufil the outlined data requirements.

This is not, however, the sole responsibility of frontend develipers to maintain: as both the definition and delivery halfs of the project rely on it for the project to operate smoothly. Of course, more responsibility naturally falls on the definition side as those people coding there will be more aware of content and what data is required to meet the designs, but make no mistake it's a collaborative process. It's very likely there will be changes throughout the initial build phase and beyond with properties being added, renamed, removed or relocated in the schema which will be the concern of the frontend developer(s) and thus these alterations will have to be mirrored by the delivery side. Conversely there will be limitations and issues with the backend implementation which frontend developers may be unaware of, requiring a specific structure and therefore changes to be made to the schema.

Here is a simplified example schema, illustrating the main basic features (there are more, but we find them mostly unnecessary):

<!-- tabs:start -->
#### **parExampleBlock.schema **
```json
{
	"id": "/parEmployee",	
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"name": {
			"type": "string"
		},
		"isFulltime": {
			"type": "boolean"
		},
		"age": {
			"type": "integer"
		},
		"hourlyWage": {
			"type": "number"
		},
		"headshot": {
			"type": "object",
			"additionalProperties": false,
			"properties": {
				"src": {
					"type": "string"
				},
				"alt": {
					"type": "string"
				}
			}
		},
		"responsibilites": {
			"type": "array",
			"items": {
				"type": "string"
			}
		}
	}
}
```
#### **parExampleBlock.json **
```json
{
	"name": "Robert Parr",
	"isFulltime": true,
	"age": 40,
	"hourlyWage": 27.16,
	"headshot": {		
		"src": "https://www.example.com/example.jpg",
		"alt": "Robert Parr Headshot"
	},
	"responsibilites": [
		"General administration", 
		"Handling claims", 
		"Helping Insuracare stay in the black"
	]
}
```
<!-- tabs:end -->

The above example contains:
-	`id`: a unique identifier for the schema (the block/page name). This is what `$ref` actually refers to when linking block/page schemas together and so is very important
-	`$schema`: used to declare that that the JSON is actually a piece of JSON Schema (not required, but good practice to)
-	`type`: defines both the data type for a schema and the schema's properties. A schema is generally an object or on the odd occasion an array. The main types for properties are string, boolean, integer, number, object and array
-	`additionalProperties`: when set to false it disallows the addition of any properties not outlined in the schema into an object property. This is important as otherwise any valid JSON properties can be added to an object without an error being thrown


While JSON schemas can be far more complex than in this example and indeed in the rest of this section, we find we don't really see any need to reach for these other options as we've managed so far with a far more limited set over multiple years, through many projects.

If you want to document your JSON schema, feel free to use the `$comment`, or when annotating use `title` or `description`.

?> If you want to delve deeper into JSON schemas than we do here, [json-schema.org](https://json-schema.org/understanding-json-schema/index.html) is a great resource.

---



# Sub-blocks (`$ref`)
As blocks are connected to pages or other blocks, through the use of `$ref` in JSON files, so too this occurs in the schema. This either occurs in a named property or in an array.

## As properties
This is the simplest way of connecting blocks. For example say a block called `parProduct` wants to include a `parProductCode` block, you'd simply assign a reference to `parProductCode` in `parProduct` to a named property (`code`) using `$ref` like so:

<!-- tabs:start -->
#### **parProduct.schema **
```json
{
	"id": "/parProduct",
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"code": {
			"$ref": "/parProductCode"
		}
	}
}
```
<!-- tabs:end -->


## In arrays
Another common way that schemas are connected is within arrays, used when you need to render the same sub-block multiple times. For example `parGallery` needs to render any number of `parImage` sub-blocks, the schema may look something like this:

<!-- tabs:start -->
#### **parGallery.schema **
```json
{
	"id": "/parGallery",
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"images": {
			"type": "array",
			"items": {
				"$ref": "/parImage"
			}
		}
	}
}
```
<!-- tabs:end -->



## anyOf
What happens when you want to render one of multiple sub-blocks within the parent page/block? How do you represent this in the schema? `anyOf` is your answer.

Simply by using `anyOf` in the schema instead of just assigning a singular `$ref` we can provide an array of different sub-blocks a property or array can contain, in the form of an array of options in the schema.

As a property:
<!-- tabs:start -->
#### **parProduct.schema **
```json
{
	"id": "/parProduct",
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"code": {
			"anyOf": [
                {
                    "$ref": "/parProductCode"
                },
                {
                    "$ref": "/parSku"
                }
            ]			
		}
	}
}
```
<!-- tabs:end -->

Or as an array:
<!-- tabs:start -->
#### **parGallery.schema **
```json
{
	"id": "/parGallery",
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"images": {
			"type": "array",
			"items": {
				"anyOf": [
                	{
						"$ref": "/parImage"
					},
                	{
						"$ref": "/parVideo"
					}
				]
			}
		}
	}
}
```
<!-- tabs:end -->

?> Don't forget, when you're rendering one of multiple sub-blocks to use the ['dynPartial' helper](definition/markup?id=dynpartial)

---

# Data Structures
Throughout this documentation we've stressed the importance of where the code for all five possible components of blocks (markup, style, schema, data, Javascript) should be placed.

Where style can be tied to a specific block or made global via a mixin in the project (allowing for tweaks through parameters), and Javascript similarly can either be tightly bound to a block or not (by being placed within its directory or a global one), the same, easy binary decision cannot be made for the other three components.

Markup, schema and data are intrinsically linked: the latter two are obvious as the data is validated against the schema, regardless of whether it is inline in a parent block/page or contained within a state, held in its own directory. The markup however only becomes its own block if it's deemed to be complex enough, follows a common-enough layout/style and can therefore be easily included by other blocks/pages which will not be able to easily change its markup. The markup and schema (and thus by extension, the data) are joined as the markup simply renders a set of values available to it.

However, what if there is a common data or structure pattern that emerges, that is simple enough to not warrant its own block, where often the parent block wants direct control of the markup? This is not easily possible if it is within its own block. The answer is data structures: they are basically blocks consisting of a solitary schema file, with their data being spread throughout the project in their parent blocks/pages. They standardise and centralise common data patterns without perscribing any markup or style.

Take for example a link: it has three main properties: it needs an href, a title and some textural content (a label). Therefore a schema for this data structure could look something like this:

<!-- tabs:start -->
#### **parDataLink.schema **
```json
{
	"id": "/parDataLink",
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"label": {
			"type": "string"
		},
		"href": {
			"type": "string"
		},
		"title": {
			"type": "string"
		}
	}
}
```
<!-- tabs:end -->

You can then reference `parDataLink` from the parent block's/page's schema. For example, say if both a navigation block (`parSiteNav`) and a product listing block (`parProductListing`) both use `parDataLink`. Their schemas may look something like this:

<!-- tabs:start -->
#### ** parSiteNav.schema **
```json
{
	"id": "/parSiteNav",
	"$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"links": {
			"type": "array",
			"items": {
				"$ref": "/parDataLink"
			}
		}
	}
}
```
#### ** parProductListing.schema **
```json
{
	"id": "/parProductListing",
	"$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"products": {
			"type": "array",
			"items": {
				"type": "object",
				"additionalProperties": false,
				"properties": {
					"link": {
						"$ref": "/parDataLink"
					},
					"image": {
						"type": "object",
						"additionalProperties": false,
						"properties": {
							"src": {
								"type": "string"
							},
							"alt": {
								"type": "string"
							}
						}
					},
					"description": {
						"type": "string"
					}
				}
			}
		}
	}
}
```
<!-- tabs:end -->


---

# File type
All of our example schemas so far have been of the the `object` type at their root, so you would be forgiven in thinking that this is the only valid option for them. However this is not the case (although the majority of the time your schemas are likely to be `objects`): the second most used type of schema in our projects for example is `array`.

<!-- tabs:start -->
#### **parContentPage.schema **
```json
{
	"id": "/parDataLink",
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"content": {
			"$ref": "/parContentBlocks"
		}
	}
}
```
#### **parContentBlocks.schema **
```json
{
	"id": "/parContentBlocks",
    "$schema": "http://json-schema.org/schema#",
	"type": "array",
	"items": {
		"anyOf": [
			{
				"$ref": "/parCta"
			},
			{
				"$ref": "/parText"
			},
			{
				"$ref": "/parImage"
			}
		]
	}
}
```
<!-- tabs:end -->

---

# Enums
Enumerated values restrict properties to a set of values, in the form of an array with one or more unique elements.

On the delivery side this helps to populate the options in a dropdown list in the CMS with the values in the enum.

It's also just useful to document and expose what values are available to use in a property, rather than relying on creating/looking through block/page states.

<!-- tabs:start -->
#### **parHero.schema **
```json
{
	"id": "/parHero",
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
	"additionalProperties": false,
	"properties": {
		"title": {
			"type": "string"
		},
		"theme": {
			"type": "string",
            "enum": ["white","black","grey"]
		}
	}
}
```
<!-- tabs:end -->

?> To render enum properties, see the [Yuzu enum helper](definition/markup?id=enum)