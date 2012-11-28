---
layout: post
title : Using GWT widgets - Part 2
tags  : [vaadin 7, client, gwt]
---

In [part 1](/content/using-gwt-widgets-vaadin-7-part-1) of Using GWT widgets, we've seen how to wrap a Vaadin component around a GWT widget so that we're able to manipulate on server-side components. For the time being, we are unable to configure the widget though. We're lacking a vital part in Vaadin-GWT teaming: the shared state.

Shared state is exactly what it means, a placeholder for both server component and client widget to share the same information. In our Bootstrap example, it will enable us to set the text shown in our brand-new BootstrapButton instances. Of course, other attributes can be configured by following the same path.

First thing first, we have to create a `BootstrapButtonState` class extending `ComponentState`. That class has to be located in the **same package as the connector** (see [part 1](/content/using-gwt-widgets-vaadin-7-part-1) for a quick refresher on the connector if the need be). The next step is to add the relevant property. 
{% highlight java linenos %}
public class BootstrapButtonState extends ComponentState {
 
    private String text = "";
 
    public String getText() {
 
        return text;
    }
 
    public void setText(String text) {
 
        this.text = text;
    }
}
{% endhighlight %}
Since connectors are listeners of change state events, we now need to override its `onStateChanged()` method. 
{% highlight java linenos %}
public void onStateChanged(StateChangeEvent stateChangeEvent) {
 
    super.onStateChanged(stateChangeEvent);
 
    BootstrapButtonState state = getState();
 
    VBootstrapButton button = getWidget();
 
    button.setText(state.getText());
}
{% endhighlight %}
Notice we also have overriden `getWidget()` and `getState()` to return the adequate type for our Bootstrap button (code not shown).

The final step is to bridge the server component to the shared state. 
{% highlight java linenos %}
public class BootstrapButton extends AbstractComponent {
 
    public BootstrapButton(String caption) {
 
        setText(caption);
        setImmediate(true); // It's a button!
    }
 
    @Override
    public BootstrapButtonState getState() {
 
        return (BootstrapButtonState) super.getState();
    }
 
    public String getText() {
 
        return getState().getText();
    }
 
    public void setText(String text) {
 
        getState().setText(text);
 
        requestRepaint();
    }
}
{% endhighlight %}
Note the `requestRepaint()` is mandatory to mark the state as changed.

That's it: we have managed to change the button's text from the server code! From this point on, we can also configure both style and size to be able to mix and match between all attributes. Sources for this article can be found on [GitHub]((https://github.com/nfrankel/More-Vaadin/tree/master/custom-component-example).

In the [final part](/content/using-gwt-widgets-vaadin-7-part-3l) of this serie, we'll look at how to send client events to the server.
