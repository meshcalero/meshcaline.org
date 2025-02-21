#About this blog

Working on a public web service API for some years now, [I (Andreas Schmidt)](#about-me) thought that posting a series of articles on the experiences made and conclusions/solutions derived from that might be worth to share with a wider audience. Hopefully this blog gives you some inspiration for your own API project and helps you avoiding some of the design iteration my team and I went through.

##Scope

You can build web service API for many different purposes. Each purpose comes with various aspects beyond the specific functionality it is supposed to deliver. In most cases other aspects will also have a massive influence on the question how good a given design serves the purpose of the API, e.g.:

* **Business**: How can I monetize the API offering? 
* **Audience**: For whom do you provide the API? How will those users of the API want, need, or expect to use the API? 
* **Client platforms**: How will clients connect to the web service API? What are the capabilities of those client platforms? 


In this blog I will focus on solutions for a very specific subset of API purposes, simply because it is the domain I've been focusing on during the last years: 

>Public API offerings, where the API provides access to a commercial offering that is expected to be called directly from within a client application on the end-user's device.

In this domain several aspects play a very important role:

* Public APIs get developed for an anonymous audience. You might have a closer relationship with some of your users, but the majority you will never meet f2f or have any direct communication with. Most developers will learn about your offering through some form of communications and then spend some time evaluating your product. If they get into your API easily and find quality and terms appropriate you have a chance to get a new customer. If not, there is a high probability that you will never receive any feedback from them and therefore have no chance to learn what they disliked.
* A commercial product competes on the market. For this it needs differentiation. Assuming that monetization of a commercial API is based on its usage, you try to bind customers as long as possible and motivate them to use your product more often. Keep in mind that this to a certain extent contradicts with your user's goals: While they might love your solution, "provider lock-in" is something that most customers don't like. That means, you must find the right balance.
* Any remote call has an impact on the performance characteristics of the calling application. API accessed directly from client applications need to take this into account, especially when those connect through mobile networks. In addition the API must be able to handle limited capabilities that specific client platforms may have.

You might have already realized that there is no "silver bullet" for API design. A given design paradigm might be the ideal solution for one purpose, but may result in a complete disaster if applied for a different one. So whenever you find some popular API you like, just copying its design paradigm into your domain won’t automatically make your API useful, loved and popular as well. At the same time, copy-with-pride can be a very good practice in API design, as it can dramatically reduce the learning curve of your potential users. Particularly when you compete with other solutions in a given domain, this may be the significant reason, why developers chose your API for further evaluation instead of another one: they simply feel already familiar with the API from the start. Therefore you not only need to understand the building blocks of an API design you like, you also have to understand which aspects of the purpose of a given API they support. While this blog is about a technical domain, quite often I will give non-technical reasons why I believe the proposed solution is worth to consider.

I assume that many of the topics I will touch in this blog can also help API designers in other domains, and I would love to receive your feedback on that.

###Be warned

Not all of the concepts described in these articles did find their way into the API I've been working on and by that had a chance to proof their value in the field. So better don't count on everything being perfect. If you find room for improvement, or confusing (hopefully not contradicting) aspects, please drop a comment on the post <script>if( window.location.pathname=="/about/" ){ var t="org", d="meshcaline", n="meshcalero"; var m=n+"@"+d+"."+t; document.write('or send me an email to <a href="mailto:'+m+'">'+m+'</a>');}</script>

##About me

If you're interested to know more about me you might want to visit my [Linkedin Profile](https://www.linkedin.com/in/andreas-schmidt-bln/) as a starting point.