---
layout: post
title: "Getting data about requests"
tags: [vaadin 7, server]
---
{% include JB/setup %}

I confess, Vaadin 6 was good, but Vaadin 7 is even better. This article will detail most nastiness that is managed by this new version so that we only have to code a few lines to achieve our goals. One of such goal is the search of information about the request and the client.

In Vaadin 6, it was possible, but cumbersome for we had to force our application to implement the `HttpServletRequestListener` interface. The interface let us access the servlet request (as well as the servlet response), and then we were on our own.

In Vaadin 7, the Vaadin team identified recurring needs to some piece of data and provided a Vaadin API to access them easily; for example:

1. The user agent type and its version (major and minor)
1. The user's locale to initialize resources bundles
1. The screen size, as well as the underlying viewpoint size to adapt the layour to the available space
1. The URL fragment (#) to restore data 

Vaadin 7 `UI`'s `init()` method takes a `VaadinRequest` as a parameter, that is the entry point into these informations. Behold how we could address the above needs:
{% highlight java %}
public class Vaadin7Root extends Root {
 
    @Override
    public void init(WrappedRequest wrapped) {
 
        BrowserDetails details = wrapped.getBrowserDetails();
 
        WebBrowser browser = details.getWebBrowser();
 
        // Need #1
        browser.getBrowserApplication();    // User agent
        browser.getBrowserMajorVersion();   // Major version
        abrowser.getBrowserMinorVersion();  // Minor version
 
        // Need #2
        browser.getLocale();                // User locale
         
        // Need #3
        browser.getClientHeight();          // Browser height
        browser.getClientWidth();           // Browser width
        browser.getScreenHeight();          // Viewpoint height
        browser.getScreenWidth();           // Viewpoint width
 
        // Need #4
        details.getUriFragment());          // URI fragment
    }
}
{% endhighlight %}
No more dirty plumbing into the `ServletRequest`, welcome to a world of easy reading!

The ready-to-use WAR file is available [here](http://morevaadin.com/sites/default/files/articles/getting-data-about-requests-vaadin-7/vaadin7-init.war)(sources included).
