---
layout: post
title: "Separation of concerns"
tags: [vaadin 6, vaadin 7, server]
---
{% include JB/setup %}

Last week, we quickly created a Delete [generated column]({% post_url 2012-02-12-table-generated-buttons %}) in a table.

Although this is enough to attain our objective, two nested anonymous classes puts a strain on maintenance costs. Moreover, behavior code (the deletion) is interwoven with GUI code (column).

Correcting these mistakes can be achieved through the creation of two top-level classes, one for the deletion behavior, and the other for the column. We have to find a way to pass the data container and the item id to the behavior class: the most obvious way to do so is to change the latter's structure to store both like so:
{% highlight java %}
public class DeleteButtonColumnGenerator implements ColumnGenerator {
 
    @Override
    public Object generateCell(Table source, Object itemId, Object columnId) {
 
        Button button = new Button("Delete");
 
        button.addListener(new DeleteClickListener(itemId, source.getContainerDataSource()));
 
        return button;
    }
}
{% endhighlight %}
This design is much more decoupled and maintenance friendly than the previous one, but we can do better. The click listener's structure is at present tightly coupled to the parameters it needs to access. It would be nice to remove this coupling. Fortunately, Vaadin components can store data on their own. Let's rely on this feature to clean our design.

First, create a placeholder for the container and the item id:
{% highlight java %}
public class ContainerItemId {
 
    private final Object itemId;
     
    private final Container container;
 
    public ContainerItemId(Container container, Object itemId) {
     
        this.itemId = itemId;
        this.container = container;
    }
 
    public Object getItemId() {
     
        return itemId;
    }
 
    public Container getContainer() {
     
        return container;
    }
}
{% endhighlight %}
Then, we remove all references to both container and item id from the click listener and use the Vaadin way:
{% highlight java %}
public class DeleteClickListener implements ClickListener {
 
    @Override
    public void buttonClick(ClickEvent event) {
 
        Button button = event.getButton();
         
        ContainerItemId cii = (ContainerItemId) button.getData();
         
        if (cii != null) {
             
            cii.getContainer().removeItem(cii.getItemId());
        }
    }
}
{% endhighlight %}
Notice how we get the reference to the button in the event, it's enough!

Finally, we have to pass the data to the button in the column generator:
{% highlight java %}
public class DeleteButtonColumnGenerator implements ColumnGenerator {
 
    @Override
    public Object generateCell(Table source, Object itemId, Object columnId) {
 
        Button button = new Button("Delete");
 
        ContainerItemId cii = new ContainerItemId(source.getContainerDataSource(), itemId);
         
        button.setData(cii);
         
        button.addListener(new DeleteClickListener());
 
        return button;
    }
}
{% endhighlight %}
This new design is much more modularized: for application that go beyond prototypes, this approach should be favored over the previous one.
