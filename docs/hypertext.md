# Why Hypertext API?

Although everyone these days is familiar with hypertext when browsing the web, in API hypertext formats are still quite uncommon. While most developers are used to URI as data-elements for accessing static resources (like jpeg photos), they are still new to using hypertext controls as mechanism to access the individual resources of an API. Also tool support for hypermedia controls in "web frameworks" is often still only limited.

There still seems to be a need to convince both, client and server side developers (and their tool-producers) of the advantages of hypertext driven API.

##Why should API users want a hypertext API?

Most developers are used to some hard coded URI templates in their application to construct the actual URI required to access an API resource. So many of them will argue that constructing the URI from values collected in the application context is a well understood problem with plenty of handy solutions available. So you better don't expect to get frenetic applause when you tell them that you now serve those much more convenient, ready to use hypermedia URI with the response. Best reaction you could expect is that your users don't care. But some might even dislike this, because their favorite "REST toolkit" expects URI templates for specific types of resources and doesn't work well with individual links per request. 

So be prepared that you need some convincing arguments!

###Changable resource addressing schema
The most obvious advantage of hypertext is the ability to change the resource addressing schema at any time. Unfortunately this looks like a very one-sided advantage for you as the provider of the API. From your user's point of view, they are not really interested in you being able to change your resource addressing schema. And it is hard to argue against users when they say that it should not be too difficult for an API provider to maintain legacy addressing schemas even when the underlying implementation dramatically changes. Worst case you have to (internally) map your legacy addressing schema to the most recent implementations. This argument misses one important aspect of the addressing schema: the domain name of the API request. As API evolve and grow, you might come to the point where you must apply some partitioning / sharding to your API. Either because you no longer can make all data available in your service efficiently available to all nodes, or because you realize that you must run different parts of your API on different machines. You could build some routing service in front of your API to overcome that problem, but not only does this additional network hub add unnecessary milliseconds to your processing time (and usually you do partitioning for performance reasons), it also introduces a new potential bottleneck in your architecture. Latest when you want to change networks for some parts of your API (e.g. when you want to run some aspects of your API through some CDN), the option for a routing service is gone, as it would kill the advantage of the partitioning. 

###Application state in stateless service
Even if you believe that you never have to partition you service, there is another important advantage that hypertext API enables: having application state in a stateless API implementation. Whoever had implemented a service that has to handle substantial load knows that things can get difficult as soon as you have maintain state specific to one client's flow though your service's use case. If you want to maintain that state in your server side implementation, then out of a sudden you need non-trivial mechanisms that ensure that all nodes in your cluster have access to that state-information. And once you have to plan for fail-over scenarios to other clusters / data centers things get really tricky. When you instead use hypertext, you get a nice, 100% fault-tolerant storage for the state information for free: the URI. While (practically) limited in size (although HTTP defines no size-restrictions for URI, some archaic, but still popular HTTP clients still have size constraints that limit for a general purpose API the URI to 2k), this is in many real-world cases sufficient space to encode state information you have to pass along. While feature-wise not equally powerful (server side state implementation ensure that state modifications will be shared with all API calls chronologically happen after the modification, while URI based state will only be available to resources that are linked directly or indirectly from the response that caused the modification), is is still sufficient in many cases. If that's what you need, you and your users get it for free. Even more important: you can introduce and change the availability and scope of state whenever needed. Often you only realize in an advanced version of your (stateless) API implementation, that information that was available in one step of your use case flow could provide improved results if also available in one of the subsequent steps. In a typical URI-template based API contract you would have no chance to pass this information along with existing clients, simply because it would require a change in existing client implementations to pass along that additional information. With hypertext you can do this kind of things, without breaking existing client implementations.

