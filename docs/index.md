### .. evolving a design paradigm for hypertext based web service API
<p/>
#### *Welcome to meshcaline.org!*{: .m} 

This blog contains a series of articles around *meshcaline*{: .m}, a design paradigm for hypertext based web service API. Although the ideas of the *meshcaline*{: .m} design are appropriate for many classes of API, it creates most value for API that are directly called from end-user devices (browser, mobile app) and that enable end-user facing business process implementations  

##### [REST in Piece](rest-in-peace)

Everybody claims to have REST API theses days. Is this correct? And is REST really the right design paradigm for APIs? Analyzing Roy Fielding's rules for REST APIs raises many concerns for the applicability of REST (as defined by the inventor) for public service API offerings. [...more](rest-in-peace)

##### [Basic meshcaline](basics)

What are the fundamental concepts and design paradigms of *meshcaline*{: .m} and how does it implement hypertext? An introduction for getting started with *meshcaline*{: .m} and a must read that sets the scene for most of the other posts. [...more](basics)

##### [Why Hypertext API?](hypertext)

Despite the success of the hypertext concept in the *browsealbe* web, in the *programmable* web of service API, hypertext is only rarely used. Read how API users, API developers and API business benefit from a hypertext based service API. [...more](hypertext)

##### [API Permalinks](permalinks)

Individual resources in a *meshcaline*{: .m} API represent steps that guide an end-user through a business process. As such they are by default temporary resources. The simple permalink pattern enables API users to keep references to steps from one business process and start new business processes from there in subsequent sessions. [...more](permalinks)

##### [Paginateable Collections](pagination)

Collections of objects with potentially large number of items risk that the size of resource representation grows beyond a reasonable limit. To avoid that risk *meshcaline*{: .m} recommends a standard representation pattern for paginateable collections. [...more](pagination)

##### [API Evolution & Compatibility](evolution)

Now that we have the basic ingredients to design the API for our product, let's analyze what we can do to iteratively evolve our product while at the same time provide a predictable and reliable service to our clients. [...more](evolution)

##### [Consumer Driven Response Types](filters)

By combining elements of consumer driven contracts and response filtering *meshcaline*{: .m} allows you to better understand how your clients use your API and with that enable you to control experimental features and to manage the introduction of incompatible changes [...more](filters)