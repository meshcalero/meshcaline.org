# Consumer Driven Response Types

A *meshcaline*{: .m} API supports _Consumer Driven Resource Types_, an API feature that allows you to manage backward incompatible changes in a controlled way, and by that help reducing the negative implications of those. Let's first look into two common problems where this becomes beneficial:

## Experimental features

As we have seen in [API Evolution & Compatibility](evolution.md) we have to carefully plan any change to our API to make sure that the change is (a) backward compatible to previous API version and (b) enables the introduction of additional features in the future. But this is not aways possible.

Daily practice shows us, that regardless how hard we try, it usually requires some iterations until a problem is well enough understood and a solution's API design is mature and sustainable. This seems to be a unresolvable conflict to backward compatibility. How can you experiment with your API design if you always have to make sure that you neither break the past nor the future? The purpose of an experiment is to learn from mistakes and then iterate until you have found an appropriate e solution.

One approach to overcome this dilemma is to run those experiments in an isolated environment. Only a set of pilot clients where you can manage the introduction of backward incompatible changes in the various iterations of the experiment is allowed to use this environment. Unfortunately this can become quite expensive, as you have to operate and maintain those dedicated environments. And in case you have more of those experiments ongoing at the same time and with different clients, then you might even need to have dedicated environments for specific combinations of experiments.

Another popular approach that tries to overcome that problem is "information hiding". By not mentioning the experimental features and resources in the API documentation, one assumes that only those clients participating in the experiment will know about the new capabilities (as only they get access to the corresponding documentation), and with that it is safe to run those experiments on production systems. Unfortunately this is a wrong assumption: Developers have a natural distrust to documentation and trust more what they see. As a result you will see that developers outside of the group of experiment participants will discover the new feature when they play with your API e.g. with [Postman](https://www.postman.com/product/api-client). And once they like what they see, they will start using it, regarless of what is written in your documentation. For exactly that reason [Google's API guidelines](https://cloud.google.com/apis/design/compatibility#changing_visible_behavior_of_existing_requests) don't allow their developers to ever change any behaviour or semantics of production API, _even when such behavior is not explicitly supported or documented_.

## Feature deprecation

For another type of problem let's pick a typical example: When you launched the API for concert ticket business you were a small local company, focusing on the US market and due to that amount in your `price` attrbiute in your API was implicitly in US$. As your business grows you want to expand into other regions and also sell tickets cross border with currency conversion. To avoid that you have to introduce a new major API version for that business expansion, you decide to introduce two additional data elements `ticket_price` and `customer_price` next to the existing `price`, both containing two attributes `amount` and `currency`. The old `price` attribute gets deprecated, but for your legacy clients who only request tickets for the US, the same API still does the job. 

After some time, you learn from your clients that they now use `ticket_price` and `customer_price` too, in order to participate in the new international business. As every deprecated feature adds a smell to your API (and the risk that also new clients would still use it) you would like to get rid of the old `price` attribute. Unfortunately deleting a mandatory element is a backward incompatible change. And you don't have big enough business case to introduce a new major version (neither for yourself, nor for your clients to migrate), as the smell (like many other forms of technical debt) doesn't directly hurt anyone.

## Telling me what you need!

The *meshcaline*{: .m} design allows you to address both problems with a single solution. It is derived from [Consumer Driven Contracts](https://martinfowler.com/articles/consumerDrivenContracts.html), where your clients tell you what behaviour they expect from you by providing you test cases that you embed into your API's build pipeline. Whenever a new version breaks an existing consumer contract your build breaks. Then you know that you have to either change your new version or talk with your client to resolve the problem before the next release. While this is a powerful tool when you have a close relationship with a handful of customers, it doesn't fit for public API with a potential large number of clients, some of which you might not even know.

Still, the idea of knowing what exactly your customers expect is compelling. Instead, of sending you tests, with *meshcaline's*{: .m} _Consumer Driven Response Types_ your clients use a query language on the data elements of your response types to tell you exactly which data elements they need. And then your API only responds exactly those elements.

Instead of inventing a new query language, *meshcaline's*{: .m} _Consumer Driven Resource Types_ reuse the well defined query language of Facebook's [GraphQL](https://graphql.org/).  

## What do you get from that?

With _Consumer Driven Resource Types_ you can rely on the "information hiding" approach for experimental features, as clients not participating in the experiment would not be able to discover the experminatal feature by chance when playing with your API. And you can also securely remove deprecated features when you see that they are no longer in use. While this is in theory still a backward incompatible change, as long as it doesn't break anything, who cares?

But there are more advantages that come with _Consumer Driven Resource Types_:

* By only responding what has been explicitly requested your clients can make best use of the available bandwidth to your API and you have eliminated the risk of [non-functional backwards incompatibility](evolution.md#non-functional-changes) introduced by additional large data elements. 
* Rather than ignoring items of untyped collections they are not aware of, your clients tell you which types of items the can/want to handle. This helps avoiding situations where clients have to iterate through multiple pages of collections before they get the elements they can use.
* You can measure the popularity of each individual data element, which gives you a significant better understanding about your clients than you would have if you would only track the access on the level of resources.






 

