---
layout: post
title : Another way to decouple your server components
tags  : [vaadin 7, server]
---

One of Vaadin strongest points is the way you can reuse components from project to project inside JARs. This can only be achieved if these components are nicely decoupled from one another.

In Learning Vaadin, I showed how to decouple them using DI. However, this way only handles "static" coupling - the way a component depends on another to be displayed, it doesn't handle "dynamic coupling", the way a component depends on another to do something. For example, a list box values could depend on the selected value of another list box.

In order to achieve this kind of behavior, it would be nice if components could send and receive events.

Vaadin doesn't provide this feature out-of-the-box but there are many frameworks that tackle events management: CDI (aka JSR 299), Google's Guava `EventBus` and so many more.

In this article, I chose Guava for the following reasons: it doesn't depend on a platform, it's usable with Java 5 and it only draws a single dependency, making it nearly self-contained. As the use-case, let's have two combo boxes, one for countries, the other for capitals. When the country changes, the capital has to be selected.

It's of first importance that the combos do not know about each other, so let's bury them underneath a hierarchy of components.
{% highlight java linenos %}
public class FirstComponent extends CustomComponent {
 
    private ComboBox cb = new ComboBox("Country");
}
{% endhighlight %}
Guava event management is based entirely on the `EventBus` class. Some steps are necessary:

+ Listener methods should be annotated with Guava's `@Subscribe`
+ Listener method should declare as their only parameter the type of event they're interested in handling:
{% highlight java linenos %}
@Subscribe
public void changeCapitalOnCountryChange(CountryChangedEvent event) {
 
    ...
}
{% endhighlight %}
Note there's no enforcement on the event type's hierarchy, you can basically pass whatever strikes your fancy (provided it's not a simple type).
+ The bus registers listener instances.
{% highlight java linenos %} 
bus.register(new SecondComponent());
{% endhighlight %}
+ Producers have to post new events into the bus. In our case, this has to be done when a `ValueChangeEvent` is received (from the client-side): 
{% highlight java linenos %}
public void valueChange(ValueChangeEvent vcEvent) {
 
    bus.post(new CountryChangedEvent());
}
{% endhighlight %}
Basically the only coupling you'll ever have is between the producer and the event bus. 

In order to reduce the previous coupling, let's design the producer's constructor to take the bus as a parameter, so as to be able to inject it: 
{% highlight java linenos %} 
public class FirstComponent extends CustomComponent {
 
    public FirstComponent(final EventBus bus) {
 
        ...
    }
}
{% endhighlight %}
The complete source code is available on [GitHub](https://github.com/nfrankel/More-Vaadin/tree/master/eventbus-example).
{% highlight java linenos %} 
{% endhighlight %}
