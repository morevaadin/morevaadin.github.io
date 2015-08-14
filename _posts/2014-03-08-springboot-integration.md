---
layout: post
title: "Spring Boot and JavaConfig integration"
tags: [vaadin 7, server, spring, servlet 3.0]
---
{% include JB/setup %}

Java EE in general and Context and Dependency Injection has been part of the Vaadin ecosystem since ages. Recently, [Spring Vaadin](https://github.com/peholmst/vaadin4spring) is a joint effort of the Vaadin and the Spring teams to bring the Spring framework into the Vaadin ecosystem, lead by [Petter Holmström](https://twitter.com/petterholmstrom) for Vaadin and [Josh Long](https://twitter.com/starbuxman) for Pivotal.

Integration is based on the [Spring Boot](http://projects.spring.io/spring-boot/) project - and its sub-modules, that aims to ease creating new Spring web projects. This article assumes the reader is familiar enough with Spring Boot. If not the case, please take some time to get to understand basic notions about the library.

Note that at the time of this writing, there's no release for Spring Vaadin. You'll need to clone the project and build it yourself.

The first step is to create the UI. In order to display usage of Spring's Dependency Injection, it should use a service dependency. Let's injection the UI through Constructor Injection to favor immutability. The only addition to a *standard* UI is to annotate it with `org.vaadin.spring.@VaadinUI`.

{% highlight java %}
@VaadinUI
public class VaadinSpringExampleUi extends UI {

    private HelloService helloService;

    public VaadinSpringExampleUi(HelloService helloService) {

        this.helloService = helloService;
    }

    @Override
    protected void init(VaadinRequest vaadinRequest) {

        String hello = helloService.sayHello();

        setContent(new Label(hello));
    }
}
{% endhighlight %}

The second step is standard Spring Java configuration. Let's create two configuration classes, one for the main context and the other for the web one. Two thing of note:

1. The method instantiating the previous UI has to be annotated with `org.vaadin.spring.@UIScope` **in addition** to standard Spring `org.springframework.context.annotation.@Bean` to bind the bean lifecycle to the new scope provided by the Spring Vaadin library.
2. At the time of this writing, a `RequestContextListener` bean must be provided. In order to be compliant with future versions of the library, it's a good practice to annotate the instantiating method with `@ConditionalOnMissingBean(RequestContextListener.class)`.

{% highlight java %}
@Configuration
public class MainConfig {

    @Bean
    public HelloService helloService() {

        return new HelloService();
    }
}

@Configuration
public class WebConfig extends MainConfig {

    @Bean
    @ConditionalOnMissingBean(RequestContextListener.class)
    public RequestContextListener requestContextListener() {

        return new RequestContextListener();
    }

    @Bean
    @UIScope
    public VaadinSpringExampleUi exampleUi() {

        return new VaadinSpringExampleUi(helloService());
    }
}
{% endhighlight %}

The final step is to create a dedicated `WebApplicationInitializer`. Spring Boot already offers a concrete implementation, we just need to reference our previous configuration classes as well as those provided by Spring Vaadin, namely `VaadinAutoConfiguration` and `VaadinConfiguration`.

{% highlight java %}
public class ApplicationInitializer extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {

        return application.showBanner(false)
                .sources(MainConfig.class)
                .sources(VaadinAutoConfiguration.class, VaadinConfiguration.class)
                .sources(WebConfig.class);
    }
}
{% endhighlight %}

At this point, we demonstrated a working Spring Vaadin sample application.

Code for this article can be browsed and forked on [Github](https://github.com/nfrankel/More-Vaadin/tree/master/springboot-example).








