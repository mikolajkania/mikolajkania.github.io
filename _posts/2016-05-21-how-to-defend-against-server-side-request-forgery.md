---
layout: post
title: How to defend against a server side request forgery?
tags:
- server side request forgery
- security
- node.js
- express
- angularJS
---

Server side request forgery occurs when attacker enters one application and is able to use to it to perform some activity on another application(s). It can be scaning internal network, calling services or making request to another website - our case. Note, that a hacked application would be responsible for an attack - as it produces a call! - not hacker's machine. More information can be found on numerous sites on the Internet, e.g. [here](https://blogs.mcafee.com/mcafee-labs/server-side-request-forgery-takes-advantage-vulnerable-app-servers/).

An example I created consists of two applications. *Hacked* is a Node.js/Express/AngularJS app that calls server with an URL of an image it would like to display. Frontend makes a call to backend which retrives a graphic.  

*Listener* is a logging app that listens to incoming calls and logs those with url containg */hack/was/made* string. Replacing an image URL with prepared one will make a *Hacked* service create unwanted request. Configuration of both apps is [described on my GitHub page](https://github.com/mikolajkania/ServerSideRequestForgery).

Let's move to a hacking part. Enter the *Hacked* app and click a button displaying qout and image of well-known author. Next check that *connections.log* in *Listener* service is still empty, as provided application works just fine downloading expacted data. Then open an favourite browser debugger and replace calling code in *hacked-ctrl.js* with this:

{% highlight js %}
$http.get('/getImage?' +
	'url=http://localhost:3000/hack/was/made&' +
	'author=coelho')
	.success(function (image) {
		$scope.image = image;
	})
{% endhighlight %}

After saving the changes and clicking the button again, *connections.log* will contain multiple rows with logged connections and out job will be done.

![placeholder](https://raw.githubusercontent.com/mikolajkania/mikolajkania.github.io/master/_images/2016-05-21-hacked-app-verified.png "Logged http requests")

In order to make my example straight I made a few simplifications:

- in the real situation webapp that is being attacked could have some kind of authentication so the attacker has to break it first
- in the real webapp an address of image repository would not be placed in JavaScipt file but on a server side; it can be replaced, however, when webapp calls another service with this url passed through REST API
- the attacker would probably prepare a script and automate whole process

How can you defend against this threat? If you have to pass URL try to include only relative part in a requst (*/image23* rather then the full address). If the requirement is to allow any URL (for example due to existance of multiple graphics servers), then you should create a white list of addresses that are allowed and validate icoming calls.

I hope that this short post make you understand the threat that comes from insecure code and explains how to prevent it.