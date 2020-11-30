# API Evolution & Compatibility

Iterative methodologies are best practice for product development. Even among those who don't practice Agile methodologies, nobody would try to develop a non-trivial software product with a waterfall process.  It is just a matter of fact that it is impossible to collect all requirements of a to be built product upfront and then correctly specify the solution for all those requirements that once implemented will meet the expectations of its users. This doesn't mean that you should not try to, but at the same time you have to remind yourself that this is just a plan and as an old military wisdom says:

> No plan of operations extends with any certainty beyond the first contact with the main hostile force
> -- Helmuth von Moltke, [On Strategy (1890)](https://books.google.de/books?id=WHgvafaY1UIC&pg=PA291#v=onepage&q&f=false)

better known in its shorter version:

> No plan survives first contact with the enemy

The simple truth is that you can't be sure how good your product actually is until customers (try to) use it. Then you learn from their feedback and iterate.

This is true for any kind of product and no different for an API product. When you assume your first version of your API will be perfect, be assured that you will be proven wrong. Even if you believe you know your (future) customer's needs inside out, they will soon hurt you by pointing to various aspects where your design and functionality doesn't fully meet their expectations. And once you have satisfied your first customers and they have integrated your API into their solution, the next wave of potential customers will surprise you with just another set of issues and gaps.

As a consequence you must be prepared for iterative API evolution. By that you will incrementally add missing functionality and remove aspects that are of no use for your customers. Unfortunately that's where a fundamental difference in API based products makes your life very difficult. While (software) products that get _shipped_ can still be used by their users after that product version has been replaced with a new one, when you _serve_ your product via an API, then you have to make sure that you don't break existing clients when you release a better but different version of your API.

In theory there is a simple mechanism to manage that problem: Treat you API like a shipped product: once shipped it doesn't get changed anymore. Unfortunately for most real-live API products this is impractical,  esp. when you apply Agile methodologies and try to release early and frequently: Even when you don't plans to touch your previous versions any more, you will have to. Any kind of software service needs maintenance for at least two reasons:
* You will have bugs in your software that need to get fixed. 
* Nobody builds software that is not based on other software components. And each of those components might release new versions and you will have important reasons to upgrade (e.g. performance improvements, security fixes, ..). 

Even when you're not applying continuous-delivery as part of your software development process and only release your updates at the end of your sprints, you'll easily end up with 10-20 additional versions of your API per year that you have to maintain. Very soon your development team will do nothing else but maintenance.  

## Backward compatibility

As a consequence of the above you have to accept the much more cumbersome approach: [Backward compatibility](https://en.wikipedia.org/wiki/Backward_compatibility). Unfortunately this approach restricts you in the amount of change you are allowed to apply to your API. The *meshcaline*{: .m} design helps you minimizing the limitations for backwards compatibility.

### Hyperlinks rather than URI templates

As already described in ["Why Hypertext API?"](hypertext/#changeable-resource-addressing-schema) *meshcaline's*{: .m} hypertext based design allows you to change the addressing scheme of the individual resources of your API, including splitting up the API implementation into multiple service. While there are various  mechanisms how you could split up the implementation of a growing API, they all come at least at the expense of the introduction of an additional component that does the traffic routing, which introduces an avoidable single-point-of-failure. With a *meshcaline*{: .m} API you get a scalable solution for free as your hypertext controls would always point to the correct service.

### Extensible API contract

By default *meshcaline*{: .m} API are open for extension. Their API documentation (and when available their schemas) of their resource types allow you to add additional resources, add new resource types and modify existing resource types as long as those changes are still backwards compatible to previous versions of your API.

Unfortunately backward compatibility is a tricky beast and you can break compatibility by different means:

#### Syntactic changes

With syntactic changes we refer to any type of change of the specification of the resource types of an API. Ideally the specification exists in form of some formal schema and you can easily verify that your current API's resource types also match the specification of your previous versions. 

Since resource types can be used for both retrieving data from your API (response types) and sending data to your API (request types), both, your service and your clients, will have to ignore any unexpected data element. You have to be careful what kind of syntax change you introduce, as depending on what the changed resource type is used for they may or may not break compatibility:

| Type of Change | Request Type | Response Type |
| --- | :---: | :---: |
| Adding an optional element | ok | ok |
| Adding a conditional element | error | ok |
| Adding a mandatory element | error | ok |
| Removing an optional element | ok | warning |
| Removing a conditional element | ok | error |
| Removing a mandatory element | ok | error |
| Extending the value range of an existing data element | ok | error |
| Restricting the value range of an exiting data element | error | ok |
| Widening the size constrains of collection elements | ok | error |
| Narrowing the size constrains of collection elements | error | ok |

As you can see the only type of change you can safely apply to the specification of your resource types is adding optional elements. When a resource type is only used for responses you have a bit more freedom to change. Sometimes you can also take advantage of the allowed changes for request types, but in practice most API also respond in some form the data elements of request type and as a result write-only data types are quite rate. 

For resource types that get used for both request and responses, you might have to introduce specific variants. This is often anyway recommended because request types usually do not contain the hypermedia controls embedded in response types. Defining distinct request/response types would also allow you to use different constraints for the same elements for added data elements: While the response type guarantees that a new data element is always available (mandatory) in the request type it is only optional, so that old clients still work.

It happens quite often that you realize that you have to replace a simple data element with a more complex data type. In that case it is best practice to keep the old definition as deprecated data element in your resource types and put the more granular new data type in an additional data element. Also here you might be required to introduce separate type definitions for request and response types.

#### Semantic changes

Semantic changes could get applied to a resource type as a whole as well as to the semantic of the individual elements of a resource type. Examples for classes of semantic changes include:

* _Modification_: An API for holiday homes has a `price` element that contains the price of a holiday apartment _per week_. If a future version would decide to represent the price _per day_ this modifies the semantic although the syntax of the resource type has not changed. 
* _Generalization_: If a resource type `#song` was explicitly specified to represent the concept of a music stream then a future version could generalize it to audio stream and contain e.g. an audio book stream if all individual elements of the resource type would be identical.
* _Specialization_: Similar to generalization, specialization changes the semantic: When we realize that all clients of our service are only interested in Punk music, then we might kick out all other types of music from our system.

| Type of Change | Request Type | Response Type |
| --- | :---: | :---: |
| Modification | error | error |
| Generalization | ok | warning |
| Specialization | error | warning |

Semantic changes are more subtle than syntactic changes. Usually no client application will break directly, but the quality of your client's application will be impacted, as they know longer deliver the expected semantic. As a result the chance will most likely cause harm to your business processes. Due to that is recommended to avoid semantic changes unless you fully understand their implications and can deal with them. 

#### Non-functional changes

The aspect of non-functional API changes is ofter forgotten, but they can also create backward incompatibilities for your clients. Let's look into some obviously harmful non-functional changes:

* _Authentication_: Assume you introduce some authentication mechanisms to your API that previously was accessible without. This obviously kills your existing clients
* _Rate limits_: Rate-limits or quotas are another common pitfall: While your are initially happy about everyone who is using our API, after it got popular you have to manage the load. As a result you might start limiting the use of your service by introducing hard rate limits or degraded service quality.

While both cases seem obvious, I've seen plenty of examples where they were forgotten in the initial design of an API and then later could not get introduced without breaking existing clients. Given that you build your API product to become successful and then those aspects will become relevant, you should never postpone those design aspects to future versions.

But there are also more subtle non-functional changes that you have to take care of:

* _Request latency_: Good customer experience has to be responsive. If your API is supposed to enable the implementation of a business process, then the latency of your service has a direct impact on the customer experience and with that quality of your clients implementation
* _Response size_: *meshcaline*{: .m} API are expected to be called directly from end-user devices. Those might not have connectivity with high bandwidth and as a result also the response size will directly impact the responsiveness of your client's application. Usually adding some data elements should not cause harm, but other types of extensions will. Examples for this include collection resources (without pagination) where the number of entries growth over time or introducing large data elements (e.g. embedding the mp3 as [Data URI](http://en.wikipedia.org/wiki/Data_URI_scheme) directly in your response)

### Processing rules

The documentation of a *meshcaline*{: .m} API usually doesn't only contain the specification of syntax and semantic of your resource types. In addition to that it may also contain processing rules. We have already seen some of them in the description of *meshcaline's*{: .m} [data model for hypertext controls](../basics#processing-rules). Those processing rules got introduced to maximize conciseness and readability of the API responses.

Another important purpose of processing rules is forward compatibility: They instruct your clients how to deal with expected future extensions of your API. 

We have seen the most important processing rule of *meshcaline*{: .m} API already in the section above: 
_"Ignore any data element that you don't expect!"_

Another example for a processing rule was used in the [Extensible business process flows](/hypertext/#extensible-business-process-flows) example, where we asked clients to _"Show the title of all related resources when you know its resource type and guide your users to them by using the provided link!"_

The latter rule is basically an extension of the _"Ignore any data element ..."_ rule, but instead of individual data elements it applies to individual items of an untyped collection. 

Other variants of processing rules might be based on some polymorphic relationship between all possible items in some collection, that would allow clients to even provide some generic functionality for resource types that were unknown at the time where the client application has been built.
 
When designed carefully such rules not only allow you extend the feature set of your API incrementally. They may even allow you to automatically extend your client's applications too.

Unfortunately processing rules have two major downsides:

- You ask your clients to invest in business logic that may or may not come and where they are not even sure they want/need it
- Neither you nor your clients have means to test if the processing rules have been implemented correctly.

The first downside is a product question that you have to address first. If your promise of future extension is not strong enough for your clients to do the extra investment (and your processing rules are hopefully easy to implement), then you better don't introduce them.

For the second downside *meshcaline*{: .m} suggests a technical solution: Extend your API implementation with a _fast-forward_ mode. When active you introduce random API extensions in your response types that match your processing rules. You might activate the fast-forward mode by default in your sandbox environment where your clients test their applications as part of their software development processes, and you might also allow your clients to activate the fast-forward model on your production service (e.g. by introducing a special header to each API request), to enable manual tests on production too.

## Think ahead!

Backward compatibility is the mechanism that allows you to iteratively develop your product without maintaining additional versions with every release. As we have seen it unfortunately limits your freedom to change substantially. As a consequence you have to be prepared that there will be a time where you have to introduce a new, backward incompatible (_major_) version. This is not doomsday, it is a regular part of the lifecycle of your API product that you have to manage. 

Instead of fighting windmills by trying to avoid new major versions, the goal of your design efforts should be to introduce them so infrequently that it is economically feasible to maintain multiple major versions at the same time. How to exactly calculate this heavily depends on your business. As a rule of thumb I would recommend to start with a best case assumption that you have to maintain a major release for at least 2 additional years after the next major release has been launched: Not only will your clients have to find budget and time to invest into the migration. Given that *meshcaline*{: .m} API are intended to be used directly from client applications, including binary applications installed on the end-users device, your clients will often not be in full control to upgrade all older versions of their applications either. In change-averse industries substantially longer maintenance periods are quite common, but in those cases your sales team hopefully negotiated a commercial agreement that compensates you for the additional maintenance costs.

Better don't assume that the maintenance period you have written in your API contract will give you a guarantee that you can decommission old major versions at their end. As long as you still have substantial traffic on those versions, decommissioning will make your customers quite unhappy and with that cause harm to your business. In practice you will have to maintain older versions substantially longer (Microsoft maintained Windows XP >7 years longer than originally announced). The contract only guarantees you that nobody can sue you, in case have more important business reasons to discontinue the support.

When you have to prepare yourself for maintaining old major versions for a long time, then one of the goals of your API design activities should be to maximize the duration between two major versions. The only way to do this is to think ahead: Make assumptions about possible extensions / changes that you see upcoming in the next 1-2 years and design how you could introduce those features without breaking compatibility for the version you are currently working on. 

<div markdown="1" class="alert alert-primary" role="alert">
Whenever I mention something like that, there is someone in the room who says: "Isn't this violating the Agile principle of _Do the simplest thing that might possibly work_?". I believe this is one of the biggest misconceptions related to Agile practices. Agile is not about _not thinking_ ahead, it is about _not building_ what hasn't proven that it is actually needed. 

I have seen quite often situations where the implementation of a product had to start from scratch, because the initial version could not scale with the load. Any yes, the initial implementation that was only used in some A/B test did not have to be ready for scale. But this doesn't imply that you should ignore the future requirement for scalability when designing the initial version, esp. when it can be expected that this requirement will come once the initial version is successful. You don't have to implement the initial version already in a fully scalable way (this would not be Agile, as your product has not yet proven that it is successful), but you should _think_ about how you can evolve it to a scalable version. And only when you then come to the conclusion that it economically doesn't make sense to already prepare for scalability, then you make (and communicate) a conscious business decision that you go into the A/B test with a to-be-thrown away prototype rather than with a first version of your future product.
</div>

You don't have to design all the details of the possible extensions you could dream of. But you need to develop a solid understanding of how you might possibly integrate such extensions in a backward compatible way, once one of the expected requirements materializes. Should you realize that preparing the design of your next version for future extensions would make the next version harder to understand/integrate (e.g. by introducing complex processing rules for your clients), then you make the conscious business decision that it is not worth risking the success of the next version for a not yet proven extension and with that accept the possible consequence of introducing a new major version should the need for the extension materialize. 
