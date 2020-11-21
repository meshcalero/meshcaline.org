#API Permalinks

Individual resources in a meshcaline API represent steps in a use-case flow of an application instance. As such they are temporary resources. The simple permalink pattern enables API users to keep references to steps from one flow and start new flows from there in subsequent sessions.

Client applications of a meshcaline API must consider all resources accessed in a use case flow as "temporary" resources. The validity / accessibility of each resource is bound to what the API developer defines as appropriate duration for one use-case flow. 

For some resources you explicitly want to allow your users to keep permanent references so that a user can start a new use case in a future session starting from a certain step visited in a previous flow. E.g. in our music API we may want to allow our customer's applications to store list of favorite songs for their end-users lists which those might have discovered in various use case flows. 

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

When you [track your use case flows](/why-hypertext#whatsinhypertextforyourbusiness) through your URI schema you can encode additional information in your bookmark URI (e.g. an identifier of the flow and a timestamp indicating when the bookmark was created). This allows your analytics jobs to associate the new use-case flow initiated through the bookmark with the one that created the bookmark. Requests using such a bookmark URI can then return a temporary redirect to the actual resource URI, and this URL will then encode the identifier of the new flow initiated by the bookmark URL.