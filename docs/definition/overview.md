# Overview
One of Yuzu's core aims is to make creating websites as simple and therefore as maintainable as possible and affects the majority of decisions made while developing the "Definition" side of Yuzu. 

It has organically grown and been refined over years of daily use while trying not to become a prescriptive, all-encompassing solution. This documentation will outline what we have found works for us and keeps headaches to a minimum when building anything from the humblest and most basic of CMS sites, to large, complex e-commerce websites.

Having only fully concentrated on Yuzu for a few short months (it's mainly taken a backseat to client work), with a tiny team, it definitely isn't perfect and there are still things we are learning and improving. There will be situations which we haven't accounted for and subtleties we haven't discovered but, in the main, we believe this is a fantastic way of working and hope you do too!

# Basics of a Block
We find dividing sites up into components, which we call "blocks", really help to keep our solutions as small, scalable and simple as possible. By identifying repeating elements and data from wireframes/designs and producing a list of named blocks you can often condense a seemingly complex, sprawling website down to a very manageable selection of blocks.

Each block is made up of 4 key components:

1. [**Schema-** ](definition/schema "Read more about schemas")<br>
	The "blueprint" for the block's data, outlining all the properties and variations a block can contain. It's arguably the most important component as it's the contract between the front-end and the back-end of a site.

	<!-- tabs:start -->
	#### ** parPerson.schema **
	``` json
	{
		"id": "/parPerson",		
    	"$schema": "http://json-schema.org/draft-04/schema#",
		"type": "object",
		"properties": {
			"fullname": {
				"type": "string"
			},
			"photos": {
				"type": "array",
				"items": {
					"type": "object",
					"properties": {
						"src": {
							"type": "string"
						},
						"alt": {
							"type": "string"
						}
					}
				}
			}
		}
	}
	```
	<!-- tabs:end -->

2.  [**Data (JSON)-** ](definition/data "Read more about data")<br>
	Contained in JSON files, data allows the creation of states, enabling blocks to be tested with different/extreme data situations (e.g. the absence of a property, a long string in a property, different numbers of items in an array) without a backend solution. 

	This allows for potential issues that varying data scenarios could create in the design to be raised and worked-around prior to integration.

	<!-- tabs:start -->
	#### ** parPerson_no-photos.json**
	``` json
	{
		"personName": "Guy Incognito",
		"photos": []
	}
	```

	#### ** parPerson_long-name.json**
	``` json
	{
		"personName": "Charles Philip Arthur George Mountbatten-Windsor",
		"photos": [
			{
				"src": "https://upload.wikimedia.org/wikipedia/commons/d/db/Charles_Prince_of_Wales.jpg",
				"alt": "Prince Charles"
			}
		]
	}
	```
	<!-- tabs:end -->


3. [**Styling (SCSS)-** ](definition/style "Read more about styling")<br>
	We personally like to use BEM and use a stylesheet per block, thereby effectively scoping our styles via the block name, reducing both CSS bleed and the need for the dreaded `!important`.

	<!-- tabs:start -->
	#### **_parPerson.scss**
	``` scss
	.person {

		&__person-name {
			font-size: 1rem;
		}

		&__image {
			display: block;
		}
	}
	```
	<!-- tabs:end -->


4. [**Markup (Handlebars)-** ](definition/markup "Read more about markup")<br>
	We use Handlebars as it uses simple syntax to render our data and covers most of the scenarios we've encountered. Where we have found it wanting we've added a very small selection of helpers to help bridge the gap.

	One of our beliefs is that the front-end to projects should be as lightweight as possible and shouldn't doing any heavy-lifting. Handlebars is purely for rendering data and should not be seen as anything more. By not being "intelligent" (unable to do calculations, complex conditions etc.) it forces the markup to be simple and easy for even fledgling developers to understand, follow and write!

	<!-- tabs:start -->
	#### **parPerson.hbs**
	``` handlebars
	<div class="person">
		{{#if personName}}
			<h2 class="person__person-name">{{personName}}</h2>
		{{/if}}
		{{#each photos}}
			<img class="person__image" src="{{src}}" alt="{{alt}}">
		{{/each}}
	</div>
	```
	<!-- tabs:end -->

## Block files
The anatomy of a typical block:
```
person
├── data
│   ├── parPerson.json
│   ├── parPerson_no-photos.json
│   └── parPerson_long-name.json
├── _parPerson.scss
├── parPerson.hbs    
└── parPerson.schema    
```
---

# Directory Structures
At the root of the project folder we have two directories `definition.src` and `delivery.src`. Of course we'll be purely concentrating within the definition folder for now.

```
project-root
├── definition.src
│   └── ...
└── delivery.src    
```

Within the definition directory we have our `node_modules`, our `gulpfile.js` and `_dev` directories as well as our `package.json` file. 

`package.json` and `node_modules` are obviously for our npm packages, whereas `gulpfile.js` contains all the files relating to the gulp build processes.

Virtually all of the time spent while developing a project however occurs in the `_dev` directory.
```
project-root
└── definition.src
    ├── _dev
	│   └── ...
    ├── gulpfile.js
	│   └── ...
    ├── node_modules
	│   └── ...
	└── package.json
```

Within the `_dev` folder is:
- `_client`: stores all compiled HTML, CSS & JS and any media files, font files etc. copied from the `_source` & `_templates` folders, within their own sub-directories. There's no real need to ever go into this directory when developing
- `_source`: contains all the global .scss files, JS files, fonts, images etc. within separate sub-directories
- `_templates`: where the majority of developers' time will be spent as it contains all the block and page markup, data, schemas and local styling
- `yuzu-def-ui`: holds the Yuzu Definition User Interface app which allows in-browser navigation around the project, editing/creating new states etc.
- `templates.html`: basically the index page for the front-end solution. Just injects the Yuzu Definition UI into the page as well as providing links to documentation and other 
```
project-root
└── definition.src
    └── _dev
		├── _client
		│   └── ...
		├── _source
		│   └── ...
		├── _templates
		│   └── ...
		├── yuzu-def-ui
		│   └── ...
		└── templates.html
```

The `_templates` directory contains the real bulk of the definition side as it holds:
- `_dataStructures`: schema files containing common/recurring data patterns 
- `_layouts`: contain the possible layouts a page (or block when viewed in isolation) can use, with a layout effectively being a wrapper for page content (header, footer, site nav as well as the html head and body tags, links to compiled stylesheets etc.)
- `blocks`: hold all the components which a page can use- grids, forms, layout blocks like site header, footer and navigation, as well as just general content blocks
- `pages`: combine blocks and layouts to generate fully formed webpages

```
project-root
└── definition.src
    └── _dev
		└── _templates			
			├── _dataStructures
			|	└── ...
			├── _layouts
			|	└── ...
			├── blocks
			|	└── ...
			└── pages
				└── ...
			
``` 

<!-- ---

# Block Areas -->


---

# Modules Overview

## Yuzu Modules

| Yuzu Module      				| Purpose 			|
| ----------------------------- | -----------------	|
| yuzu-definition-api 			| 					|
| yuzu-definition-core 			| 					|
| yuzu-definition-hbs-helpers 	| 					|
| yuzu-definition-ui 			| 					|

## Other modules

| Module        				| Purpose 			|
| ----------------------------- | -----------------	|
| ansi-colors 					| 					|
| browser-sync 					| 					|
| del 							| 					|
| fancy-log 					| 					|
| gulp 							| 					|
| gulp-autoprefixer 			| 					|
| gulp-clean-css 				| 					|
| gulp-flatten 					| 					|
| gulp-load-plugins 			| 					|
| gulp-plumber 					| 					|
| gulp-rename 					| 					|
| gulp-sass 					| 					|
| gulp-sass-glob 				| 					|
| gulp-sourcemaps 				| 					|
| gulp-strip-css-comments 		| 					|
| path 							| 					|
| prettyjson 					| 					|

---

# Gulp tasks

| Command			| Alias				| Description 			|
| -----------------	| ----------------- | ---------------------	|
| gulp ui			| npm run serve		|						|
| gulp buildUi		| npm run build		|						|
| gulp dist			| npm run dist		|						|
| gulp showPaths	| 					|						|

<!-- ---
# When to block? -->