###Extensible UX flows
With **meshcaline**'s flavor of hypertext you can enable one additional, very powerful feature: extensible UX! Having resource types as a core concept of a **meshcaline** design, an application could build extensible UX components, which can automatically (without any code change) offer new features to the their users as the feature-set of your API evolves: Assume our simple music service has four resource types: individual songs and artist (`#song`, `#artist`) and lists of those (`#songs`, `#artists`). Now suppose we define for each of the four resource types an attribute "related" and define it to hold a collection of hypertext controls, each containing a "title" element. E.g. in an early version of your API you offer a link to other popular songs of the same artist:

	{
	     "title" : "London Calling",
	     "interpreter" : {
	          "name" : "The Clash",
	          "href" : "https:/....",
	          "type" : "#artist", ...
	     },
	     "related": [
	          {
	               "title": "More songs of 'The Clash'",
	               "href":"https:/...",
	               "type":"#songs"
	          }
	     ]
	}

If  application developers render in their UI some buttons displaying the title attribute and on selection of that button display a UI appropriate for resource-type, their applications will automatically offer additional features to their users when subsequent releases of your API introduce additional features:

	{
	     "title" : "London Calling",
	     "interpreter" : {
	          "name" : "The Clash",
	          "href" : "https:/....",
	          "type" : "#artist", ...
	     },
	     "related": [
	          {
	               "title": "More songs of 'The Clash'",
	               "href":"https:/...",
	               "type":"#songs"
	          },
	          {
	               "title": "Other British post-punk bands",
	               "href":"https:/...",
	               "type":"#artists"
	          },
	          {
	               "title": "Similar songs'",
	               "href":"https:/...",
	               "type":"#songs"
	          }
	     ]
	}

While in URI template based API your users would read you release notes to learn about the new features, check in documentation how to call those three different features and then implement some code that constructs the request, in a **meshcaline** API the amount of client code is primarily influenced by the number of resource types your API supports, not by the amount of API features. So as long as you don't introduce new result types, a forward compatible app implementation would have the new features automatically.

As you might have seen the `related` collection contains hypertext controls for multiple resource types (both `songs` and `#artists`). We could even introduce new resource types over time (e.g. `#gigs`). All your user's applications have to be prepared for is to ignore any link with an unknown resource type (in the same way as they are used to ignore data attributes they don't know) and for the remaining ones let the user select one of the `title` elements and then pick an appropriate UI component for rendering the returned resource type. Such simple "forward compatibility processing rules" allow you to expand the features delivered by you API without being forced to introduce a new API version.

In summary: A hypertext based API allows the API developers to improve and extend the implementation of their offering without breaking existing clients and allows API users to automatically gain from those improvements without the need to modify / reship their apps.

##Developer Experience

Beyond the technical advantages of hypermedia, one of the most compelling reasons for a hypertext based public API offering is not technical at all: By using hypertext as interaction paradigm you can help your users to more easily understand your API, simply by using it within a browser. Even when you do nothing special but just serve your standard JSON representation, your users could just use one of the browser plugins that provide a nice JSON rendering and then see everything they can do with your API by playing around with your API in a browser and follow the links to other steps within the use-case flows.

With some extra efforts you can even achieve significantly more: You could use standard HTML as representation format or, by supporting HTTP content negotiation, allow your users the select amongst various representation formats the one best suited for their needs. Then your API would automatically return the HTML representation when an API is entered in a browsers address field.

Instead of / in addition to the JSON representation

	{
	     "title" : "London Calling",
	     "interpreter" : {
	          "name" : "The Clash",
	          "href" : "https:/....",
	          "type" : "#artist", ...
	     },
	     "related": [
	          {
	               "title": "More songs of 'The Clash'",
	               "href":"https:/...",
	               "type":"#songs"
	          }
	     ]
	}

you provide a HTML representation of your resource types:

	<!DOCTYPE html>
	<html>
	<body>
	<dl>
		<dt>title</dt><dd>London Calling</dd>
		<dt>interpreter</dt><dd>
			<a href="https:/....">
				<dl>
					<dt>name</dt><dd>The Clash</dd>
					<dt>type</dt><dd>#artist</dd>
				</dl>
			</a>
		</dd>
		<dt>related</dt><dd>
			<ul>
				<li>
					<a href="https:/....">
						<dl>
							<dt>title</dt><dd>More songs of 'The Clash'</dd>
							<dt>type</dt><dd>#songs</dd>
						</dl>
					</a>
				</li>
			</ul>
		</dd>
	</dl>
	</body>
	</html>

