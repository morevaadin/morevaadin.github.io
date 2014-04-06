---
layout: post
title: "The right usage of lambdas with event listeners"
tags: [vaadin 7, server, java 8]
---
{% include JB/setup %}

There's been much advertisement about how Java 8's lambdas bring a big asset to Vaadin development. The following is a pre-Java 8 usage of a `ClickListener`:
{% highlight java %}
button.addClickListener(new Button.ClickListener() {

    @Override
    public void buttonClick(Button.ClickEvent event) {

        container.addComponent(new Label("Button clicked"));
    }
});
{% endhighlight %}
Java 8's lambda make it much easier, as this snippet shows:
{% highlight java %}
button.addClickListener(e -> container.addComponent(new Label("Button clicked")));
{% endhighlight %}
However, it is my opinion, that using lambdas this way leads to bad architecture and unmaintainable design, for it strongly couples GUI components and behavior.

Object-oriented design should follow [SOLID](http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29) principles, first among them being the [Single Responsibility Principle](http://en.wikipedia.org/wiki/Single_responsibility_principle):

>In object-oriented programming, the single responsibility principle states that every class should have a single responsibility, and that responsibility should be entirely encapsulated by the class. All its services should be narrowly aligned with that responsibility.

Using lambdas as in the previous snippet just defeats this principle. Enforcing it requires a dedicated class for the behavior:
{% highlight java %}
public class ShowLabelListener implements Button.ClickListener {

    private final Container container;
    
    public ShowLabelListener(Container container) {
    
        this.container = container;
    }

    @Override
    public void buttonClick(Button.ClickEvent event) {

        container.addComponent(new Label("Button clicked"));
    }
}
{% endhighlight %}
Not only is this snippet sound Object-Oriented design, it allows to pass parameters to the constructor and store them as attributes. Usage of this listener would be as the following:
{% highlight java %}
button.addClickListener(new ShowLabelListener(container));
{% endhighlight %}
Storing the container as an attribute in the listener is better than using lambdas directly, but introduces strong coupling from the behavior to the GUI in the form of an association. In this regard, I've already introduced Guava's [EventBus](http://code.google.com/p/guava-libraries/wiki/EventBusExplained) in a former [post](../another-way-decouple-your-server-components).

This would translate into the following code:
{% highlight java %}
public class ButtonClickedEvent {

    private ComponentContainer container;

    public ButtonClickedEvent(ComponentContainer container) {

        this.container = container;
    }

    public ComponentContainer getContainer() {

        return container;
    }
}

public class ShowClickMeListener {

    @Subscribe
    public void onClick(ButtonClickedEvent event) {

        event.getContainer().addComponent(new Label("Button clicked"));
    }
}
{% endhighlight %}
Usage then becomes:
{% highlight java %}
button.addClickListener(new Button.ClickListener() {

    @Override
    public void buttonClick(Button.ClickEvent event) {

        bus.post(new ButtonClickedEvent(container));
    }
});

eventBus.register(new ShowClickMeListener());
{% endhighlight %}
Of course, this is a little verbose, and that's when lambdas come in here handy. The previous snippet can be updated with Java 8's lambdas as:
{% highlight java %}
button.addClickListener(e -> bus.post(new ButtonClickedEvent(container)));

eventBus.register(new ShowClickMeListener());
{% endhighlight %}

IMHO, this is the right usage of lambdas in regard to Vaadin event listeners.

You can find relevant sources for this article on [Github](https://github.com/nfrankel/More-Vaadin/tree/master/lambda).