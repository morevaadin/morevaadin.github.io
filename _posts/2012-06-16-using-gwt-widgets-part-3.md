---
layout: post
title : Using GWT widgets - Part 3
tags  : [vaadin 7, client, gwt]
---

This is the final part in our articles serie regarding using GWT widgets in Vaadin 7. In the [first part]({% post_url 2012-06-02-using-gwt-widgets-part-1 %}), we looked at how to wrap GWT widgets in Vaadin components. In the [second part]({% post_url 2012-06-09-using-gwt-widgets-part-2 %}), we detailed how to configure widgets from components. In this third and final part, we'll see how to intercept events coming from the client side in our components.

The central component is client-server communication is an interface extending `ServerRpc`. In our context, the widget is a button so the only method of the interface should be something like `click()`:
{% highlight java linenos %}
public interface BootstrapButtonRpc extends ServerRpc {
 
    void click(MouseEventDetails mouseEventDetails);
}
{% endhighlight %}
Note we have a single MouseEventDetails parameter taken from Vaadin API, that should be enough to convey relevant information.

On the client side, several steps are necessary:

+ In the connector, create a proxy around the RPC interface
+ Let the connector implement GWT's `ClickHandler.onClick()` and call the RPC method defined in the interface
+ Add the connector as the button's click handler 
{% highlight java linenos %}
public class BootstrapButtonConnector extends AbstractComponentConnector implements ClickHandler {
 
    private BootstrapButtonRpc rpc = RpcProxy.create(BootstrapButtonRpc.class, this);
 
    public BootstrapButtonConnector() {
         
        getWidget().addClickHandler(this);
    }
 
    public void onClick(ClickEvent event) {
 
        MouseEventDetails details = MouseEventDetailsBuilder.buildMouseEventDetails(
            event.getNativeEvent(), getWidget().getElement());
 
        rpc.click(details);
    }
 
    // Code from previous articles goes here
}
{% endhighlight %}
On the server side, we have to:

+ create a custom event type that extends `com.vaadin.ui.Component.Event`
+ implement the RPC interface so that calling `click()` fires a new instance of our custom event type
+ register a new instance of the RPC implementation
{% highlight java linenos %}
public class BootstrapButton extends AbstractComponent {
 
    private BootstrapButtonRpc rpc = new BootstrapButtonRpc() {
     
        @Override
        public void click(MouseEventDetails details) {
 
            fireEvent(new BootstrapClickEvent(BootstrapButton.this));
        }
    };
 
    public BootstrapButton(String caption) {
 
        setText(caption);
        setImmediate(true);
         
        registerRpc(rpc);
    }
 
    // Code from previous articles goes here
}
{% endhighlight %}
Now, when the user clicks on the button widget, the GWT framework calls the `onClick()` method. In there, the Vaadin framework bridges the call from the RPC client-side code to the RPC server-side code. On the server-side, the `click()` implementation fires a new event: it's just for us to listen to such events in standard Vaadin API.

In conclusion, I hope this serie convinced you wrapping GWT widgets is not hard if you follow the tips described here. You can find the whole code on [Github](https://github.com/nfrankel/More-Vaadin/tree/master/custom-component-example).

