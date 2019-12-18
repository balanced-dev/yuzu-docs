# Javascript Interactions
Javascript is the area in which Yuzu is the least prescriptive- it's often where frontend developers need to make the decision themselves on what the best way of approaching this is.

For example, many brochure sites don't require much in the way of JS- maybe some very basic, framework-less scripts to just toggle some classes will suffice. Perhaps no Javascript is needed at all or maybe, despite it being considered 'outdated' by many, jQuery would be best so you can leverage some of the vast array of plugins available (e.g. a carousel).

We are not against using Angular, Vue or any other framework as we have occasionally used them on some sites to fulfil specific roles in specific areas. It's about choosing the right tool for the job.

Despite this, we have a basic API and have provided a few examples of a few ways of writing JS in a Yuzu environment to help you get stated 

---

# API
As of the time of writing, our API only has one endpoint (`getResolved`). It returns the compiled HTML & the data of a given state, which allows us to mimic our Handlebars block partials being rendered on the backend (for AJAX calls returning compiled HTML) or an API call to return JSON respectively.

`/api/getResolved/{state file name}`

`{state file name}` is just the name of your JSON file for a block/page state without the `.json` extention. Directory structure is irrelevant.

For example if we had a file located at `_dev/_templates/src/blocks/_homepage/carousel/data/carousel_no-slides.json`, to retrieve the HTML and JSON purely for that state we'd call `/api/getResolved/carousel_no-slides`

For the HTML we'd use the response object's `text` property, whereas for the data we'd use `json`. 

---

# Our preferences
-   We prefer to use data attributes over JS prefixed classes as they're more obvious in the markup, more flexible if data as you can assign values to them and keeps classes for styling only
-   We tend to pass most variables into the JS via a JSON object being passed through a data attribute, picked up and used in our scripts

    For example:
    ```handlebars
    <header 
        data-header-settings='{
            "toggleClass": "site-header--is-open"
        }'
    >
        ...
    </header>
    ```
-   Usually pass an endpoint URL to the markup via a property in the JSON, allowing for the backend to be mimicked by using our state API which can be used and overwritten by the delivery side in production.

---

# Examples

## Stimulus
Without going into it too much, Stimulus is a modest, small, JavaScript framework for use with server side rendered html. It revolves around just three main concepts: controllers, actions, and targets.

Let's create a basic and very simplified example: a button to load comments on an article.

We'll include the Stimulus app setup & the comments controller in the same JS file to simplify for this example and assume you already added the Stimulus package to the page already:

<!-- tabs:start -->
#### **parCommentsSection.hbs **
```handlebars
<div class="comment-section"
    data-controller="comments"
    data-comments-settings='{
        "commentsListUrl": "{{commentsListEndpoint}}"
    }'
>
    <button data-action="click->comments#loadComments">{{buttonText}}</button>
    <div class="comment-section__comments" data-target="comments.list">
        {{!-- parCommentList will be injected here --}}
    </div>
</div>
```
#### **parCommentsSection.json **
```json
{
    "buttonText": "Load Comments",
    "commentListEndpoint": "/api/getPreview/parCommentsList"
}
```
##### **comments.js**
```javascript
const StimulusApp = Stimulus.Application.start();

StimulusApp.register('comments', class extends Stimulus.Controller {
	static get targets() {
		return [
			'list'
		];
	};

	initialize() {
        this.settings = JSON.parse(this.data.get('settings'));
    }

    loadComments() {
        fetch(this.settings.commentListUrl)
			.then(response => response.text())
			.then((html) => this.updateListHtml(html));
	}

	updateListHtml(htmlContent) {
		this.listTarget.innerHTML = htmlContent;
	}
}); 
```
<!-- tabs:end -->

<!-- tabs:start -->
##### **parCommentsList.hbs**
```handlebars
<ul class="comments-list">
    {{#each items}}
        <li class="comments-list__comment">
            {{#if headshot.src}}
                <img class="comments-list__comment-image" src="{{headshot.src}}" alt="{{headshot.alt}}"/>
            {{/if}}
            <div class="comments-list__comment-body">
                {{#if name}}
                    <h3 class="comments-list__comment-name">{{name}}</h3>
                {{/if}}
                {{#if timestamp}}
                    <div class="comments-list__comment-datetime">{{timestamp}}</div>
                {{/if}}
                {{#if comment}}
                    <p class="comments-list__comment-text">{{comment}}</p>
                {{/if}}
            </div>
        </li>
    {{/each}}
</ul>
```
##### **parCommentsList.json**
```json
{
    "items": [
        {
            "headshot": {
                "src": "/_client/images/person_1.jpg",
                "alt": ""
            },
            "name": "Jean Doe",
            "timestamp": "January 9, 2018 at 2:21pm",
            "comment": "Lorem ipsum dolor sit amet, consectetur adipisicing elit. Pariatur quidem laborum necessitatibus, ipsam impedit vitae autem, eum officia, fugiat saepe enim sapiente iste iure! Quam voluptas earum impedit necessitatibus, nihil?"
        }
    ]
}
```
<!-- tabs:end -->

