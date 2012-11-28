---
layout: post
title : Integrating Vaadin into legacy applications
tags  : [integration, server]
---

Though embedding Vaadin parts in legacy applications was covered in [Learning Vaadin](http://www.packtpub.com/learning-vaadin-rias/book), you may feel the need to go further toward this goal.

The description I made works, but your user interface has to be neatly separated into different rectangular zones for Vaadin-served `iframe`s or `div`s to take place. This clearly won't achieve the best integration results: we can do better with the [`ExternalLayout`](http://vaadin.com/directory/-/directory/addon/externallayout) add-on. It let you compose Vaadin components in your legacy application, irrelevant of where your Vaadin `div` is located on it: use-cases include a search box, a shopping cart, a menu bar, you name it!

In this article, we'll focus on a menu bar, situated at the top of the page while the main Vaadin application consists of a form located just under the title of a page (see below mockup).

<img src="/assets/images/mockup.png" width="640" height="480" />

The form creation is easy enough and out of the scope of this article. What we want, however, is to have both the menu bar and the form to be served by the same Vaadin application so as to have possible integration between them.

Here comes the external layout add-on; when embedding a Vaadin application inside a web page, we can put any Vaadin component on any placeholder inside this page, regardless of the position of the embedded application.

Detailed instructions for embedding an application can be found in [Learning Vaadin](http://www.packtpub.com/learning-vaadin-rias/book). In order to use the add-on, here are the necessary steps:

1. Get the library, either as a [download](https://vaadin.com/directory#downloading/externallayout/1096/1) or by using the following Maven dependency:
{% highlight xml linenos %}
<dependency>
   <groupId>org.vaadin.addons</groupId>
   <artifactId>externallayout</artifactId>
   <version>1.0</version>
</dependency>
{% endhighlight %}
2. Create the external layout by supplying both the placeholder's `id` and the desired component: 
{% highlight java linenos %}
ExternalLayout elayout = new ExternalLayout("menu", new EclipseMenuBar());
{% endhighlight %}
3. Finally, add the layout as a component to the window: 
{% highlight java linenos %}
wnidow.addComponent(elayout);
{% endhighlight %}
That's it! If you provide a simple HTML page with a `div` which id matches "menu", the menu bar will be rendered on it.

Complete sources for this article can be found on [GitHub](https://github.com/nfrankel/More-Vaadin/tree/master/external-layout-example).
