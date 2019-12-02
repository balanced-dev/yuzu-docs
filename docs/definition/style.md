# Style
As with most things, people have their preferred way of thinking and operating and like to stick to their methods. Anything that deviates from that often is met with at least some resistance: changing the way you work can be uncomfortable and frankly fruitless exercise if the end result is slower, less understandable or yields no additional benefits even after getting used to it. That's fine- if you've got an efficient way of working that outweighs its downsides, then feel free to keep using it! We're just going to present all our favoured approach, using BEM (Block Element Modifier), which we've found gives us the cleanest output possible and which aligns nicely with our Yuzu ethos while not noticeably slowing our rate of output.

In the past we have built sites using CSS frameworks like Bootstrap, which often rely on using decorator classes. We  always seemed to run into issues however, which would detract from the great inital starting point they provide. They are not without merit and definitely have a time and a place to be used but, when building bespoke sites and once a good way into the project, we would seem to run into specificity clashes (requiring the dreaded `!important` in some cases) and/or muddied markup from the sprinkling of decorator classes tacked onto most elements (with additional classes for custom styles). 

It was because of these problems and general feeling of restriction which drove us to trying BEM and really liking how it felt to use. We found it helped to remove the feeling of, when using helper classes, the actual styling between the markup and stylesheets- you need both to know how it is going to look. A single class per element (ignoring modifiers), a single SCSS file per block, a single point of truth.

While not perfect, it works well with our "block" structure and helped shape the way we work in general! As we have were "componentising" our markup, doing the sense with our styling made sense: a SCSS stylesheet for each block or page, with a unique class name for every element, whilst following a pattern for each meant we could modularise our code, leading to little to no specificity issues, unexpected cascading styles etc. It just makes sense as markup and styles for a block are intrinsically linked- knowing that a block's styling is centralised and won't be being applied anywhere but in its stylesheet greatly reduces confusion.

It's also simple, easy to learn/understand and gives our stylesheets a common structure and layout, which makes going between working on stylesheets not only within a project, but across projects, far easier. Not only that but it makes lifting blocks from other projects and customising them to fit into the current site very easy.

We'll also delve into some of our other preferences, such as how we name & organise our global SCSS files, how we like to write our block stylesheets

---

# Directory Structure
Stylesheets should basically only be stored in 2 main folders:
- for global styles (3rd party CSS stylesheets, your custom SCSS setup etc.) used throughout the project
	```
	project-root
	└── definition.src
		└── _dev
			└── _source
				├── styles
				|	├──	css
				|	|	└── ...
				|	└── scss
				|		└── ...
				└── ...
	```
- or local styles (within blocks/pages).
	
	For example:
	```
	project-root
	└── definition.src
		└── _dev
			└── _templates
				└── src
					├──	_layouts
					|	└── ...
					├──	blocks
					|	└── exampleBlock
					|		├──	_parExampleBlock.scss
					|		└── ...
					└── pages
						└── ...
	```

Within the global SCSS styles (`/definition.src/_dev/_source/scss`) we personally use 3 files for our project:
- 	main.scss:<br>
	Imports all the setup SCSS files (containing mixins, variables etc.)
-	frontend.scss:<br>
	Imports main.scss and also mass imports all block/page SCSS stylesheets (`/definition.src/_templates/src/**/*.scss`) using `gulp-sass-glob`. Used for generating the final CSS file to be used on the live site
-	backoffice.scss:<br>
	Basically the same as frontend.scss but nests the imports in the class `.yuzu-back-office` to be used to style the previews in the Umbraco backoffice
	
How you go on from this starting point of is totally up to you as everyone has their preferences. As a team, we personally like putting project-agnostic SCSS files into a library folder, containing all the common tools, utilities and patterns we've found to work whilst building sites. Things like CSS resets (setting `box-sizing: border-box;` globally etc.), common mixins (media queries, visually hide, clearfix and font sizing etc.) and functions (remification, sizing etc.). These all belong here in our projects, within the "library" directory.

We then have another directory called "project" which unsurprisingly contains all mixins, variables etc. relevant to the current project specifically which will be used throughout the "local" stylesheets.

It works for us, but none of this is provided as every team has their own favourite way of tackling each of these problems and organising their main SCSS setup files.

---

# Gulp Processes
As just discussed, in our build we use `gulp-sass-glob` to mass-import all block and page styling. We do this for a couple of reasons:
- The order of which the SCSS is compiled, shouldn't matter with blocks and pages ("local stylesheets"): none of them should be reliant on each other. The only thing they should be depending on is the global SCSS files for the project which are imported before the local stylesheets.
-  It cuts down on a lot of unecessary overhead: if there's no need for the local stylesheets to be imported in a specific order, why should you manually have to import your SCSS and go through the hassle of maintaining it?

