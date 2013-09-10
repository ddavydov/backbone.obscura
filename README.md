
# Backbone.Obscura

Backbone.Obscura is a read-only proxy of a Backbone.Collection that can easily be 
filtered, sorted, and paginated, while still implementing all of the read-only
Backbone.Collection methods. As the underlying collection is changed the proxy 
is efficiently kept in sync, taking into account all of the transformations. All
transformations can be modified or removed at any time.

You can pass the proxy into a Backbone.View and let Backbone.Obscura take care of 
the filtering or paginating logic, leaving your view to only re-render itself as 
the collection changes. This keeps your views simple and [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself).
This works particularly well with [Marionette's](https://github.com/marionettejs/backbone.marionette)
[CollectionView](https://github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.collectionview.md) 
and [CompositeView](https://github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.compositeview.md).

```javascript
var proxy = new Backbone.Obscura(originalCollection);

// Set the transformations on the original collection
proxy
  .setPerPage(25)
  .setSort('age', 'desc')
  .filterBy(function(model) {
    return model.get('age') > 17 && model.get('age') < 70;
  });

// Read-only Backbone.Collection functions work on the transformed proxy
proxy.toJSON();
proxy.pluck('age');
proxy.at(3);
proxy.first();

// 'add', 'remove', 'change', 'reset' events are all forwarded for models in the proxy
proxy.on('add', function() { /* ... */ });

// Pass the proxy to a view that knows how to react to a changing collection
var view = new CollectionView({ collection: proxy });

// In another view or a controller, you can modify the state of the filters
if (proxy.hasNextPage()) {
  proxy.nextPage();
}

$('button').on('click', function() {
  proxy.reverseSort();
});
```

[![Build Status](https://secure.travis-ci.org/jmorrell/backbone.obscura.png?branch=master)](http://travis-ci.org/jmorrell/backbone.obscura)

### What's with the name?

![Logo](https://raw.github.com/jmorrell/backbone.obscura/master/img/CameraObscura.jpg)

The camera obscura is an optical device that projects an image of its surroundings 
on a screen. In a similar way, we are using a crude projection of the original
collection to "draw" our views.

## Use Cases

* You want to move pagination, filtering code out of your views
* You have several ways of filtering down an in-memory collection (find-as-you-type, value ranges) that all need to work together
* You have a collection that might be updated via push and need a filtered view to react to this change
* You have a centralized place where you manage your app's data, and need to visualize part of this data
* You want to have an additional view that presents a different view on the same data (Top 5, Latest)

## Methods

This library is effectively a convenience wrapper around [backbone-filtered-collection](https://github.com/jmorrell/backbone-filtered-collection),
[backbone-sorted-collection](https://github.com/jmorrell/backbone-sorted-collection), 
and [backbone-paginated-collection](https://github.com/jmorrell/backbone-paginated-collection).

### new Backbone.Obscura(collection [, options])

Initialize a new Obscura collection by passing in the original collection.

```javascript
var proxy = new Backbone.Obscura(originalCollection);
```

You may also optionally pass an options hash. Currently the only supported option is setting the `perPage` setting of the paginated transform.
```javascript
var proxy = new Backbone.Obscura(originalCollection, { perPage: 25 });
```

#### proxy.superset()

Return a reference to the original collection.

```javascript
proxy.superset();
```

#### proxy.filterBy([filterName], filter)

Apply a new filter to the set. Takes an optional filter name.

Can be a simple object that defines required key / value pairs.
```javascript
filtered.filterBy('foo and bar filter', { foo: 2, bar: 3 });
```

Or the you can pass a filter function instead of a value.
```javascript
filtered.filterBy('a > 2', { a: function(val) { 
  return val > 2;
}});
```

Or you can use an arbitrary filter function on the model itself.

```javascript
filtered.filterBy('age', function(model) {
  return model.get('age') > 10 && model.get('age') < 40;
});
```

#### proxy.removeFilter(filterName)

Remove a previously applied filter. Accepts a filter name.

```javascript
proxy.removeFilter('age');
```

#### proxy.resetFilters()

Removes all applied filters. After the collection should be the same as the superset.

```javascript
proxy.resetFilters();
```

#### proxy.refilter()

If the collections get out of sync (ex: change events have been suppressed) force
the collection to refilter all of the models.

```javascript
proxy.refilter();
```

Can also be forced to run on one model in particular.

```javascript
proxy.refilter(model);
```


## Alternative Libraries

There are several libraries that offer similar functionality, but none that offered the combination of features that I wanted.

* JavaScript, not CoffeeScript
* No need to define filters or sorting on initialization
* Ability to use arbitrary functions for filters or sorting
* Transparency, if no transforms are defined, the proxy should be the same as the original collection
* Ability to add and remove multiple filters
* Easy to use with Browserify, but also easy to throw into an AMD project

If this library doesn't meet your needs, maybe one of the following will:

* [Backbone.Projections](https://github.com/andreypopp/backbone.projections)
* [backbone.collectionsubset](https://github.com/anthonyshort/backbone.collectionsubset)
* [Backbone.VirtualCollection](https://github.com/p3drosola/Backbone.VirtualCollection)
* [Backbone.Subset](https://github.com/masylum/Backbone.Subset)

## Installation

### Usage with Browserify

Install with npm, use with [Browserify](http://browserify.org/)

```
> npm install backbone.obscura
```

and in your code

```javascript
var Obscura = require('backbone.obscura');
```

### Usage with Bower

Install with [Bower](http://bower.io):

```
bower install backbone.obscura
```

The component can be used as a Common JS module, an AMD module, or a global.

### Usage as browser global

You can include `backbone.obscura.js` directly in a script tag. Make 
sure that it is loaded after underscore and backbone. It's exported as 
`Backbone.Obscura`.

```HTML
<script src="underscore.js"></script>
<script src="backbone.js"></script>
<script src="backbone.obscura.js"></script>
```

## Testing

Install [Node](http://nodejs.org) (comes with npm) and Bower.

From the repo root, install the project's development dependencies:

```
npm install
bower install
```

Testing relies on the Karma test-runner. If you'd like to use Karma to
automatically watch and re-run the test file during development, it's easiest
to globally install Karma and run it from the CLI.

```
npm install -g karma
karma start
```

To run the tests in Firefox, just once, as CI would:

```
npm test
```

## License

MIT

## Acknowledgements

[Photo](http://www.flickr.com/photos/garunaborbor/8787779395) taken from Flickr, licensed under Creative Commons

[Photo](http://commons.wikimedia.org/wiki/File:Camera_Obscura_box18thCentury.jpg) taken from Wikimedia, and is in the public domain