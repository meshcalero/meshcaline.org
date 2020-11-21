# Paginateable Collections

Often resources can contain collections of objects, or collections of hypertext controls to other resources, where the size of the collections would make it a bad idea to return the full collection in one API response. Except in cases where it is known upfront that a specific collection can never grow beyond some reasonable size, a **meshcaline** API should use a standardized pattern for paginateable collections. 

This representation then holds at least two elements: the individual items of the first page of the collection in an attribute `items` and an optional hypertext control `next` that points to the next page:

	{
		"name" : "The Clash",
		"popularSongs": {
			"next": "http://....",
			"items": [
				{
					"title" : "London Calling",
					"href": "http://",
					"type": "#song", ...
				},
				{ ...
				}, ...
			] 
		}, ...
	}

Hypertext controls for pagination have a resource type `#implied` and a method `GET` and therefore the controls can always use the simplified URI string representation. Following such a link does return the resource type implicitly defined from the link's context. In our example it contains the next chunk of items of the `popularSongs` collection:

	{
		"next": "http://....",
		"items": [
			{
				"title" : "Rock the Casbah",
				"href": "http://",
				"type": "#song", ...
			}, ...
		] 
	}

The end of the collection is reached, when there is no more `next` attribute.

The specific representations of the `items` attribute may vary, depending on the type of collection (Maps, Lists, Sets).

A **meshcaline** API may provide additional attributes in their collection representation. The usual suspects include:

* the total number of available items, 
* some offset of the first item in the current page in the whole a collection, or
* additional hypertext controls for direct access to the previous, first or last page of a collection.

You better think twice before you introduce such convenience features: Your API users will expect a consistent set of collection attributes for all collections in your API. And while you may initially think this is not too hard to achieve, there exist classes of collections where this is not simple, if not impossible: Assume we want to add a list of next concerts to our artist representation, but not having such data yourself you rely for that feature on a ticket supplier's API that only allows you to fetch the next 10 gigs after a given date. Then you would neither know how many concerts will be available in total, nor can you easily build a `prev` link, unless you encode somehow the dates of all previously visited pages in the URI. A `last` link would even require you to iterate the 3rd party API until you get the first response with less than 10 results back. Therefore it is best practice to keep the pagination representation minimal unless you learn from your users that more complex pagination constructs are really required for the application use cases your API shall support.