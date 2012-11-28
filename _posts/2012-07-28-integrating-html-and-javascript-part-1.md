---
layout: post
title : Integrating HTML and JavaScript - part 1
---

In Vaadin 6, extending existing components required creating subclasses and overriding the desired methods when possible. When not, this meant hacking protected methods and/or copying-pasting code. In this two-part serie, we'll have a look at how Vaadin 7 makes it easier for us to extend components through "composition" rather than inheritance:

+ This article will show us how to create an extension using HTML and the GWT API
+ In the [follow-up](/content/integrating-html-and-javascript-part-2), we will create an extension using purely JavaScript, without any GWT dependency! 

Vaadin 7 introduces the concept of extension. Extensions are features that can be attached to existing components (or applications) and may have a GUI part.

In order to illustrate our point, we'll create hover tooltips on hyperlinks.

Among Vaadin class hierarchy, an extension simply implements `Extension` (which is a marker inteface), but Vaadin provides `AbstractExtension` to ease our work. We just have to specify which components are extendable by this extension:
{% highlight java linenos %}
public class TooltipExtension extends AbstractExtension {
 
    // Only Link are allowed tooltips
    public void extend(Link link) {
 
        super.extend(link);
    }
}
{% endhighlight %}
Like any Vaadin components, extensions must have an associated client connector, that **has to be under the `client` subpackage** for it to be compiled. Connectors do the real work but need some knowledge of native GWT. For this reason, we won't go into much detail; just extends `AbstractExtensionConnector`, the rest is up to you. The next snippet displays HTML code when the mouse hovers over the client widget (and hides it when it moves out).
Note the `@Connect` annotation, just like any other connector. 
{% highlight java linenos %}
@Connect(TooltipExtension.class)
public class TooltipConnector extends AbstractExtensionConnector {
 
    @Override
    protected void extend(ServerConnector target) {
 
        final Widget hyperlink = ((ComponentConnector) target).getWidget();
 
        final VOverlay tooltip = new VOverlay();
 
        tooltip.add(new HTML("<div class='c-tooltip'>This is a static tooltip</div>"));
 
        hyperlink.addDomHandler(new MouseOverHandler() {
 
            @Override
            public void onMouseOver(MouseOverEvent event) {
 
                tooltip.showRelativeTo(hyperlink);
            }
 
        }, MouseOverEvent.getType());
 
        hyperlink.addDomHandler(new MouseOutHandler() {
 
            @Override
            public void onMouseOut(MouseOutEvent event) {
 
                tooltip.hide();
            }
 
        }, MouseOutEvent.getType());
    }
}
{% endhighlight %}
Finally, we have to connect our brand new extension to desired components:
{% highlight java linenos %}
Link morevaadin = new Link("More Vaadin", new ExternalResource("http://morevaadin.com/"));
 
new BasicTooltipExtension().extend(morevaadin);
{% endhighlight %}
While the second line may seem reversed, it let us restrict the type of component extends.

Our code has a big drawback: the text displayed in the tooltip is static. In order to provide a customizable tooltip, we just have to create a state, like for components.
{% highlight java linenos %}
public class TooltipState extends SharedState {
 
    private String display;
 
    public String getDisplay() {
     
        return display;
    }
 
    public void setDisplay(String display) {
     
        this.display = display;
    }
}
{% endhighlight %}
We also need to change our extension, to use the newly-created state:
{% highlight java linenos %}
@Connect(TooltipExtension.class)
public class TooltipConnector extends AbstractExtensionConnector {
 
    @Override
    protected void extend(ServerConnector target) {
 
        final Widget hyperlink = ((ComponentConnector) target).getWidget();
 
        final VOverlay tooltip = new VOverlay();
 
        String display = getState().getDisplay();
         
        tooltip.add(new HTML("<div class='c-tooltip'>" + display + "</div>"));
 
        hyperlink.addDomHandler(new MouseOverHandler() {
 
            @Override
            public void onMouseOver(MouseOverEvent event) {
 
                tooltip.showRelativeTo(hyperlink);
            }
 
        }, MouseOverEvent.getType());
 
        hyperlink.addDomHandler(new MouseOutHandler() {
 
            @Override
            public void onMouseOut(MouseOutEvent event) {
 
                tooltip.hide();
            }
 
        }, MouseOutEvent.getType());
    }
 
    @Override
    public TooltipState getState() {
 
        return (TooltipState) super.getState();
    }
}
{% endhighlight %}
Last but not least, we have to devise a way to get the tooltip text. Let's pretend we want to display the URL as a tooltip:
{% highlight java linenos %}
public class TooltipExtension extends AbstractExtension {
 
    public void extend(Link link) {
 
        Resource resource = link.getResource();
         
        String display = resource instanceof ExternalResource ? ((ExternalResource) resource).getURL().toString() : "???";
         
        getState().setDisplay(display);
 
        super.extend(link);
    }
 
    @Override
        public TooltipState getState() {
 
        return (TooltipState) super.getState();
    }
}
{% endhighlight %}
You can find sources for this article on [Github](https://github.com/nfrankel/More-Vaadin/tree/master/html-js-integration).

In the [next part](/content/integrating-html-and-javascript-part-2) of this serie, we'll look at how to integrate already existing JavaScript frameworks that provide tootlip features.
