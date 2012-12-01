---
layout: post
title : Spring security integration
tags  : [server, security]
---

Along with [Apache Shiro](https://shiro.apache.org/), [Spring Security](http://static.springsource.org/spring-security/site/) is one of the two most used security component used in the Java world. Using Spring Security with Vaadin needs a little work. In this article, I will show you how you can adapt your Vaadin application to play nice with Spring Security.

Credits go to Henri Sara of the Vaadin team who provided [his valuable insight](https://vaadin.com/forum/-/message_boards/view_message/569686#_19_message_373038).
Requirements: this article assumes you know some Spring Security and uses advanced Vaadin navigator concepts

Nominal uses of Spring Security mandate for the use of subcontexts in the webapp, each one then can be configured for different access levels. For example, `/public` is accessible with anonymous access, while `/private` needs some specific authorizations. Unfortunately, Vaadin doesn't work that way: [views]({% post_url 2012-07-07-navigation-basics %}) are translated into fragments, not subcontexts. From this point on, there are two options: either tweak Vaadin to use subcontexts, or embed Spring Security inside our application. We will use the latter. 

+ The first step is to create a Login form view, that send login events.
+ The root subscribes to login events, and handles authentication attemps through a dedicated authentication handler.
+ The handler has to be passed the login, the password and the http request. Since Vaadin hides the latter in the API, we have to create a special servlet that stores it in a thread local with the help of an utility class:
{% highlight java linenos %}
public class RequestHolderApplicationServlet extends ApplicationServlet {
 
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 
        RequestHolder.setRequest(request);
 
        super.service(request, response);
 
        // We remove the request from the thread local, there's no reason to keep it once the work is done
        RequestHolder.clean();
    }
}
{% endhighlight %}
+ The login handler uses the Spring Security API to create the username/password token needed by the framework. Then, it gets the authentication manager from the Spring context and calls the relevant method, delegating real authentication to the configured backend. Last but not least, we set authentication data into the Spring Security context for latter uses. 
{% highlight java linenos %}
public class AuthenticationService {
 
    public void handleAuthentication(String login, String password, HttpServletRequest httpRequest) {
         
        UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(login, password);
 
        token.setDetails(new WebAuthenticationDetails(httpRequest));
 
        ServletContext servletContext = httpRequest.getSession().getServletContext();
 
        WebApplicationContext wac = WebApplicationContextUtils.getRequiredWebApplicationContext(servletContext);
 
        AuthenticationManager authManager = wac.getBean(AuthenticationManager.class);
 
        Authentication authentication = authManager.authenticate(token);
 
        SecurityContextHolder.getContext().setAuthentication(authentication);
    }
}
{% endhighlight %}
+ Don't forget to also create and clean the Spring Security context. Since we already developed a custom servlet, we'll update it.
{% highlight java linenos %}
public class RequestHolderApplicationServlet extends ApplicationServlet {
 
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
 
        SecurityContextHolder.setContext(SecurityContextHolder.createEmptyContext());
 
        RequestHolder.setRequest(request);
 
        super.service(request, response);
 
        RequestHolder.clean();
 
        SecurityContextHolder.clearContext();
    }
}
{% endhighlight %}
Now, the root just has to try/catch the authentication results and navigate to the desired view if succesfull. Job done!

Wait, what if the user knows about the view's name and types the fragment directly in his/her browser's URL bar? No login event, no authentication: he/she will have access directly to the main view, regardless of his/her credentials.

So, there's one last step to implement: we have to bind all our views into a navigator and let the latter handle navigation. Besides, we register it a `ViewChangeListener`l that will check for credentials before changing the view.
{% highlight java linenos %}
public class ViewChangeSecurityChecker implements ViewChangeListener {
 
    @Override
    public boolean isViewChangeAllowed(ViewChangeEvent event) {
 
        if (event.getNewView() instanceof LoginView) {
 
            return true;
        }
 
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
 
        return authentication == null ? false : authentication.isAuthenticated();
    }
 
    @Override
    public void navigatorViewChanged(ViewChangeEvent event) {}
}
{% endhighlight %}
Now, when an unauthenticated user directly types the view name as a fragment, there won't be any action. Notice that the login view has to accessible in any case.

Note that the above code only checks for authenticated status, not for credential. You can easily enhance it to do just that by forking the [GitHub repo](https://github.com/nfrankel/More-Vaadin/tree/master/springsecurity-integration). 
