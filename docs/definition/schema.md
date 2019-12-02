# Schema


---



# Sub-blocks


---

# Data Structures
Throughout this documentation we've stressed the importance of where the code for all five possible components of blocks (markup, style, schema, data, Javascript) should be placed.

Where style can be tied to a specific block or made global via a mixin in the project (allowing for tweaks through parameters), and Javascript similarly can either be tightly bound to a block or not (by being placed within its directory or a global one), the same, easy binary decision cannot be made for the other three components.

Markup, schema and data are intrinsically linked: the latter two are obvious as the data is validated against the schema, regardless of whether it is inline in a parent block/page or contained within a state, held in its own directory. The markup however only becomes its own block if it's deemed to be complex enough, follows a common-enough layout/style and can therefore be easily included by other blocks/pages which will not be able to easily change its markup. The markup and schema (and thus by extension, the data) are joined as the markup simply renders a set of values available to it.

However, what if there is a common data or structure pattern that emerges, that is simple enough to not warrant its own block, where often the parent block wants direct control of the markup which is not easily possible if it is within its own block. The answer is data structures: they basically are basically blocks made up of a solitary schema file, with their data being spread throughout the project in their parent blocks/pages. They standardise and centralise common data patterns without perscribing any markup or style.

Take for example a link: it has three main properties: it needs an href, a title and some textural content (a label). Therefore a schema for this data structure could look something like this:

<!-- tabs:start -->
#### **parDataLink.schema **
```json
{
	"id": "/parDataLink",
	"type": "object",
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
	},
	"additionalProperties": false
}
```
<!-- tabs:end -->

You can then reference `parDataLink` from the parent block's/page's schema. For example, say if both a navigation block (`parSiteNav`) and a product listing block (`parProductListing`) both use `parDataLink`. Their schemas may look something like this:

<!-- tabs:start -->
#### ** parSiteNav.schema **
```json
{
	"id": "/parSiteNav",
	"type": "object",
	"properties": {
		"links": {
			"type": "array",
			"items": {
				"$ref": "/parDataLink"
			}
		}
	},
	"additionalProperties": false
}
```
#### ** parSiteNav.schema **
```json
{
	"id": "/parProductListing",
	"type": "object",
	"properties": {
		"products": {
			"type": "array",
			"items": {
				"type": "object",
				"properties": {
					"link": {
						"$ref": "/parDataLink"
					},
					"image": {
						"type": "object",
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
	},
	"additionalProperties": false
}
```
<!-- tabs:end -->


---

# File type


---

# Enums
