---
layout: post
title: "Vaadin 7 eases your theme development"
---
{% include JB/setup %}
Vaadin theming is a nice way to reuse your Cascading Style Sheets across different projects. Vaadin 7 adds even more sugar to the whole theming thing since it allows you to use [Syntactically Awesome Stylesheets](http://sass-lang.com/).

The purpose of this article is not to describe technology since it's already well documented on the [SASS website](http://sass-lang.com/) but to expose how to use SASS within Vaadin.

In order to use the SASS technology, a few steps are necessary:

+ First, the `UI` base class has to be annotated with the `@Theme` annotation referencing our own new theme.
{% highlight java linenos %}
@Theme("custom")
public class SassUI extends UI {
    ... 
}
{% endhighlight %}
+ Remember to set this UI as the Vaadin servlet parameter in the web deployment descriptor:
{% highlight xml linenos %}
<servlet>
    <servlet-name>VaadinServlet</servlet-name>
    <servlet-class>com.vaadin.server.VaadinServlet</servlet-class>
    <init-param>
        <param-name>UI</param-name>
        <param-value>com.morevaadin.vaadin7.sass.SassUI</param-value>
    </init-param>
</servlet>
{% endhighlight %}
+ Finally, instead of a CSS file, provide a styles.scss file under `/VAADIN/themes/<name>` (in your case, the name is "custom"). The code below is just an example:
{% highlight css linenos %}
@import "../reindeer/styles.css";
 
$color: rgb(51,204,255);
$dark-color: darken($color, 20%);
$spacing: 4px;
 
.v-button-caption {
    color: $color;
}
 
@mixin border-radius($radius) {
    -webkit-border-radius: $radius;
    -moz-border-radius: $radius;
    border-radius: $radius;
}
 
.rounded-borders {
    @include border-radius(4px);
    border: 1px solid $color;
}
 
.v-label {
    color: $dark-color;
    padding-left: $spacing;
    margin-top: $spacing;
    margin-bottom: $spacing;
    font-weight: bold;
}
{% endhighlight %}
That's it. Empowering your themes with SASS technology is as simple as that! Sources for this article can be found on [GitHub](https://github.com/nfrankel/More-Vaadin/tree/master/sass-example).

Note that some SASS features are still missing at the time of this writing.
