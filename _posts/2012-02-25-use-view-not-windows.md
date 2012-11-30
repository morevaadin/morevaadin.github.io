---
layout: post
title: "Use view, not windows"
tags: [vaadin 6, server]
---
{% include JB/setup %}

In Vaadin 6, the `Window` class is used for both main windows - i.e. windows that fill the entire screen, and popup windows aka subwindows.

This has lead some developers (including me) to think main windows could be set and then removed later on: this is not the case and may lead to nasty bugs when the page is refreshed by the user just after having switched main windows.

*Note: Vaadin 7 tackles the problem by merging the application and the main window into a single class (more in a [later article]({% post_url 2012-03-02-windows-switching %}).*

In essence, when one has to fundamentally change what is shown to the user, one should use a custom component I call view. A view is just a group of components that are laid out together. Then, when we need to switch windows, we keep the main window and switch its content from one view to another: the main window becomes a just a placeholder. As an added value, since the main window is kept, it has always access to the parent application.

The application becomes something like this:
{% highlight java %}
public class MyApplication extends Application {
 
  @Override
  public void init() {
 
    setMainWindow(new Window());
  }
}
{% endhighlight %}
The view should look something like that:
{% highlight java %}
public class LoginView extends CustomComponent {
 
  private TextField login = new TextField("Login");
 
  private TextField password = new TextField("Password");
 
  public LoginView() {
 
    FormLayout layout = new FormLayout();
 
    setCompositionRoot(layout);
 
    layout.addComponent(login);
    layout.addComponent(password);
 
    Button button = new Button("Login");
 
    layout.addComponent(button);
 
    button.addListener(new ClickListener() {
 
      @Override
      public void buttonClick(ClickEvent event) {
 
        getApplication().getMainWindow().setContent(new MainView());
      }
    });
  }
}
{% endhighlight %}
Important notes:

1. `MainView` is another custom component with the components we want displayed
1. Don't forget [separation of concerns]({% 2012-02-18-separation-of-concerns %}), the switching behavior mixed in the GUI code is shown only for readability purposes
1. Login and logout should perhaps be designed at the application level so we could call `getApplication().login()`
