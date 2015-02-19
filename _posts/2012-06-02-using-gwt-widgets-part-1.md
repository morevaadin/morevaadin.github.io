---
layout: post
title : Using GWT widgets - Part 1
tags  : [vaadin 7, client, gwt]
---

Even if Vaadin provides you with plenty of components out-of-the-box, chances are sooner or later, you'll want to use that special GWT widget you just saw the last day. In that case, be happy because Vaadin let you do that. Then, you'll be able to package the component in a simple JAR archive and reuse it in all projects you want, just like Vaadin components themselves. In this 3 parts article serie, we'll see how to do just that with Vaadin 7:

+ This first part is about just wrapping GWT widgets in Vaadin components
+ In the second part, we'll see how to configure the client widget from the server component
+ The final part will detail how to communicate from client to server 

The first step it to choose which widget to wrap. As an illustration, we'll use GWT-Bootstrap, a GWT implementation of Twitter Bootstrap and most specifically the `com.github.gwtbootstrap.client.ui.Button`. Of course, lessons learned here can easily be transposed for other GWT widgets.

Wrapping a GWT widget in Vaadin code requires a couple of classes:

+ The first class to create is the Vaadin component itself. Such components should extends `com.vaadin.ui.AbstractComponent` (or `com.vaadin.ui.AbstractComponentContainer` for components that contains other components, which is not the case for buttons):
{% highlight java linenos %}
public class BootstrapButton extends AbstractComponent {
 
    ...
}
{% endhighlight %}
+ The second class to create is the client widget itself. It's very simple: just extend the wanted GWT widget (the Bootstrap button in our case). Note that **it's mandatory to locate it in a `client.ui` subpackage** relative to the above component:
{% highlight java linenos %}
public class VBootstrapButton extends Button {
 
    public VBootstrapButton() {
 
        addStyleName("v-button-bootstrap");
    }
}
{% endhighlight %}
+ The final class will bind the component and the widget. In Vaadin semantics, it's called a connector. Connectors have to extend `com.vaadin.terminal.gwt.client.ui.AbstractComponentConnector` (or `com.vaadin.terminal.gwt.client.ui.AbstractComponentContainerConnector` for those widgets that contain other widgets). Connecting is achieved by implement the `createWidget()` that should return the previously implemented client class and by annotating the connector with the `@Connect` that takes the server class as the value. 
{% highlight java linenos %}
@Connect(com.morevaadin.vaadin7.custom.BootstrapButton.class)
public class BootstrapButtonConnector extends AbstractComponentConnector {
 
    @Override
    protected Widget createWidget() {
 
        return GWT.create(VBootstrapButton.class);
    }
}
{% endhighlight %}
It also has to be located in the `client.ui` subpackage. 

GWT reads Java code and renders HTML and JavaScript. When using standard Vaadin components, the code is precompiled and available in the Vaadin JAR, but when adding third-party widgets, we need to compile both Vaadin and the other widgets. In order to achieve this, two things are necessary:

+ First, we have to create a GWT widgetset `gwt.xml` file, referencing both the Vaadin and the Bootstrap widgetset: 
{% highlight xml linenos %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE module PUBLIC "-//Google Inc.//DTD Google Web Toolkit 1.7.0//EN" "http://google-web-toolkit.googlecode.com/svn/tags/1.7.0/distro-source/core/src/gwt-module.dtd">
<module>
    <inherits name="com.vaadin.terminal.gwt.DefaultWidgetSet" />
    <inherits name="com.github.gwtbootstrap.Bootstrap" />
    <!-- Reduces compilation time in development mode -->
    <!--
        <set-property name="user.agent" value="safari,gecko1_8" />
    -->
</module>
{% endhighlight %}
+ Then, we have to handle the compilation itself. Either uses the Eclipse Vaadin plugin compilation feature or insert the following snippet in the Maven POM:
{% highlight xml linenos %}
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>gwt-maven-plugin</artifactId>
            <version>2.4.0</version>
            <configuration>
                <webappDirectory>${project.build.directory}/${project.build.finalName}/VAADIN/widgetsets</webappDirectory>
                <extraJvmArgs>-Xmx512M -Xss1024k</extraJvmArgs>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>resources</goal>
                        <goal>compile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
        <plugin>
            <groupId>com.vaadin</groupId>
            <artifactId>vaadin-maven-plugin</artifactId>
            <version>1.0.2</version>
            <executions>
                <execution>
                    <configuration>
                    </configuration>
                    <goals>
                        <goal>update-widgetset</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
{% endhighlight %}
Sources for this article can be found on [GitHub](https://github.com/nfrankel/More-Vaadin/tree/master/custom-component-example). In the [following part]({% post_url 2012-06-09-using-gwt-widgets-part-2 %}), we'll detail how to let developers customize widgets though components on the server side.
