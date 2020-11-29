# Basic meshcaline

*meshcaline*{: .m} propagates a hypertext based API design paradigm for HTTP(S) based web service API. It is intended to suit service products that want to expose their offering primarily through an API ("The API is the product"). As such they need to expose a simple to understand and easy to use API (otherwise it is unlikely that the product will become successful) that allows for ongoing enhancements.

The basic idea of the *meshcaline*{: .m} API design is quite simple:  
A *meshcaline*{: .m} API represents the business processes supported by the service offering. Most non-trivial business processes consist of multiple steps. At each step either the developer or the end user of the developer's app decides, based on information obtained in the previous step and additional information contributed by the app/user, where to go / what they need next. 
Entering such a business process as well as each subsequent step of the business process corresponds to an API request, with responses containing necessary information and hyperlinks to the next possible steps. 

In short: 
<div markdown="1" class="alert alert-primary lead" role="alert">
The *meshcaline*{: .m} API design adopts the interaction principles of web sites to web service API.
</div>
While web sites are designed for human beings, capable grasping a lot out of unstructured context information which then allows them to pick and choose from the available hypertext options, APIs must be designed for algorithms and the developers that have to implement those algorithms. And while it is quite easy to for human being to identify that a link they followed isn't appropriate for what they were looking for, developers (and their algorithms) definitively don't want to make API calls and analyze the response, just to find out that this is not what they need. Therefore one of the most important parts of the *meshcaline*{: .m} API design is the exposure of meta information along with the hyperlinks that supports developers in the decision making which of the alternative subsequent requests they want to make or offer to their end users for selection. The combination of hyperlinks and their corresponding meta information is called hypertext controls.

*meshcaline*{: .m} implements the HATEOAS constraint, but provides quite different means to interact with the API than Roy Fielding's REST rules do enforce. By following the *meshcaline*{: .m} API design paradigms, you might still realize that you end up with a product that is more RESTful than most of the other so-called REST API you find out there on the web. Your API should be on level-3 according to the [Richardson Maturity Model](http://martinfowler.com/articles/richardsonMaturityModel.html). 


##Content types and resource types

*meshcaline*{: .m} APIs can use any regular content type that allows embedding hypertext controls, including popular ones like json, xml, html or their binary counterparts. A *meshcaline*{: .m} API may support various content types simultaneously and allow clients to use HTTP content negotiation mechanisms to indicate the one they prefer. 

Contrary to plain REST *meshcaline*{: .m} introduces explicitly a concept for resource types used in hypertext controls embedded in API responses. Every hypertext control identifies the type of resource the URI is pointing to. Resource types use identifiers specified in the API documentation. 

It is important to emphasize that resource types are independent from content types. A resource type specifies the content elements required to describe the concept a resource represents. Those elements may then be serialized in various formats, which are specified by the content type. The documentation of an API usually specifies the set of supported content types for a given resource type. 

Resource types are also used to specify, where necessary, the type of resource a client has to send to the API with a request message. Equally to the resource types for API responses, *meshcaline*{: .m} doesn't enforce any specific content type used to serialize a resource type. Only for resource types describing request parameter for GET methods a standard binding to URI query string parameters has to be applied. 

*meshcaline*{: .m}'s resource types are primarily used to describe the hypertext resources within a given API. For links to other web resource types (e.g. jpeg-files, web-sites), esp. those outside the territory of the API, it is usually simpler and equally appropriate to provide the resource's URI as value of a specific content attribute. Nevertheless it is perfectly valid to use hypertext controls for those links too and if an API implementation decides to do this, it is only required to specify resource types of those links too. It is recommended to use a regular media-range value as specified for the HTTP accept header as resource type identifiers for those links.

Three resource types are predefined:

* `#none`: The resource doesn't have any representation
* `#any`: The resource may return anything; 
* `#implied`: The resource type is equal to the type specification of the hypertext control's context (the element that contains the control).

The type `#implied` helps avoiding inflationary definitions of resource types. While an API documentation may only describe the mayor response types, such a type introduces an anonymous resource type that is defined as equally to the specification of the sub-element within the resource type they are embedded with. 

For many structured content types used in API, there also exists some schema language that allows specifying the syntax and semantics of a given entity of that content type. If a *meshcaline*{: .m} API provides such schema for their resource types within the content types it supports, then the resource type identifier should allow resolving the corresponding schema element. 

In most examples throughout this site JSON will be used as content type, simply because its the least noisy and easiest to read for human beings amongst the popular representations. This should not imply any restriction on the applicability of *meshcaline*{: .m} to other content types.