While most developers would most likely still prefer to use a regular JSON/XML within their applications, supporting HTML representation in addition allows developers to immediately see all API features within a regular web browser. The can even directly follow all links with GET requests without additional work for you.

By adding some JavaScript and CSS, you can then decorate your API with a fully fledged playground and turn it into a marketing tool / learning tool for your API:

* decorate resource types values in `type` and `accept` attributes with links to the corresponding API documentation sections
* use `accept` values to select widgets that allow to enter values required for following the hyperlink and construct the HTTP request
* use `auth` values to manage authorization needs and when required to display login widgets
* render a full dump of the request (e.g. by means of a curl statement) along to response representation 
* provide a "report a  problem" link along with every response. If your developers have a problem with / identify a bug in your API, they would simply fire the corresponding API call in a browser, then click on the report button, and the corresponding issue-reporting widget could then post your user's report along with the full HTTP request and the returned response into your issue tracking system
* provide UI widgets for rendering your resource types, that illustrate how the information returned could/should get displayed

Contrary to most other playground solutions, this approach guarantees that the playground is always up-to-date with your API, as it is not a separate component independently developed from your API, but instead the API IS the playground. And still: the actual playground implementation can be nicely decoupled from your actual API code, maybe with some minor exceptions, like embedding JavaScript and CSS links into your HTML representation or some hacks required to overcome limitations of browsers (e.g. tunnel PUT/PATCH requests through POST, as browsers only support GET and POST in forms)

Most modern web service frameworks provide support for multiple serialization formats and HTTP content negotiation, so building the initial support for multiple content types is doable with little extra efforts. Of course the actual implementation of the playground features like UI widgets in JavaScript and CSS can become quite some work -- depending on your own expectations and goals. 

##What does hypertext bring to your business?

All the advantages listed above are important for you business, as they reduce the cost of change, help you promoting/explaining your offering to potential users and whatever helps your users is also good for your API business.

An additional, and maybe the most important aspect of hypertext based API on the long run is the possibility of a detailed analysis of your API usage. If your API is representing your business offering, you need to understand not only which resources of your API are popular, but also how your user's applications combine the various resources in their application flow.

Take the efforts successful websites spend to analyze the behavior of their users and how they navigate through their website. I believe nobody disagrees that detailed understanding of user behavior and optimization around that is what differentiates a good website from a successful one. Now transfer this analogy from the website to your API: A classical, URI template based API is equivalent to a website where the only analytic tool you have available are page impression counts, as a HTTP based API (contrary to websites, where the browser implicitly sets the referrer header) lacks mechanisms that allow you to analyze the use case flow your users have taken. With a hypertext based API design instead where the URLs that are linking between the individual steps of a use case flow are nothing but meaningless opaque strings for the API user, you can easily encode additional information into the URI that allows you to track the detailed flow of your users (or better: the end-users of their applications).

If your API provides more that just a HTTP based interface to well-defined functionality, if you build a solution where you have somewhere in your implementation to make the choice which of the potential responses might be the best / most appropriate for a given use case flow, or how to best rank the list of next possible use case flow alternatives, then detailed tracking of your API usage is mission critical for the incremental improvement of a good API towards a successful one. 

The collected usage data can become the strategic business asset securing your business success once other players come and try to copy your offering. You better don't rely on the hope that you're too clever for others to copy your innovation. If Gartner is right and [information will become the oil of the 21st century](http://www.gartner.com/newsroom/id/1453519), then you better start collecting data early, so that you can convert it into information required to stay successful in your domain. And if your API offering is in the heart of your business, then you better ensure that the tracking happens implicitly (through your links) and you don't rely on your API users to send back tracking information to you.
