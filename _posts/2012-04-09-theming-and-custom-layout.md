---
layout: post
title : Theming and custom layout
tags  : [vaadin 6, client]
---

In [Learning Vaadin](http://www.packtpub.com/learning-vaadin-rias/book), I deliberately left out theming, because I wholeheartedly believe a whole book could be written on the topic. This article is an attempt at giving an overview of the available theme and layout features in Vaadin.

At the most basic level, developers can change the look and feel of the application with the simple `setTheme()` method.
{% highlight java linenos %}
public class MyApplication extends Application {
 
    public void init() {
 
        setTheme("reindeer");
    }
}
{% endhighlight %}
Now, given the [available themes](http://vaadin.com/directory#browse/type/2) in Vaadin's directory, it can get you very far indeed. For example, we just have to put the [ReindeerMods](http://vaadin.com/directory#addon/reindeermods) in the `WEB-INF` lib directory and programmatically change the theme and presto, we are done.
{% highlight java linenos %}
public class MyApplication extends Application {
 
    public void init() {
 
        setTheme("reindeermods");
    }
}
{% endhighlight %}
The next logical step is to be able to create our own theme. At the most basic level, a theme is just a CSS named `styles.css` and located under `WEB-INF/VAADIN/themes/<my-theme-name>`.

For a well rounded theme, we have to redefine all possible Vaadin classes. Since it's a dauting task to start from scratch, it's advised to start from an available theme. Just import the wanted CSS into your own and redefine what is needed:
{% highlight java linenos %}
@import "../reindeermods/styles.css";
 
.v-menubar .v-menubar-menuitem {
  cursor: pointer;
  font-weight: bold;
}
{% endhighlight %}
Since it shows the height of my graphics skills, we'll let it at that on the CSS front.

Vaadin themes can also package custom layouts. Custom layouts are useful when one wants to have more versatility than vertical and horizatontal layout. In a custom layout, we define a HTML template with placeholders identified by the location attribute. For example:
{% highlight html linenos %}
<div location="top" id="top"></div>
<div>
    <span location="bottom" id="bottom"></span>
    <span location="left" id="left"></span>
</div>
{% endhighlight %}
This HTML snippet should be located under `WEB-INF/VAADIN/themes/<my-theme-name>/layouts/my-layout.html`. In order to use a custom layout, instantiate a `CustomLayout` object and pass the layout name as a parameter.
{% highlight java %}
CustomLayout layout = new CustomLayout("my-layout");
{% endhighlight %}
Then, when adding components, we use the overloaded `addComponent(Component, String)` method. The second parameter references the location and tells Vaadin where it should put the component in the custom layout.
{% highlight html linenos %}
layout.addComponent(menuBar, "top");
layout.addComponent(new Button("Does nothing"), "bottom");
 
VerticalLayout vLayout = new VerticalLayout();
 
vLayout.addComponent(new InlineDateField());
vLayout.addComponent(new TextField("", "Nothing to put in here"));
vLayout.setSpacing(true);
vLayout.setMargin(true);
 
layout.addComponent(vLayout, "left");
{% endhighlight %}
As is seen in the previous snippet, nothing prevents us to add containers instead of mere components. Likewise, custom layouts can be used at any level of granularity, from base components to windows. Using both options in combination, custom layouts can provide a powerful way to configure our GUI.

Astute readers may have noticed that a layout belongs to a theme. This has important consequences:

1. Custom layouts can be shipped in JARs, wrapped in themes
1. When changing a theme, we can change the layout accordingly. As a corollary, this also means we **need** to provide a layout when in this case 

Finally, what makes theme so powerful in Vaadin is that they can be changed programmatically or by the user. The `setTheme()` method reloads the GUI if a new theme is set.

A full-fledged example illustrating the various points in this article is available on [Github](https://github.com/nfrankel/More-Vaadin/tree/master/theming-example), to play with as you like.