We also use an auto-prefixer (`gulp-autoprefixer`), purely to remove the need for the majority of vendor-prefixed styles, thereby culling a lot of code from stylesheets.

---

# Global vs Local
Basically the philosophy we prescribe to is is that everything outside of the project setup should be kept within the block/page (local) unless it is re-used and becomes a common pattern i.e. the global directory should be as light as possible. 

Code shouldn't preemptively be put into the global- wait until it is needed in multiple places, then create a new mixin/function in global.

---

# Our preferences
## Global SCSS strucutre/naming convention
We came across ITCSS (Inverted Triangle CSS) while searching for a way to give a clear structure to our SCSS files in the global directory, which gave us a basis for our own naming structure and import order. We made little adjustments to make it our own: removed "Components" as they are basically our blocks and "Elements" (as we felt there's little difference between "generic" and "elements"). We're also a little different in the face that the only part of the project which explicitly outputs CSS (apart from blocks/pages) are the "generic" SCSS files- everything else is contained within functions/mixins and are called throughout the project (we'll touch on this later) and so we have a different order to ITCSS.

Import order (first to last)
1.	Generic:<br>
	Contains resets and/or normalise styles, box-sizing definition, etc. but also standardisation/redefinition of bare HTML element default styles e.g. removing margin bottom off all headings and paragraphs

	e.g. `_generic.normalise.scss`
2.	Settings:<br>
	Where variables are defined which are used throughout the projects (fonts, colours, spacing etc.)

	e.g. `_settings.colours.scss`
3.	Tools:<br>
	Globally used mixins and functions, which return very little CSS at all- mainly for things like media queries, sizing etc. Very little in the way of actual styles and more actual logic

	e.g. `_tools.media-queries.scss`
4.	Utilities:<br>
	Contain helper mixins (which would usually be utility/helper classes in other frameworks). Things like clearfix, visually hide etc.

	e.g. `_utility.visually-hide.scss`.

	To differentiate these mixins from tool mixins, our preference is to prefix them with a "u" e.g. `@mixin u-visually-hide`
5.	Objects:<br>
	Mixins used for specific repeating styling instances: for example a specific button/link style, an arrow made out of ::before & ::after, a specific list style etc.

	It's basically a place for common styling patterns, used across multiple blocks, that cannot be its own block.

	These often take parameters so that they can be flexible and customised where used throughout the project.

	e.g. `_object.arrow-icon.scss`

	Our preference is to prefix object mixins with with a "o" e.g. `@mixin o-arrow-icon`



## Use of @mixins
We believe in the use of `@mixin` over helper classes and even `%placeholders` really. We find the potential for repeating code throughout the project to be acceptable because:
1.	Decorator classes are often bundled into sites' CSS, regardless of whether they are used or not, and we find we rarely generate large mixin files. Mixins are also rarely included into more than a handful of blocks each, so the bloat really isn't too bad. If they are not used `@mixins` and `%placeholders` aren't included in the CSS output.
2.	Unlike `%placeholder`s you know exactly where the SCSS will place that selector in the CSS output, removing an potential issues due to order of styles
3.	`@mixins` allow for greater flexibility as they allow parameters to be passed, where you'd otherwise have to create a new class/placeholder
4.	Mixins can be nested in media-queries, whereas placeholders cannot. This means that you either have to create breakpoint specific placeholders or you have to override styles (which can lead to all sorts of issues)

To us the peace-of-mind and adaptability that `@mixins` afford outweigh the potential for CSS bloat that comes with it.

## Alphabetised styles
This is just a team preference but, just so that we have an easy-to-follow order to our styles, we adopted ordering our styles alphabetically. It took some time to sink in, but now it's just second nature and speeds up looking through CSS for a specific declarations. Our selectors never seem to contain enough code for grouping our styles to be beneficial and it was harder to adhere to as everybody had their own way of doing it, which isn't an issue when alphabetising.

`@mixins` however take priority over all the styles within a selector and come first.

## Nesting selectors
We keep everything nested within a single selector: the block's base class. All other selectors are nested within it and concatinate using `&` to build up their class names/modifiers.

For example, say you had the following classes from the markup:
-	`block`
-	`block__header`
-	`block__header-title`
-	`block__header-subtitle`

The stylesheet would look something like this:
```scss
.block {
	background-color: $colour-red;

	&__header {

		&-title {
			@include font-size($font-size-large);
			@include bold-font;
			color: $colour-grey-xlight;
			text-transform: uppercase;
		}

		&-subtitle {
			@include font-size($font-size-large);
			@include light-font;
		}
	}
}
```

Rather than separating it like:
```scss
// Not ideal!
.block {

}

.block__header {

}

.block__header-title {

}
```
or
```scss
// Not ideal!
.block {

}

.block__header {

	&-title {

	}
}
```

Repetition of the block's base class is pointless and just adds more places it needs to be updated if the class name changes. The same line of thinking applies to nesting the other selectors: why repeat youreself and not nest, as it helps to understand the markup better?

This also works for modifiers:
```scss
.block {
	&__header {

		&-title {

		}

		&-subtitle {

		}
	}

	&--modifier-class {

	}
}
```
## Modifiers at end of stylesheet
As a rule we always put our modifiers at the end of the stylesheet/selector as they will likely be overriding things in previous lines above it within the stylesheet

---

# Style Modifiers
When using modifiers we tend to apply them as soon as possible in the markup. In practice this means that most modifiers are applied at the root element of blocks, with the exceptions being when using iterators are used and a modifier is needed on that item.

For example:
<!-- tabs:start -->
#### ** Data (JSON) **
```json
{
	"theme": "dark",
	"roundedCorners": true,
	"listItems": [
		{
			"isImporant": false,
			"text": "List item 1"
		},
		{
			"isImporant": true,
			"text": "List item 2"
		},
		{
			"isImporant": false,
			"text": "List item 3"
		}
	]
}
```
#### ** Handlebars **
```handlebars
<ul class="list list--{{theme}}-theme {{#if roundedCorners}}list--rounded{{/if}}">
	{{#each listItems}}
		<li class="list__item {{#if isImporant}}list__item--important{{/if}}">
			{{text}}
		</li>
	{{/each}}
</ul>
```
#### ** HTML Output **
```html
<ul class="list list--dark-theme list--rounded">
		<li class="list__item ">
			List item 1
		</li>
		<li class="list__item list__item--important">
			List item 2
		</li>
		<li class="list__item ">
			List item 3
		</li>
</ul>
```
<!-- tabs:end -->

This standardises the markup and SCSS: the markup is not littered with conditions for classes save for where absolutely necessary and it means that most modifiers should be at the same place in the stylesheets (both root and nested modifiers are placed at the end of their selectors so that they override previous styles. Root modifiers are nested one level deep within the base class selector, hence making up the final section of the stylesheet).

For the above example with some basic styles would look something like this:
```scss
.list {
    list-style: none;
    margin: 0;
    padding: 25px;
	
	&__item {
        padding: 10px 5px;

        &:not(:last-child) {
            margin-bottom: 10px;
        }

		// Apply styles to important list items
		&--important {
			display: block;
            font-weight: 700;
			text-transform: uppercase;
		}
	}

	&--light-theme {
        background-color: #cccccc;
	}

	&--dark-theme {
        background-color: #111111;
	}

	&--rounded {
        border-radius: 10px;
	}
}
```
We were able to style the important list items easily enough, but how do we make the necessary style changes to nested elements from wthin the root modifier selectors, like the colour themes? Easy: by storing the root class name in a variable, like so:
```scss
.list {
	// Store root class in variable
    $this: '.list';

    list-style: none;
    margin: 0;
    padding: 25px;
	
	&__item {
        padding: 10px 5px;

        &:not(:last-child) {
            margin-bottom: 10px;
        }

		&--important {
			display: block;
            font-weight: 700;
			text-transform: uppercase;
		}
	}

	&--light-theme {
        background-color: #cccccc;

		#{$this} {
			&__item {
                background-color: #ffffff;
                color: #333333;
			}
		}
	}

	&--dark-theme {
        background-color: #111111;
        
        #{$this} {
            &__item {
                background-color: #333333;
                color: #cccccc;
			}
		}
	}

	&--rounded {
        border-radius: 10px;

		#{$this} {
			&__item {
                border-radius: 5px;
			}
		}
	}
}
```

?> 	**Note**: you can actually use the ampersand trick to avoid having to manually assign the class name to `$this`, like so: <br>
	`.list {
	$this: &; // Will contain '.list'
}`<br>
	However won't work on the backoffice in the previews if you are wrapping all your styles in a class as '&' will equal something like `.yuzu-back-office .list` and thus won't work as expected, hence the hardcoding of the class instead.

?> **For further reading about modifiers**: See the [modPartial markup section](definition/markup?id=modPartial)