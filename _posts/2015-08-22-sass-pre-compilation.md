---
layout: post
title: "SASS pre-compilation"
tags: [vaadin 7, theming, client]
---
{% include JB/setup %}

Vaadin 7 comes complete with theming which in turn uses <abbr title="Cascading Style Sheet">CSS</abbr>. Though a back-end developer, I think I know CSS pretty well. Still, working with CSS, with Vaadin or not, is pretty boring and involves a lot of copy-paste. There are two technologies that aim to improve upon this: [LESS](http://lesscss.org/) and [SASS](http://sass-lang.com/) and Vaadin integrates the later.

Here's how it works: when the UI is configured with a theme, Vaadin will look for the `styles.css` file in the relevant theme folder. If not found, it will search for an alternate `styles.scss`. In this case, it will compile the SCSS using its internal compiler and make it available as a regular CSS. This behavior is exactly the same as with <abbr title="Java Server Pages">JSP</abbr> in application servers. This is extremely valuable during development as it prevents compile/deploy cycles and thus saves a lot of valuable time. However, it comes with two problems:

* The first hit to the themed UI will incur the compile cost
* If there's an issue with the compilation, it will break the application for all pages using the theme

As for JSP, it's advised to pre-compile SCSS for application packages that will be deployed to production environments. Vaadin provides the compiler class as a sort of public API, so it's quite easy. Basically, you just set the SCSS input and the CSS output and you're done.

If you use Maven, this is achieved by inserting the following snippet in your POM:

{% highlight xml %}
<plugin>
  <groupId>org.codehaus.mojo</groupId>
  <artifactId>exec-maven-plugin</artifactId>
  <version>1.2.1</version>
  <configuration>
    <mainClass>com.vaadin.sass.SassCompiler</mainClass>
    <arguments>
      <argument>
        ${project.basedir}/src/main/webapp/VAADIN/themes/mytheme/styles.scss
      </argument>
      <argument>${project.build.directory}/styles.css</argument>
    </arguments>
  </configuration>
  <executions>
    <execution>
      <goals>
        <goal>java</goal>
      </goals>
      <phase>process-resources</phase>
    </execution>
  </executions>
</plugin>
{% endhighlight %}