<!-- tabs:start -->
#### **HTML output when button pressed**
```html
<div class="comment-section"
    data-controller="comments"
    data-comments-settings='{
        "commentsListUrl": "{{commentsListEndpoint}}"
    }'
>
    <button data-action="click->comments#loadComments">Load Comments</button>
    <div class="comment-section__comments" data-target="comments.list">
        <ul class="comments-list">
            <li class="comments-list__comment">
                <img class="comments-list__comment-image" src="/_client/images/person_1.jpg" alt="">
                <div class="comments-list__comment-body">
                    <h3 class="comments-list__comment-name">Jean Doe</h3>
                    <div class="comments-list__comment-datetime">January 9, 2018 at 2:21pm</div>
                    <p class="comments-list__comment-text">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Pariatur quidem laborum necessitatibus, ipsam impedit vitae autem, eum officia, fugiat saepe enim sapiente iste iure! Quam voluptas earum impedit necessitatibus, nihil?</p>
                </div>
            </li>
        </ul>
    </div>
</div>
```
<!-- tabs:end -->



## Vue.js
This Vue example again is as simple as we could make it, removing any of the setup purely to illustrate how our API can be used to return JSON and how Vue can easily be applied to a section of sites using Yuzu.

<!-- tabs:start -->
#### **parGallery.hbs**
```handlebars
<div id="gallery-app">
    <gallery data='{{{toString this}}}'></gallery>
</div>
```
#### **gallery-vue-app.js**
```javascript
Vue.component('gallery', {
    data: function () {
        return {
            photos: [],
            nextPageEndpoint: ""
        };
    },
    props: ['data'],
    created: function() {
        this.fetchInitial();
    },
    methods: {
        fetchInitial: function() {
            let initialView = JSON.parse(this.$props.data);
            this.updatePhotos(initialView);
        },
        fetchNextPage: function() {
            fetch(this.nextPageEndpoint)
                .then(response => response.json())
                .then(this.updatePhotos);
        },
        updatePhotos: function(response) {
            this.nextPageEndpoint = response.nextPageEndpoint;
            this.photos = [...this.photos, ...response.photos];
        }
    },
    template: `
        <section class="gallery">
            <template v-for="photo in photos">
                <template v-if="photo.image.src">
                    <img class="gallery__photo" :src="photo.image.src" :alt="photo.image.alt"/>
                </template>
            </template>
            <button class="gallery__more-btn" v-if="nextPageEndpoint" v-on:click="fetchNextPage">
                {{buttonText}}
            </button>
        </section>
    `
});

var teacherProfiles = new Vue({
    el: '#gallery-app'
});
```
#### **parGallery.json**
```json
{
    "photos": [
        {
            "image": {
                "src": "/_client/images/person_1.jpg",
                "alt": ""
            }
        },
        {
            "image": {
                "src": "/_client/images/person_2.jpg",
                "alt": ""
            }
        },
        {
            "image": {
                "src": "/_client/images/person_3.jpg",
                "alt": ""
            }  
        } 
    ],
    "nextPageEndpoint": "/api/getResolved/parTeacherProfiles_1",
    "buttonText": "View more photos"
}
```
#### **parGallery_1.json**
```json
{
    "photos": [
        {
            "image": {
                "src": "/_client/images/person_2.jpg",
                "alt": ""
            }
        },
        {
            "image": {
                "src": "/_client/images/person_3.jpg",
                "alt": ""
            }
        },
        {
            "image": {
                "src": "/_client/images/person_1.jpg",
                "alt": ""
            }  
        } 
    ],
    "nextPageEndpoint": "/api/getResolved/parTeacherProfiles_2",
    "buttonText": "View last photos"
}
```
#### **parGallery_2.json**
```json
{
    "photos": [
        {
            "image": {
                "src": "/_client/images/person_3.jpg",
                "alt": ""
            }
        },
        {
            "image": {
                "src": "/_client/images/person_1.jpg",
                "alt": ""
            }
        },
        {
            "image": {
                "src": "/_client/images/person_2.jpg",
                "alt": ""
            }  
        } 
    ],
    "nextPageEndpoint": "",
    "buttonText": ""
}
```
<!-- tabs:end -->

