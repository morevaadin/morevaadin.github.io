---
layout: post
title : Navigation basics
tags  : [server]
---

Vaadin implements the Single-Page Interface, meaning screen content can change but URL stays the same, as opposed to standard web application page navigation. Switching screen content is described in [Use views, not windows]({% post_url 2012-02-25-use-view-not-windows %}). Given SPI and before Vaadin 7, however, the only way to link a set of content with an URL was to use the [UriFragmentUtility](https://vaadin.com/api/com/vaadin/ui/UriFragmentUtility.html) component. With Vaadin 7, however, there's no need of such a component: all you need is to use the Navigator API.

The Navigation API is composed of three main classes:

<ul>
<li>The <code>View</code> interface lies at the root of the API. A view is an component (that should generally be a <code>Component</code>) that is managed by a <code>Navigator</code>. 
{% highlight java %}
public class ViewLayout extends VerticalLayout implements View {
 
    ...
     
    @Override
    public void navigateTo(String fragmentParameters) {}
}
{% endhighlight %}
Note that the <code>navigateTo()</code> method is just a hook: if you don't need to do anything, just leave it empty as in the above example. On the contrary, you could use the method to add a specific content on the view, depending on the fragment i.e. using the same view class but setting components based on the fragment.
<li>Next comes the <code>ViewDisplay</code> interface. This type is just a placeholder for a view to be displayed on. Navigator provides two implemetations: <code>SimpleViewDisplay</code> is a component that can be part of any container, while <code>ComponentContainerViewDisplay</code> is used internally and is a wrapper around a component container. In the latter case, just provide the container and the navigator takes care of the plumbing.
<li>Finally, the <code>Navigator</code> component is basically a map with views as values. Keys are strings that are used to both store the view in the navigator and to access it through a URL fragment. 
{% highlight java %}
Panel panel = new Panel();
 
Navigator navigator = new Navigator(panel);
 
navigator.addView("aview", new ViewLayout());
{% endhighlight %}
Note that if you provide content for your initial panel but dont make it accessible as a view, users won't be able to access it after the initial display.</ul>

From this point on, there are two ways to access a view:

+ Programmatically, through the `navigateTo()` method of the **navigator**
{% highlight java %}
navigator.navigateTo("aview");
{% endhighlight %}
+ By setting the URL to the Vaadin servlet's URL plus the view's name as the fragment. For example, if the Vaadin servlet is mapped to `/*`, just adding `#aview` to the context root will display the view. You can of course provide such an hyperlink for the user to click on. 

From this point on, you can manage views. In the next article, we'll handle view events and view providers.

Code for this article can be browsed and forked on [Github](https://github.com/nfrankel/More-Vaadin/tree/master/navigation-example).
