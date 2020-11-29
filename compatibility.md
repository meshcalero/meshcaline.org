#Compatibility, Extensibility and Versioning

API users expect that, once they have chosen and integrated an API in their solution, the API's contract and behavior doesn't change anymore. Any change holds the risk that their application is negatively impacted. Even behavioral changes that everyone might see as improvement could cause an existing application to fail, if it relies on the old behavior.

API developers instead are primarily interested in continuous improvement of their offering, in order to stay ahead of the competition and to win new customers. While maintaining existing users is equally important, they know that an API product that doesn't evolve will sooner or later be dead.

It is obvious that this conflict can't get resolved by other means than a compromise. But be prepared, that you as API provider must take the bigger burden, if you want your API to become successful.

#Compatibility for service offerings

In the traditional software library business you release a software artifact to your users and once shipped you can more or less forget about it. Your users pick a specific version of your library and embed it into their application. You maintain distinct branches of your source code in order to do bug-fixes of a specific version when needed, but even if you have to do this, you still do this more or less in isolation to all other software releases you ever had. A given piece of code that is in use at runtime in one of your customer's applications only serves a specific version of your library product. The [major/minor/revision-versioning scheme](http://en.wikipedia.org/wiki/Software_versioning) is a well-established schema that allows your customers to estimate the effort/risk for replacing a previous version of your library with a newer one. But as those updates usually never happen without rebuilding (and by that re-testing) your customer's applications, violating the compatibility promises implied in your version number, although they definitively doesn't not make your users happy, is still no disaster that can put your customer's business immediately at risk. 

When running a service API, you're facing nearly the opposite: The ultimate goal of running a service is continuous delivery. You no longer actively care about individual releases of your service. Instead, as soon as an improvement is ready (regardless how big or small it may be), you want to make it available to your customers. For both economical and technical reasons, you want to serve many (if not all) customers of your service with the same runtime components, regardless which version of API was current when they shipped their application. This means that you must be able to run multiple versions of your service in parallel (your customers would even prefer *all* version ever released *forever*).  The version of the API product your customers are using and the software artifact that provides this service are still related, but other than in library business no longer have a 1:1 relation. Instead the software artifact providing the service has a 1:n relationship to all the API products it supports. Which leads immediately to the next observation: if a software artifact represents many API products, you can't use one version schema for both. 

In a service business your users anyways don't care about your software version. The only thing that counts for them is the version of the interface contract of your API product (both functional as well as non-functional) they shipped their application with. Which service software version implements that contract is irrelevant to your customers, as the good-old-library-times, where they could choose one of those versions themselves are gone too. 

The last point shows another radical difference to the library business: When your customers no longer can choose which software version of your service is in use in their applications, a release that is not backward compatible will now result in a disaster for your customer's applications. To make it worse: If you fail on your compatibility promises, there is nothing your customers could have done to prevent their application from failing. But the worst of all: Even if you have clearly specified your compatibility commitments for future releases and tested your release against those commitments, you should not expect that your API users have read and fully understood your specification. You might be able state a [RTFM](http://en.wikipedia.org/wiki/RTFM) when their applications then fall apart, but if your goal is to prevent your customers from failing, you need to help your users to prevent this from happening.

This all leads to three consequences for a successful service API offering that keeps its users satisfied:

* You must be very precise with the compatibility commitment of your service. This must also clearly state rules for enhancements and extensibility.
* You must have tests for all your compatibility commitments your API has ever given and run **all** of them against your current release of your service implementation 
* You must provide your users means to test at least the [forward compatibility](http://en.wikipedia.org/wiki/Forward_compatibility) of their application and where possible also their [extensibility](http://en.wikipedia.org/wiki/Extensibility)

##Compatibility of meshcaline resource types

The primary subjects for the compatibility specification of a meshcaline API are the individual resource types. A release of a service implementation is backward compatible, when it serves all its resource types in a backward compatible manner.

### Semantic changes of resource types
Any semantic change of either a resource type as a whole or individual elements of a resource type is a backward **incompatible** change.
In meshcaline even the allowed scope of backward incompatible semantic changes for whole resource types is restricted: The only allowed semantic change of existing resource types is more specificity, but not generalization:


The most important aspects of compatibility is the semantic of a resource type as a whole as well as the semantic of the individual elements of a resource type. A service implementation can only be compatible when it serves the same semantics. 
Examples for semantic changes include:
* An API for holiday homes has a `price` element that contains the price of a holiday apartment per week. A future release that returns the price per day is not backward compatible. 
* Generalization of resource types:If a resource type `#song`was explicitly specified to represent the concept of a music stream (not a generic audio stream) then a future version can't represent an audio book stream in that resource type, even if all individual elements of the resource type would be identical.

You can only mark optional elements as deprecated to indicate that some future (then incompatible release) may discontinue them

The remaining compatibility commitments for individual elements of a resource type differ between those used for responses and those used in requests (as available in `accept` property of a hypertext control):

For future releases of response resource type implementations the backward compatibility is given when for each element the following rules are honored:

* Preexisting elements can only become more restrictive:

	* Optional fields may become mandatory
	* The value range may become more narrow
	* Generic hypertext controls (those where the documentation of the link relation type lists multiple allowed resource types as destination) may link to a smaller amount of resource types

* New elements can get added at will, unless they massively impact non-functional behavior of a previous release. E.g. while you can add to our `#song` resource type example a URI pointing to a mp3 file of the song, you can't embed the mp3 binary as [Data URI](http://en.wikipedia.org/wiki/Data_URI_scheme) directly in the response, as the massive increase of the response size will negatively impact the runtime characteristics of the calling applications. 

For request resource types the backward compatibility rules are the the other way around:

* Preexisting elements can only become less restrictive:

	* Mandatory fields may become optional

	* The value range my become wider

* New elements can only get added as optional elements

For resource types that are in use for both requests and responses, the only backward compatible change is addition new optional elements.

## Resource type versioning

A meshcaline service implementation may have to support multiple versions of its resource types in parallel. Meshcaline API apply a consistent versioning number schema for their resource type versions: 

* An increase in a major version number indicates a backward incompatible, **semantic change**
* An increase in a minor version number indicates a backward incompatible, **syntax change**
* Optionally, a meshcaline API may provide revision numbers for compatible changes, but this is not mandatory

## Service API versioning

A meshcaline service API is defined through a set of resource types and a (set of) entry point URI.

A meshcaline service API version is therefore indirectly defined through the set of resource type versions it serves.

