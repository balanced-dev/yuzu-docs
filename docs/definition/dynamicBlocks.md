# Dynamic Blocks
Before beginning this section, you should be comfortable with the ['anyOf' property](definition/schema?id=anyOf), the ['dynPartial' helper](definition/markup?id=dynPartial) and, most importantly, the concepts introduced by [schema 'data structures'](definition/schema?id=data-structures)

Within this section we'll be taking the the idea of centralising & standardising schemas as seen in schema data structures and taking them further by introducing the `ofType` block structure.

---

# ofType
"ofType" structures, as we call them, is a strategy which allows us to create data structures that are rigid and generic where possible, but allow for variation where necessary (via mutliple uses of `anyOf`).

That probably doesn't make a whole lot of sense initially and it may be easier to break this down a bit and then give a very simple example.

For "ofType" to work we need 3 things:
1.  **The generic schema**:
    This outlines what is common and what is variable within the structure. Like the previously discussed, more basic "data structure", it is simply a schema file, with no associated markup, data etc.

    The variable properties will use `anyOf` to reference sub-schemas which will outline what it can contain (usually sub-blocks).

    It also uses an additional, custom property called `anyOfTypes` to register the sub-block option(s) which rely on the generic schema and will do the rendering 
2.  **The specific schema(s) & markup**:
    This is basically a block, oddly enough, that doesn't have a it's own specific schema, but does contain markup and can contain styling.

    What it will also contain is the sub-schema(s) for any variable (`anyOf`) properties in the generic schema, which it relies on.

    In other words if, for example, one of the `anyOfTypes` options is `parExample` in a "generic" schema, there would be a `parExample.hbs` file and one or more sub-schema files with a prefix of "parExample" (e.g. `parExampleItems.schema`)
3.  **The parent/instance schema**
    This is just a block/page schema that references the outlined generic schema for one of its properties, using not only `$ref` but also an additional property within the value of the property: `anyOfType` containing the chosen option out of the selection outlined by the `anyOfTypes` property within the generic schema

Now that's a lot to digest so, before going on further, we'll give a totally abstract example (but hopefully one that will make it make more sense!).

<!-- tabs:start -->
#### **parDataFarm.schema (generic)**
```json
{
	"id": "/parDataFarm",
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
    "anyOfTypes": ["parDairy", "parPoultry", "parMeat", "parArable", "parMixed"],
    "additionalProperties": false,
	"properties": {
        "address": {
            "type": "string"
        },
		"animals": {
            "type": "array",
            "items": {
                "anyOf": [
                    {
                        "$ref": "/parDairyAnimals"
                    },
                    {
                        "$ref": "/parPoultryAnimals"
                    },
                    {
                        "$ref": "/parMeatAnimals"
                    },
                    {
                        "$ref": "/parMixedAnimals"
                    }
                ]
            }
        },
        "crops": {
            "type": "array",
            "items": {
                "anyOf": [
                    {
                        "$ref": "/parArableCrops"
                    },
                    {
                        "$ref": "/parMixedCrops"
                    }
                ]
            }
        }
	}
}
```
<!-- tabs:end -->

<!-- tabs:start -->
#### **parMixed.hbs**
```handlebars
<div class="mixed">
    <h3 class="mixed__title">Address:</h3>
    <address class="mixed__address">{{address}}</address>
    <h3 class="mixed__title">Animals kept:</h3>
    <ul class="mixed__list">
        {{#each animals}}
            <li class="mixed__list-item">
                {{{ dynPartial this._ref this }}}
            </li>
        {{/each}}
    </ul>
    <h3 class="mixed__title">Crops grown:</h3>
    <ul class="mixed__list">
        {{#each crops}}
            <li class="mixed__list-item">
                {{{ dynPartial this._ref this }}}
            </li>
        {{/each}}
    </ul>
</div>
```
#### **parMixedAnimals.schema**
```json
{
	"id": "/parMixedAnimals",
    "$schema": "http://json-schema.org/schema#",
	"type": "array",
    "items": {
        "anyOf": [
            {
                "$ref": "/parChicken"
            },
            {
                "$ref": "/parCattle"
            },
            {
                "$ref": "/parPig"
            },
            {
                "$ref": "/parSheep"
            },
            {
                "$ref": "/parGoat"
            }
        ]
    }        
}
```
#### **parMixedCrops.schema**
```json
{
	"id": "/parMixedCrops",
    "$schema": "http://json-schema.org/schema#",
	"type": "array",
    "items": {
        "anyOf": [
            {
                "$ref": "/parWheat"
            },
            {
                "$ref": "/parBarley"
            },
            {
                "$ref": "/parOats"
            },
            {
                "$ref": "/parPotatoes"
            },
            {
                "$ref": "/parVegetables"
            },
            {
                "$ref": "/parMushrooms"
            }
        ]
    }        
}
```
<!-- tabs:end -->

