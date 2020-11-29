#API Permalinks

Client applications of a *meshcaline*{: .m} API must consider all resources accessed during a business process as "temporary" resources. The validity / accessibility of each resource is bound to what the API developer defines as appropriate duration for one business process. 

But business processes are usually not strictly isolated concepts. Often one business process (e.g. searching for some object) leads to some resource (e.g. the object searched for) from where you can start new business processes (e.g. update the object). As a result you might have resources for which you want to allow your application developers to keep permanent references so that their users can start a new business process in a future session from that resource. E.g. in our music API we may want to allow our customer's applications to store lists of favorite songs for their end-users lists which those might have discovered in various business processes (e.g. a search business process, recommendation business processes, etc.). 

In those cases the response representation contains a hypertext control with link-relation [`bookmark`](http://www.w3.org/TR/html5/links.html#link-type-bookmark). Those hypertext controls will usually use the simplified URI-string representation: 

	{
		 "bookmark" : "http://..",
	     "title": "London Calling",
	     "interpreter": {
	          "name": "The Clash",
	          "href": "http:/....",
	          "type": "#artist",
	           ...
	     },  ...
	}

You should not introduce permalinks for every resource, but only for those where you have a reason. With permalinks you give a long term promise not only to the availability of the resource, but also to the addressing schema used in the URI. While you don't have to disclose the addressing schema directly, you still must ensure that all subsequent iterations of your offering will still understand all addressing schemas ever used for bookmarks. 

When you [track your business process steps](/hypertext#what-does-hypertext-bring-to-your-business) through your URI schema you can encode additional information in your bookmark URI (e.g. an identifier of the flow and a time-stamp indicating when the bookmark was created). This allows your analytics jobs to associate the new use-case flow initiated through the bookmark with the one that created the bookmark. Requests using such a bookmark URI can then return a temporary redirect to the actual resource URI, and this URL will then encode the identifier of the new business process initiated by the bookmark URL.