---
layout: post
title: "Dynamic Vaadin servlet registration"
tags: [vaadin 7, server, servlet 3.0]
---
{% include JB/setup %}

Servlet 3.0 is part of Java EE 6 and offers dynamic servlet registration. This feature let us automatically register servlets without the need of a web deployment descriptor in our webapp. Spring already provides this for its own DispatcherServlet with [`SpringServletContainerInitializer`](http://static.springsource.org/spring/docs/3.2.x/javadoc-api/org/springframework/web/SpringServletContainerInitializer.html).

This approach has for main advantage to use Java code and thus to compile-time type-checking. It would be nice to have the same one for Vaadin servlet. In order to achieve this, we need two classes, one entry-point and another for registration proper.

The registration class sole responsibility is to dynamically register the Vaadin servlet(s). We can design it however we choose, we need to access the servlet context for registration purpose. We also need to create programmatic setters to handle previously available servlet initialization parameters. Even better, we could choose to put in place sensible defaults. Here's such an example:

{% highlight java %}
public abstract class SingleRootVaadinRegistar {

    private static final String UI = "UI";

    @Override
    public void register(ServletContext context) {

        ServletRegistration.Dynamic servlet = context.addServlet(getServletName(), new VaadinServlet());

        if (getLoadOnStartup().isPresent()) {

            servlet.setLoadOnStartup(getLoadOnStartup().get().intValue());
        }

        for (String mapping : getMappings()) {

            servlet.addMapping(mapping);
        }

        Map<String, String> initParams = new HashMap<String, String>();

        if (getUi().isPresent()) {

            initParams.put(UI, getUi().get().toString());
        }

        if (getUiProvider().isPresent()) {

            initParams.put(SERVLET_PARAMETER_UI_PROVIDER, getUiProvider().get().toString());
        }

        if (isProductionMode().isPresent()) {

            initParams.put(SERVLET_PARAMETER_PRODUCTION_MODE, String.valueOf(isProductionMode().get().booleanValue()));
        }

        if (getWidgetSet().isPresent()) {

            initParams.put(PARAMETER_WIDGETSET, getWidgetSet().get());
        }

        if (isXsrfProtectionDisabled().isPresent()) {

            initParams.put(SERVLET_PARAMETER_DISABLE_XSRF_PROTECTION, String.valueOf(isXsrfProtectionDisabled().get().booleanValue()));
        }

        if (getCheckHeartBeatInterval().isPresent()) {

            initParams.put(SERVLET_PARAMETER_HEARTBEAT_INTERVAL, String.valueOf(getCheckHeartBeatInterval().get().intValue()));
        }

        if (isCloseIdleSessions().isPresent()) {

            initParams.put(SERVLET_PARAMETER_CLOSE_IDLE_SESSIONS, String.valueOf(isCloseIdleSessions().get().booleanValue()));
        }

        if (getResourcesCacheTime().isPresent()) {

            initParams.put(SERVLET_PARAMETER_RESOURCE_CACHE_TIME, String.valueOf(getResourcesCacheTime().get().intValue()));
        }

        servlet.setInitParameters(initParams);
    }

    abstract protected Optional<Class<? extends UI>> getUi();

    @Override
    protected String getServletName() {

        return "DynamicallyRegisteredVaadinServlet";
    }

    @Override
    protected String[] getMappings() {

        return new String[] { "/" };
    }

    @Override
    protected Optional<Boolean> isProductionMode() {

        return Optional.of(TRUE);
    }

    @Override
    protected Optional<Integer> getLoadOnStartup() {

        return absent();
    }

    @Override
    protected Optional<Class<? extends UIProvider>> getUiProvider() {

        return absent();
    }

    @Override
    protected Optional<Boolean> isProductionMode() {

        return absent();
    }

    @Override
    protected Optional<Boolean> isXsrfProtectionDisabled() {

        return absent();
    }

    @Override
    protected Optional<String> getWidgetSet() {

        return absent();
    }

    @Override
    protected Optional<Integer> getCheckHeartBeatInterval() {

        return absent();
    }

    @Override
    protected Optional<Boolean> isCloseIdleSessions() {

        return absent();
    }

    @Override
    protected Optional<Integer> getResourcesCacheTime() {

        return absent();
    }        
}
{% endhighlight %}

Note the only method to implement is `getUI()` since it has to be customized for each web application. All methods provide sensible defaults but can be overriden should it be needed.

On the other hand, the entry-point implementation should do something like this: for each type referenced, instantiate it and calls it `register()` method. This leads to two requirements:

* Implement [`javax.servlet.ServletContainerInitializer`](http://docs.oracle.com/javaee/6/api/javax/servlet/ServletContainerInitializer.html) more specifically the single `onStartup()` method
* Be annotated with [`@javax.servlet.annotation.HandlesTypes`](http://docs.oracle.com/javaee/6/api/javax/servlet/annotation/HandlesTypes.html) to reference relevant types

{% highlight java %}
@HandlesTypes(SingleRootVaadinRegistar.class)
public class DynaVaadinInitializer implements ServletContainerInitializer {

    @Override
    public void onStartup(Set<Class<?>> classes, ServletContext servletContext) throws ServletException {

        if (classes != null) {

            for (Class<?> clazz : classes) {

                if (!clazz.isInterface() && !Modifier.isAbstract(clazz.getModifiers()) &&
                        VaadinServletRegistar.class.isAssignableFrom(clazz)) {

                    try {

                        VaadinServletRegistar initializer = (VaadinServletRegistar) clazz.newInstance();

                        initializer.register(servletContext);

                    } catch (Throwable e) {

                        throw new ServletException("Failed to instantiate VaadinServletRegistar class", e);
                    }
                }
            }
        }
    }
}

{% endhighlight %}

The final step is to implement a service provider: create a `javax.servlet.ServletContainerInitializer` file under `META-INF/services` and set its content to the fully-qualified name of `SingleRootVaadinRegistar`.

Compiling and packaging the above classes and resources in a JAR put in your `WEB-INF/lib` folder and extending `SingleRootVaadinRegistar` will achieve our goal. The extended class will register a Vaadin servlet, with the UI we returned.

Of course, coding those for every project is boring: it would be better to have a reusable component. There's a shot at this on [Vaadin directory]().