This very simple example simply loads the next batch or page of photos, with the next endpoint being updated after every call (new URL with a query string to get pagination working on backend) until the endpoint URL is falsy (i.e. there are no more photos), whilst updating the "load more" button's text.

Alternatively we could, instead of putting the markup within the JS, embedded it in the Handlebars file to keep the Vue app cleaner.

We achieve this by simply changing the `parGallery.hbs` and `gallery-vue-app.js` files accordingly (using a `<script type="x-template">` in the Handlebars and changing the template to point to `#gallery-template` in the JS):
<!-- tabs:start -->
#### **parGallery.hbs**
```handlebars
<div id="gallery-app">
    <gallery data='{{{toString this}}}'></gallery>
</div>

<script type="x-template" id="gallery-template">
    <section class="gallery">
        <template v-for="photo in photos">
            <template v-if="photo.image.src">
                <img class="gallery__photo" :src="photo.image.src" :alt="photo.image.alt"/>
            </template>
        </template>
        <button class="gallery__more-btn" v-if="nextPageEndpoint" v-on:click="fetchNextPage">
            \{{buttonText}}
        </button>
    </section>
</script>
```
#### **gallery-vue-app.js**
```javascript
Vue.component('gallery', {
    data: function () {
        return {
            photos: [],
            nextPageEndpoint: "",
            buttonText: ""
        };
    },
    props: ['data'],
    created: function() {
        this.fetchInitial();
    },
    methods: {
        fetchInitial: function() {
            let initialView = JSON.parse(this.$props.data);
            this.updatePhotos(initialView);
        },
        fetchNextPage: function() {
            fetch(this.nextPageEndpoint)
                .then(response => response.json())
                .then(this.updatePhotos);
        },
        updatePhotos: function(response) {
            this.nextPageEndpoint = response.nextPageEndpoint;
            this.buttonText = response.buttonText;
            this.photos = [...this.photos, ...response.photos];
        }
    },
    template: '#gallery-template'
});

var teacherProfiles = new Vue({
    el: '#gallery-app'
});
```
<!-- tabs:end -->

Now you may have noticed that we've had to escape our double curly braces within the button, whereas we didn't in our previous example (`\{{buttonText}}`). This is because Handlebars needs to be prevented from running it, which wasn't an issue in before with the template being within the JS file.

We can fix this by adding our own custom delimiters to the component in the JS and using them in the Handlebars file like so:
<!-- tabs:start -->
#### **parGallery.hbs**
```handlebars
<div id="gallery-app">
    <gallery data='{{{toString this}}}'></gallery>
</div>

<script type="x-template" id="gallery-template">
    <section class="gallery">
        <template v-for="photo in photos">
            <template v-if="photo.image.src">
                <img class="gallery__photo" :src="photo.image.src" :alt="photo.image.alt"/>
            </template>
        </template>
        <button class="gallery__more-btn" v-if="nextPageEndpoint" v-on:click="fetchNextPage">
            [[buttonText]]
        </button>
    </section>
</script>
```
#### **gallery-vue-app.js**
```javascript
Vue.component('gallery', {
    data: function () {
        return {
            photos: [],
            nextPageEndpoint: "",
            buttonText: ""
        };
    },
    props: ['data'],
    created: function() {
        this.fetchInitial();
    },
    methods: {
        fetchInitial: function() {
            let initialView = JSON.parse(this.$props.data);
            this.updatePhotos(initialView);
        },
        fetchNextPage: function() {
            fetch(this.nextPageEndpoint)
                .then(response => response.json())
                .then(this.updatePhotos);
        },
        updatePhotos: function(response) {
            this.nextPageEndpoint = response.nextPageEndpoint;
            this.buttonText = response.buttonText;
            this.photos = [...this.photos, ...response.photos];
        }
    },
    template: '#gallery-template',
    delimiters: ['[[', ']]']
});

var teacherProfiles = new Vue({
    el: '#gallery-app'
});
```
<!-- tabs:end -->