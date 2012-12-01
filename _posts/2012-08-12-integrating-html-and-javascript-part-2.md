---
layout: post
title : Integrating HTML and JavaScript - part 2
---

In the [previous article]({% post_url 012-07-28-integrating-html-and-javascript-part-1 %}), we successfully integrated a custom-made tooltip over our hyperlinks. In this article, we'll integrate an already existing tooltip library.

Since we already played with Twitter Bootstrap library, we'll try to reuse their code.

If you follow this site regularly, we already used the Bootstrap library, or more exactly the GWT porting of the library in the [Using GWT widgets]({% post_url 2012-06-02-using-gwt-widgets-part-1 %}) serie. This time, we'll use the library directly, without any third-party GWT wrapper.

The process mirrors what we already did when creating custom HTML code.

+ The first step is to create the server part, which is the extension. For JavaScript components, note that there exists a specialized class, JavaScriptExtension. Moreover, we have to tell Vaadin which scripts will be used: those scripts can either be local to the webapp or available through an absolute URL. It's done through a simple annotation. 
{% highlight java linenos %}
@JavaScript({ "https://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.js", "bootstrap.js", "bootstrap_connector.js" })
public class JavascriptTooltipExtension extends AbstractJavaScriptExtension {
 
    public void extend(Link link) {
 
        Resource resource = link.getResource();
 
        String display = resource instanceof ExternalResource ? ((ExternalResource) resource).getURL().toString() : "???";
 
        getState().setDisplay(display);
 
        super.extend(link);
         
        attachTooltip();
    }
 
    protected void attachTooltip(Object... commandAndArguments) {
 
        invokeCallback("attach", commandAndArguments);
    }
 
    @Override
    protected Class<? extends ClientConnector> getSupportedParentType() {
 
        return Link.class;
    }
 
    @Override
    public BootstrapTooltipState getState() {
 
        return (BootstrapTooltipState) super.getState();
    }
}
{% endhighlight %}
+ Next, we have to provide the aforementioned local scripts. This is done by packaging them in the WAR as the previous class. So, if the extension was in the `com.morevaadin.vaadin7.html.js` package, put the scripts in exactly the same one.
+ Last but not least, we have to provide the JavaScript glue that bind the components together. You probably noticed the `invokeCallback()` in the server code: it's the connector between server and client code. Vaadin will search for an anonymous JavaScript function under the package name (where dots have been replaced by underscores) that provide a subfunction named as the first argument of the `invokeCallback()` function. This is the reason why we parameterized the `bootstrap_connector.js` script previously (though you're free to call it what you want).
{% highlight java linenos %}
window.com_morevaadin_vaadin7_html_js_JavascriptTooltipExtension = function() {
 
    this.attach = function(options) {
 
        var connectorId = this.getParentId();
 
        var element = this.getElement(connectorId);
 
        var a = element.childNodes[0];
         
        a.rel = "tooltip";
        a.title = this.getState().display;
 
        $(a).tooltip();
    }
}
{% endhighlight %}
The code itself depends on the particular library.

There are two important things to note:

+ Nothing prevents you for providing multiple features in your server-side API since you can invoke whichever callback you want on the client side.
+ In our case, we have access to the jQuery API since it was configured in the extension.

Compared to wrapping our own HTML, wrapping already existing JavaScript is a breeze. In conclusion, Vaadin 7 makes it possible to easily integrate JavaScript without the need for GWT wrappers. If (a part of) your team is fluent in JavaScript, it's a real asset for your applications.

The code of this article can be found on [GitHub](https://github.com/nfrankel/More-Vaadin/tree/master/html-js-integration).
