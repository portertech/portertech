---
title: "Phones, Servers, and User Experience"
date: 2010-07-19
draft: false
toc: false
images:
tags:
  - mobile
  - api
  - sinatra
  - mongodb
  - nginx
---

Successful mobile applications are enjoyable to use, offering a intuitive, useful, and responsive set of features.

GOOD mobile developers spend a large portion of their time designing their user interface, followed by performance optimization. GREAT developers know that creating a better experience for their users is far more important than adding features.

As the demand for data-rich mobile applications grows, producing a simple, easy to use interface becomes more challenging, usually accompanied by performance issues. Application optimization can only go so far, as developers are still limited by the device. Only so much processing power can be squeezed into a phone, keeping battery life in mind and not burning a hole in your pants. This is where web API's come into play, tons of horse power sitting up in the "cloud" with buckets of data. Web API's can offer access to a vast amount of data and the ability to offload computation, reducing the responsibilities of the mobile application. As users become thirsty for more data, with the expectations that it should be delivered quickly, web API performance should be of serious concern.

![web api](/images/2010-07-19/web-api.png)

While I was at Gastown Labs, we built an iPhone application that required a decent amount of geo location data, plotted on a map while a user panned and zoomed. In the beginning we had to determine the best way to handle that data. As we saw it, there were two possible solutions: store the dataset locally and parse/process it all on the phone, or build a web API to handle the geospatial processing and serve the results to the mobile application.

Storing the entire dataset locally would provide the ability to use the application completely offline, without any network connectivity. Offline usage is great, however, parsing and processing causes the phone to become completely unresponsive.

We chose to build a simple, RESTful web API (SOAP sucks) and we didn't look back. Compared to storing the dataset locally, it was night and day, even with limited connectivity the performance was outstanding. Users could pan and zoom on the map, loading location data from the web API, leaving them satisfied (you can get your data and use it too). A fantastic side effect of using a web API is flexibility, data updates and additions can happen without technical intervention, completely curated by the application owner.

The web stack:

![web stack](/images/2010-07-19/api-web-stack.png)