<!-- tabs:start -->
#### **mixedFarms.schema**
```json
{
	"id": "/mixedFarms",
    "$schema": "http://json-schema.org/schema#",
	"type": "object",
    "additionalProperties": false,
	"properties": {
        "areaName": {
            "type": "string"
        },
        "farms": {
            "type": "array",
            "items": {
                "type": "object",
                "additionalProperties": false,
                "properties": {
                    "name": {
                        "type": "string"
                    },
                    "details": {
                        "$ref": "/parDataFarm",
                        "anyOfType": "parMixed"
                    }
                }
            }
        }
    }        
}
```
#### **mixedFarms.hbs**
```handlebars
<section class="mixed-farms">
    <h1 class="mixed-farms__title">Mixed farms in the {{areaName}} area:</h1>
    {{#each farms}}
        <section class="mixed-farms__farm">
            <h2 class="mixed-farms__farm-title">{{name}}</h2>
            {{> parMixed details}}
        </section>
    {{/each}}
</section>
```
#### **mixedFarms.json**
```json
{
    "areaName": "Eastfarthing",
    "farms": [
        {
            "name": "Bamfurlong",
            "details": {
                "address": "1 Maggot Lane, Marish, Eastfarthing, Shire",
                "animals": [
                    {
                        "$ref": "/parPig"
                    },
                    {
                        "$ref": "/parChicken"
                    }
                ],
                "crops": [
                    {
                        "$ref": "/parPotatoes"
                    },
                    {
                        "$ref": "/parVegetables"
                    },
                    {
                        "$ref": "/parMushrooms"
                    }
                ]
            }
        }
    ]
}
```
<!-- tabs:end -->

Although this is a bit of a contrived/abstract example, it quite nicely illustrates and reinforces what was explained before: `parDataFarm` has multiple sub-schemas which are picked depending on what "type" of schema it is (in our example we showed a `parMixed` version of the generic `parDataFarm`, therefore using `parMixedAnimals` & `parMixedCrops` specific schemas for the `animals` and `crops` properties respectively). It also showed the `parMixed` markup would be used to render the combined generic and specific schema and also the use of the specific schema being used by a parent schema (the `mixedFarms` page).

In the example we haven't even touched on `parDairy`, `parPoultry`, `parMeat`, `parArable` farms which would all have their own, different "generic" schemas to `parMixed`, all using their own markup. These different versions allow for flexible use of a common data structure across projects, with potentially multiple implementations within the same project.

We find this to be useful when dealing with grids and forms.

---

# Grid
A pattern of creating grid layouts became apparent over the course of building many sites in Umbraco using Yuzu and was the first place we implemented the "ofType" structure. By simply allowing specific schemas for row/column configuration and the items within the grid, it enables different grid markup and implementations across projects and indeed within the same project, creating an unchanging, consistent core structure for all grids.

Usually we have two different versions within our projects: one where it is just a "row-based" grid, where there are no columns and each row is just a block, and we have another version which includes columns which contain the blocks.

If you want to get into the detail of how to build/use a "ofType" grid see the ["Building a grid examples" ](definition/examples?id=building-a-grid)

---

# Forms
Creating forms was a common problem across most of our projects and one that we found the "ofType" structure could solve- we needed common, to iterate over form elements, with potentially several versions throughout a site with each one requiring a different selection of inputs and/or styling (and therefore markup).

See the ["Building a form examples"](definition/examples?id=building-a-grid) to learn how to implement forms using "ofType".