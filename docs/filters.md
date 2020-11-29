# Consumer Driven Response Types

A *meshcaline*{: .m} API supports _Consumer Driven Resource Types_, a feature that allows you to manage backward incompatible changes in a controlled way, and by that help reducing negative implications of those. Let's first look into two common problems where this becomes beneficial:

## Experimental features

As we have seen in [API Evolution & Compatibility](evolution) we have to carefully plan any change to our API to make sure that the change is (a) backward compatible to previous API version and (b) enables the introduction of additional features in the future. But this is not aways possible

Daily practice shows us, that regardless how hard we try, it usually requires some iterations until a problem is well enough understood and a solution design is mature and sustainable. This seems to be a unresolvable conflict. How can you experiment with your API design if you always have to make sure that you neither break the past nor the future? The purpose of an experiment is to learn from mistakes and then iterate until you have the solution.

One approach to overcome this dilemma is to run those experiments in a controlled environment with a set of pilot clients where you can manage the need to also introduce non-backward compatible changes in the various iterations of the experiment. Unfortunately this can become quite expensive, as you have to operate and maintain those dedicated environments. And in case you have more of those experiments ongoing at the same time and with different clients, then you might even need to have dedicated environments for specific combinations of experiments.

Another popular approach that tries to overcome that problem is "information hiding". By just not mentioning the new features and resources in an API documentation, one assumes that only those clients participating in the experiment will know about the new capabilities (as only they get access to the corresponding documentation), and then those experiments can also run on production systems. Unfortunately this is a wrong assumption: Developers have a natural distrust to documentation and trust more what they see. As a result you will see that developers outside the group of experiment participants will discover the new feature when they play with your API e.g. with [Postman](https://www.postman.com/product/api-client). And once they like what they see, they will start using it, regarless of what is written in your documentation. For exactly that reason [Google's API guidelines](https://cloud.google.com/apis/design/compatibility#changing_visible_behavior_of_existing_requests) don't allow their developers to ever change any behaviour or semantics of production API, _even when such behavior is not explicitly supported or documented_.

## Feature deprecation

For another type of problem let's pick a typical example: When you started your with your concert ticket API, you were focusing on the US market and due to that amount in your `price` attrbiute in your API is implicitly in US$. After some time your business grows and you want to expand into other markets, including selling tickets cross border with currency conversion. To avoid that you have to introduce a new major API version for that business expansion, you decide to introduce two additional data elements `ticket_price` and `customer_price` next to the existing `price`, both containing two attributes `amount` and `currency`. The old `price` attribute gets deprecated, but for your old clients who only request tickets for the US, it still does the job. 

After some time, you learn that from your clients that they have migrated to use ticket_price and customer_price in order to participate in the new international business too. As every deprecated feature adds a smell to your API (and the risk that also new clients start using it) you would like to get rid of it. But as deleting a mandatory element is a backward incompatible change and the smell is not worth to launch a new major version of your API just for the removal of the `price` attribute you don't have a strong enough business case for that. 

## Telling me what you need!

The *meshcaline*{: .m} design allows you to address both problems with a single solution. It is based on idea of [Consumer Driven Contracts](https://martinfowler.com/articles/consumerDrivenContracts.html), where your clients tell you what behaviour they expect from you by providing you test cases that you embed into your API's build pipeline. Whenever a new version breaks an existing client contract your build breaks and you know that you have to either change the new version or talk with your client to resolve the problem before. While this is a powerful tool when you have a close relationship with a handful of customers, it doesn't fit for public API with a potential large number of clients, some of which you might not even know.s 

Stil the idea is compellting. With *meshcaline*{: .m} a client doesn't sends you tests for your build pipeline. Instead, with _Consumer Driven Resource Types_ you receive with each request which data elements they expect, by using a query language on the data elements of your response types. And then your API only responds exactly those elements.

Instead of inventing a new query language, *meshcaline's*{: .m} _Consumer Driven Resource Type_ feature reuses the well defined query language of Facebook's [graphQl](https://graphql.org/).  

## What do you get from that?

With _Consumer Driven Resource Types_ you can rely on the "information hiding" approach for experimental features, as clients not participating in the experiment would not be able to discover the experminatal feature by chance when playing with your API. And you can also securely remove deprecated features that are no longer in use. While this is in theory still a backward incompatible change, as long as it doesn't break anything, who cares?

But there are more advantages that come with _Consumer Driven Resource Types_:

* By only responding what has been requested your clients can make best use of the available bandwidth to your API and you have eliminated the risk of [non-functional backwards incompatibility](/evolution#non-functional-changes) introduced by additional large data elements. 
* Rather than ignoring items of untyped collections they are not aware of, your clients tell you which types of items the can/want to handle. This helps avoid situations where client have to iterate through multiple pages of collections before they get elements they can use.
* You can measure the popularity of each individual data element, which gives you a significant better understanding about your clients than you would have if you would only track the access on the level of resources.






 

