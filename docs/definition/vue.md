# Vue

Mixing Vue into handlebars files works, but it has its downsides. The similarity between Vue and Hbs syntax makes it easy to transition between the two. But, when using them both together a prefix on the property notation (\{{propertyName}}) is needed to distinguish between client and server side rendering. We found that this leads to confusion when editing templates, making it harder to work on projects.

We decided to separate the rendering of Hbs and Vue files into two template file types. Each block can either be a Hbs or Vue block and Vue files are stored in blocks as .html files. 

Here is an example of a Vue block

<!-- tabs:start -->
#### ** Vue Init **

``` js
const parVue = new Vue({
    el: '.parVue',
    data: {
        message: ''
    },
    async mounted() {
        await blockResolver(this);
    }
})
```

#### ** Vue **

``` Vue
<div class="parVue">
    <p>{{ message }}</p>
</div>
```

<!-- tabs:end -->

The vue setup js file above initialises the Vue app at the Dom entry point and from here everything is Vue (including components). 

The data property on the Vue options is an empty json data file that matches the top-level schema properties, Vue uses this as placeholder properties so it knows which properties to expect when rendering the app. Unfortunately, this is a duplication of properties from the schema and json files, there is no reliable way to pull this in, and if it doesn't match data may not render correctly. 

## Vue initial state

In the Vue function `mounted` above, the global function blockResolver is called, which hydrates the initial state of the Vue app at startup. There are two ways for Vue to get its initial state data. 

- endpoint : the json state data has a property of _endpoint which points to an API endpoint. On server side render the data attribute `endpoint` is added to the block root element and the Vue app uses this to bootstrap the app with the data. 
- context : the data context is passed from a parent, the data is written into a data attribute 'context' at the block root element. The vue app serialises this data to bootstrap the app. 

## Transitioning from Hbs to Vue

A special handlebars helpers exists to render out child blocks without processing taking place.

<!-- tabs:start -->
#### ** Handlebars **

``` Hbs
{{{staticPartial 'parVue'}}}
```

<!-- tabs:end -->

As hinted at above you can also pass context down into a Vue partial

<!-- tabs:start -->
#### ** Handlebars **

``` Hbs
{{{staticPartial 'parVue' data}}}
```

<!-- tabs:end -->

## Vue Components

Yuzu supports Vue components as Yuzu blocks with props for passing data form parent Vue blocks.

<!-- tabs:start -->
#### ** Vue Init **

``` js
Vue.component('vue-component', {
    template: '#vueComponent',
    props: ['list']
});
```

#### ** Vue **

``` Vue
<aside>
    <ol>
        <li v-for="l in list">
            {{l.title}}
        </li>
    </ol>
</aside>
```
<!-- tabs:end -->

We have found the best way to register components is through `text/x-template` definitions in the layout files.

<!-- tabs:start -->
#### ** Handlebars **

``` Hbs
<script type="text/x-template" id="vueComponent">
    {{{staticPartial 'parVueComp'}}}
</script>
```

<!-- tabs:end -->