---
layout: post
title: "Preserve state betwen refreshes"
tags: [vaadin 7, server]
---
{% include JB/setup %}

Last week, we had a look at the new Vaadin 7 `UI` that takes the place of the old version 6 `Window`. Though this evolution cleans window management in version 7, it brings a change that can have important consequences for those unaware of it.

In Vaadin 7, state is not kept between refreshes, whereas in v6, it was. Anyway, this is only the default behavior, and the Vaadin team provides us with the mean to do as we please. The only thing to do is to get a handle on the UI instance annotate it with `@PreserveOnRefresh`, like so:
{% highlight java %}
@PreserveOnRefresh
public class Vaadin7UI extends UI {

    public void init(VaadinRequest request) {
         ...
    }
}
{% endhighlight %}
You can find [here](https://github.com/nfrankel/More-Vaadin/tree/master/refresh-example) the sources of a little example application that let us play with this behavior. Let's try some things:

1. Call the URL with a named fragment beneath it (like #test). The page displays it right of the URI fragment label.
1. Change the fragment in the adress bar (like #testchanged) and refresh the page. The page should display the new URI fragment in the page.
1. Now, check the Preserve root checkbox. Change the fragment for the second time (like #testsecond) and behold, the page still displays the "testchanged" value. 

Whatever you want the behavior to be, Vaadin provides it :-)