##Hypertext controls

The hypertext controls in *meshcaline*{: .m} are used to link between individual API resources representing the available steps in a given business process. They provide client applications three things:

* Describe the relationship between a given API resource and the API resource the control links to
* Ensure that the resource type returned from that linked resource is a type the application is able / willing to process
* Verify that the application knows everything needed to construct the request to the linked resource properly

The hypertext controls and the API's resource type specification gives client application developers everything necessary to know why, if and how to successfully follow a specific link. 

Contrary to the hypertext controls in HTML (e.g. forms) the *meshcaline*{: .m} hypertext controls in general don't aim to support generic UI components, although a specific API might use them for that purpose too.

The hypertext controls provide various information elements to the application
 
* A URI linking to the other API resource (`href`)
* The resource type of the target response (`type`)
* The HTTP method to be used for that hyperlink (`method`)
* The resource type(s) specifying the data expected from the resource (`accept`). 
* The authentication schema(s) supported for the target resource (`auth`)
* A link-relation type, describing the type of relationship between the resource that contains the control and the resource it links to.

The link-relation type is not a specific attribute of a hypertext control. Instead the name of the data element that represents the hypertext control shall be the type of relationship. Is is recommended to use nouns for hypermedia controls of related resources (e.g. _author_) and verbs for those that represent client/user activities (e.g. _delete_). _*meshcaline*{: .m}_ encourages the usage of [standard link-relation types](http://www.iana.org/assignments/link-relations/link-relations.xhtml) where appropriate (`next`, `previous`, `self`...). 

In addition to the above elements, a hypertext control may contain arbitrary attributes which the API designer deems to be relevant for the application developer in the context of the link.

*meshcaline*{: .m}

<div markdown="1" class="alert alert-info" role="alert">
#### Example 1:

Let's start with a simple JSON representation of a resource type `#song` that also contains a link to a resource representing the interpreter of that song

	{
	     "title": "London Calling",
	     "interpreter": {
	          "name": "The Clash",
	          "href": "https:/....",
	          "type": "#artist",
	          "accept": "#none",
	          "method": "GET"
	          "auth": "BASIC", 
	           ...
	     },  ...
	}

In this example the JSON attribute "interpreter" is a hypertext control:

* `interpreter` specifies the link-relation-type between the song and the linked resource
* `href` contains the URI linking to the target resource
* `type` specifies that the target resource is of type `#artist`
* `method` specifies that the application should use HTTP GET to access the resource
* `auth` indicates that the resource will require BASIC authentication

The attribute `name` is a regular data attribute within the hypertext control.

</div>


Allowing arbitrary additional attributes in a hypertext control is an important part of *meshcaline*{: .m}. Claiming to model the individual decision points in a business process, a control has to provide all information that allows to make that decision. In some cases the existence of the link and its relation type might be sufficient. But usually, and especially when the end user is supposed to make the decision, the application must present some UI element to the end-user that indicates based on the additional attributes in the hypertext control, what this decision is about. Those additional attributes may describe either the resource the control is linking to or give additional information related to the relationship between the two linked resources or both.

<div markdown="1" class="alert alert-info" role="alert">
####Example 2:

Let the representation of the `#artist` resource type, when representing a band, also list the individual members of the band

	{
	     "name" : "The Clash",
	     "members" : [ {
	          "name": "Joe Strummer",
	          "instruments": ["lead vocals", "guitar", "harmonica", "keyboards"]
	          "href": "https:/....",
	          "type": "#artist",
	          "accept": "#none",
	          "method": "GET"
	          "auth": "BASIC",  ...
	     },...
	     ],  ...
	}

In that context, the hypertext control to the resource type `#artist` not only contains the name of the artist, but also the instruments the artist played in that band. This information is not an attribute of the linked artist, but specific to the relationship between this band and the artist, as the artist might have played other instruments in other bands.
</div>

*meshcaline*{: .m}'s hypertext controls don't come with any special meta objects. They are nothing but standardized names for attributes of regular content elements. While other hypertext schema introduce special meta-objects that allow client applications to "discover" hyperlinks in a representation, *meshcaline*{: .m} doesn't. Initially we thought this might be a nice concept, but during user testing we had to learn that our users anyway don't want to search for hyperlinks. They want to know at coding time, which content element represents a link they might have to follow. As a consequence *meshcaline*{: .m} forces the documentation of a resource type to indicate which elements are hypertext controls.

Nevertheless *meshcaline*{: .m} allows extending an API over time as the capabilities of the API evolve in a backwards compatible way.

<div markdown="1" class="alert alert-info" role="alert">
####Example 3:

Assume in your initial API version, the "#song" resource did not yet have a dedicated resource for the interpreter, but only some attributes.

	{
	     "title": "London Calling",
	     "interpreter" : {
	          "name": "The Clash", ...
	     },  ...
	}

In a subsequent versions you could still extend "interpreter" to become a hypertext control as shown in Example 1 in a backwards compatible way. 
</div>

You might have already realized that backwards compatibility in this example is only given under the assumption that the client application does ignore unknown elements in a response. A dedicated article on compatibility will give a more thorough introduction on rules around extensibility and backward compatibility with *meshcaline*{: .m}

## Processing rules

In addition to the fundamental hypertext control elements, a set of processing rules ensures that not every hypertext control must contain all attributes, and by that reduce the noise and consumed bandwidth introduced by the hypertext controls:

 * `method`: HTTP method defaults to `GET`

 * `auth`: If no authentication schema is given, the same authentication schema as provided for the origin resource is appropriate  
In many cases API need just a single authentication mechanism (if at all), and then the `auth` parameter is never needed. But some API may require different types of authentication for different API resources (some *public* resources, some *protected* resources which require the client to authorize, some *personalized* resources which require authentication of the end-user of the client application). To achieve the goal that the hypermedia control provides clients means to successfully construct the next request, the hypertext control indicates any required switch of authorization.

 * `accept`: For HTTP methods without request body (most important `GET`) `accept` defaults to `#none`. For HTTP methods that contain a request body, it defaults to `#implied`.

<div markdown="1" class="alert alert-info" role="alert">
####Example 4:

Assuming the origin resource was already fetched with BASIC authentication rules 1 to 3 would allow a much more readable representation for the hypertext control of example 1 

	{
	     "title" : "London Calling",
	     "interpreter" : {
	          "name" : "The Clash",
	          "href" : "https:/....",
	          "type" : "#artist", ...
	     }, ...
	}

</div>

 * `type`: If no resource type is specified `#implied` is implied.

<div markdown="1" class="alert alert-info" role="alert">
####Example 5:

Assume your API allows you to post some star rating for each song

	{
	     "title": "London Calling",
	     "interpreter": {
	           ...
	     },  
	     "rating": {
	          "average": "3.8",
	          "count": "17",
	          "create": {
	               "href": "https://...",
	               "method": "POST",
	               "accept": "#ratingValue",
	               ...
	          }
	     }...
	}

While the request body is specified with resource type `#ratingValue`, the control doesn't explicitly specify a target response type. The response will therefore have the identical structure as the context the `create` link is embedded in (the `rating` element):
	
	{
	          "average" : "3.9",
	          "count" : "18",
	          "create" : {
	               "href":"https://",
	               "method":"POST",
	               "accept":"#ratingValue",
	               ...
	          }
	}
</div>

 * For specific link-relation types a *meshcaline*{: .m} API may represent a whole hypertext control with just a URI string and define specific values for the hypertext control attributes of that type (except for the `href` attribute) that are valid throughout the whole API.   
This rule should only get applied to [standard link relation types](http://www.iana.org/assignments/link-relations/link-relations.xhtml), like `self`, `next`, etc and will in most practical cases be limited to `GET` methods.  
In those cases the simplified hypertext control representation is more readable and fully sufficient.

<div markdown="1" class="alert alert-info" role="alert">
####Example 6:

A resource type `#album` shall contain a linked list of cover-art jpegs

	{
	     "title" : "London Calling",
	     "coverart" : {
	          "src" : "http://",
	          "page" : "front",
	          "next": "https://,...
	     },  ...
	}

This example `next` is a link relationship type that uses the simplified representation. The `coverart` element instead represents the URI to the actual cover binary as data-element `src`, not as hypertext control. 
</div>

Those rules do not necessarily simplify the processing of the hypermedia controls -- they even add some complexity on the client's processing logic. Also the reduced bandwidth will only be marginal  -- as soon as you in start working with hypermedia you should anyway use gzip compression as transfer encoding and this would eat up most of the redundancy too. 

The primary reason for those rules is noise reduction -- not for technical reasons but for clarity: While API output is primarily going to be consumed by algorithms, the prerequisite for any output to get consumed by an algorithm is that a developer who writes those algorithms understands the API. The less avoidable information you have in your representation, the easier it becomes for a developer to understand the essence of you API. 