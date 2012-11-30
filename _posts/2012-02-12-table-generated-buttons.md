---
layout: post
title: "Table generated buttons"
tags: [vaadin 6, vaadin 7, server]
---
{% include JB/setup %}

Before diving in the middle of the subject, let's have a use-case first. Imagine we have a <abbr title="Create Read Update Delete">CRUD</abbr> application and our current screen lists the datastore entities in a table, a line per entity and a column per property.

Now, in order to implement the Delete functionality, we use the `addGeneratedColumn` method of the `Table` component to display a "Delete" button, like in the following screenshot:

<img src="/assets/images/delete_button.png" width="312" height="158" />

The problem lies in the action that has to take place when the button is pressed. The behavior needs to be determined when the page is generated, so that the event is processed directly on the server-side: in other words, we need a way to pass the entity identifier (or the container's item or even the container's item id) to the button somehow.

The solution is very simple to implement, with just the use of the `final` keyword, to let nested anonymous class methods access parameters:

{% highlight java %}
table.addGeneratedColumn("", new ColumnGenerator() {
 
  @Override public Object generateCell(final Table source, final Object itemId, Object columnId) {
 
    Button button = new Button("Delete");
 
    button.addListener(new ClickListener() {
 
      @Override public void buttonClick(ClickEvent event) {
 
        source.getContainerDataSource().removeItem(itemId);
      }
    });
 
    return button;
  }
});
{% endhighlight %}


