---
layout: post
title : Testing with Embed for Vaadin
tags  : [vaadin 6, vaadin 7, server]
---

In order to execute integration tests of a web application, there are not so many tools available. One of such tools is [JBoss Arquilian](https://www.jboss.org/arquillian.html): it let you create an archive of your to-be-tested Java classes. [Embed Vaadin](https://vaadin.com/directory#addon/embed-for-vaadin) does the same, but is specifically targeted at Vaadin applications and components!

Just select the component you want to test, and presto, it's wrapped in a dummy application and launched in an embedded Tomcat:
{% highlight java %}
EmbedVaadin.forComponent(new FormAdvancedLayoutExample()).wait(false).start();
{% endhighlight %}
If you need to go beyond that and test the full application, that's also possible. The library takes care of instantiating the application class and all the underlying gory details.
{% highlight java %}
EmbedVaadin.forApplication(EmbedApplication.class).wait(false).start();
{% endhighlight %}
Of course, there are configuration parameters. For example, you can choose the HTTP port (in order to avoid port conflict) and to launch a browser window. Presto, your integration test is ready to run!
{% highlight java %}
EmbedVaadin.forApplication(EmbedApplication.class).withHttpPort(8888).wait(true).openBrowser(true).start();
{% endhighlight %}
For Maven users, the good news is that the add-on is readily provided as a Maven dependency:
{% highlight xml %}
<dependency>
    <groupId>com.bsb.common.vaadin</groupId>
    <artifactId>com.bsb.common.vaadin.embed</artifactId>
    <version>0.5</version>
    <scope>test</scope>
</dependency>
{% endhighlight %}
For Vaadin 7, please use:
{% highlight xml %}
<dependency>
    <groupId>com.bsb.common.vaadin</groupId>
    <artifactId>com.bsb.common.vaadin7.embed</artifactId>
    <version>0.5</version>
    <scope>test</scope>
</dependency>
{% endhighlight %}
For those wishing to go further, a full-fledged example is available on [Github](https://github.com/nfrankel/More-Vaadin/tree/master/embed-example).

*Note that Embed Vaadin is available under the friendly Apache 2.0 license.*
{% highlight java linenos %}
{% endhighlight %}
