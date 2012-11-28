---
layout: post
title : Develop Vaadin apps with Scala
tags  : [vaadin 6, server]
---

More than a year ago, I tried to [use Scala to develop an example Vaadin application](http://blog.frankel.ch/mixing-vaadin-and-scala). Even though I'm no Scala guru (far from it), I must admit results were below what I expected them to be. Time has passed and Henri Kerola, a Vaadin team member, has commited to create a Scala add-on that aims to ease Scala Vaadin integration: the Scaladin add-on (formerly known as scala-wrappers) is more than meeting my expectations.

There are four brilliant ideas I want to focus on in this article:

+ The first idea is to provide a simple `Application` with a single main window with a content set during the `init()` method. As a developer, I just have to override the default content and I have everything I need to quickstart my development:
{% highlight scala linenos %}
import vaadin.scala.SimpleApplication
 
class VaadinScalaApp extends SimpleApplication {
 
  override def main = new CompositeFieldButton()
}
{% endhighlight %}
+ The second idea is to use Scala's named parameters to ease components configuration. In the Java API, components constructors are few and far between and generally are available in three forms: default, one with the label and the last with both label and value. If you do want no label and a value, you have to pass null as the first parameter.
In Scala, named parameters allow to write the following:
{% highlight scala linenos %}
new vaadin.scala.TextField(value = "world!")
{% endhighlight %}
+ The third idea is to use Scala's functional nature in place of anonymous inner classes for event management. In Java, event management is handled like so:
{% highlight java linenos %}
Button button = new Button("Click");
 
button.addListener(new Button.ClickListener() {
    public void buttonClick(ClickEvent event) {
        getWindow().showNotification("Click me!");
    }
});
{% endhighlight %}
It's not very concise and most of the code is only boilerplate. With Scaladin, leveraging Scala's functional nature and the former point, it can be replaced by:
{% highlight scala %}
new vaadin.scala.Button(action = _ => getWindow().showNotification("Click me!"))
{% endhighlight %}
+ The final idea is to provide a concise syntax for adding components. With the Java API, we have to instantiate a component in order to reference it, only can we add it to a container:
{% highlight java linenos %}
HorizontalLayout layout = new HorizontalLayout();
 
Button button = new Button("Click me!");
 
layout.add(button);
{% endhighlight %}
With Scala and Scaladin, adding a component to a container gets much easier:
{% highlight scala linenos %}
val layout = new HorizontalLayout() {
 
  val button = add(new Button(caption = "Click me!"))
}
{% endhighlight %}
Note: inner fields can be referenced by using the dotted notation (for example, `layout.button`).

With the help of all previous features, I redeveloped my first Scala example with only two classes, the application (which code can be found above) and the custom component:
{% highlight scala linenos %}
import com.vaadin.ui.CustomComponent
import vaadin.scala.{ HorizontalLayout, TextField, Button }
 
@SerialVersionUID(1L)
class CompositeFieldButton() extends CustomComponent {
 
  val layout = new HorizontalLayout(spacing = true, margin = true) {
 
    val button = add(new Button(caption = "Hello", action = _ => displayMessage()))
 
    val field = add(new TextField(value = "world!"))
  }
 
  setCompositionRoot(layout)
 
  def displayMessage(): Unit = {
 
    getWindow().showNotification(layout.button.getCaption() + " " + layout.field.getValue().asInstanceOf[String])
  }
}
{% endhighlight %}
In conclusion, where Scala and Vaadin don't play nice together, Scaladin let us really leverage Scala's power to ease our Vaadin development. If you're fan of Scala and Vaadin, you should probably run to get the Scaladin add-on.

Complete source for this article can be found [here](https://github.com/nfrankel/More-Vaadin/tree/master/vaadin-scala-example).
