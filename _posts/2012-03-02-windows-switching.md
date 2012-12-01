---
layout: post
title: "Views content switching"
tags: [ vaadin 6, server]
---
{% include JB/setup %}

In my [last article]({% post_url 2012-02-25-use-view-not-windows %}), I definitely advised that when needing to radically change components displayed on the screen, you need to switch the main window's contents - the view - and not the window itself.

Fortunately, this confusion is not possible anymore in Vaadin 7 since the application object and the main window are merged into the [UI] class.
{% highlight java %}
package com.morevaadin.vaadin7.example;
 
import com.vaadin.server.VaadinRequest;
import com.vaadin.ui.Label;
import com.vaadin.ui.UI;
import com.vaadin.ui.VerticalLayout;
 
@SuppressWarnings("serial")
public class Vaadin7UIApplication extends UI {
 
    @Override
    protected void init(VaadinRequest request) {
 
        VerticalLayout layout = new VerticalLayout();
 
        layout.setMargin(true);
 
        Label label = new Label("Hello Vaadin user");
 
        layout.addComponent(label);
 
        setContent(layout);
    }
}
{% endhighlight %}
Note that `UI` instances have no root content (unlike Vaadin 6's `Window`): this means we have to set it explicitly, like the layout in the previous snippet.

As a corollary, this also means a `UI` class has to be configured in the web deployment descriptor for the Vaadin servlet (and not an `Application` class anymore). This directly translates into the following snippet:
{% highlight xml %}
<servlet>
    <servlet-name>Vaadin 7 Root Example</servlet-name>
    <servlet-class>com.vaadin.server.VaadinServlet</servlet-class>
    <init-param>
        <param-name>UI</param-name>
        <param-value>com.morevaadin.vaadin7.example.Vaadin7UIApplication</param-value>
    </init-param>
</servlet>
{% endhighlight %}
Conclusion: Vaadin 7 is less confusing than Vaadin 6 for managing full-screen windows. Besides, it achieves the same result in as many lines of code: it's a definite step toward cleaner windows management.

The archive that highlights the above code can be downloaded [here](http://morevaadin.com/sites/default/files/articles/windows-switching-vaadin-7/vaadin-7-example.zip).
