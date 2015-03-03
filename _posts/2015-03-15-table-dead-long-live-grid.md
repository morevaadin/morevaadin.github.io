---
layout: post
title: "The Table is dead, long live the Grid"
tags: [vaadin 7, vaadin 7.4, grid, server]
---
{% include JB/setup %}

Vaadin 7.4.0 is finally out and with it comes the long awaited successor of the [Table](http://vaadin.com/download/release/7.4/7.4.0/docs/api/com/vaadin/ui/Table.html) component, the [Grid](http://vaadin.com/download/release/7.4/7.4.0/docs/api/com/vaadin/ui/Grid.html). As for the rest of Vaadin, it's quite [well-documented](https://vaadin.com/book/-/page/components.grid.html).

This article aims at showing a simple example of this new component to highlights differences with the previous Table. In order to do so, let's use an existing sample - namely my [Vaadin workshop](https://github.com/nfrankel/vaadin7-workshop/blob/vaadin7.2/src/main/java/ch/frankel/vaadin/workshop/ui/MessageTable.java) that displays a table of messages, with date, author, text and a delete button.

<div class="imagecenter">
  <img src="/assets/images/table.png" width="610" height="296" />
</div>

First steps with Grid
-------------

A first draft implementation would look like this:

```java
public class SampleGrid extends Grid {

    public SampleGrid(Container.Indexed indexed) {
        setContainerDataSource(indexed);
    }
}
```

Grid's columns
-------------

It's simple and quite straightforward. At this point, nothing distinguishes the new `Grid` from the old `Table`, only default sortable columns. As most chat applications don't allow that - and the one from the workshop neither, let's disable it:

```java
public class SampleGrid extends Grid {

    public SampleGrid(Container.Indexed indexed) {
        setContainerDataSource(indexed);
        getColumns().stream().forEach(c -> c.setSortable(false));
    }
}
```

So the `Grid` component has the concept of columns. One can get a reference on a single column column by passing the `propertyId` of the underlying item with `getColumn(propertyId)` or the sequence of all columns with `getColumns()`. Note that Java 8 helps with the handling of each column with the Stream API and a simple lambda's usage.

Wrapped container
-------------

The next step is to hide the message's id column; with `Table`, it was achieved by calling `setVisibleColumns()` and omitting the `id` property in the parameter array. With Vaadin 7.4, the `Container` hierarchy has been enriched with the Delegate pattern: the `GeneratedPropertyContainer` is a wrapper around another container but offers additional methods, including `removeContainerProperty(propertyId)` that effectively hides a property from the wrapped container. This produces the following code:

```java
public class SampleGrid extends Grid {

    public SampleGrid(Container.Indexed indexed) {
        GeneratedPropertyContainer wrapperContainer = new GeneratedPropertyContainer(indexed);
        wrapperContainer.removeContainerProperty("id");
        setContainerDataSource(wrapperContainer);
        getColumns().stream().forEach(c -> c.setSortable(false));
    }
}
```

Renderers
-------------

Now, let's format the date column. With `Table`, it required implementing a dedicated `ColumnGenerator`. At first glance, it seems using the `addGeneratedProperty` of the wrapped container mentioned in the above section would be the thing to do. However, for simple formatting issues, Vaadin 7.4 provides a `Renderer` interface: any column can be set such a renderer. Icing on the cake, a `DateRenderer` is provided that accepts a format string as an argument.

```java
public class SampleGrid extends Grid {

    private static final String FORMAT = "%1$td/%1$tm/%1$tY %1$tH:%1$tM:%1$tS";

    public SampleGrid(Container.Indexed indexed) {
        GeneratedPropertyContainer wrapperContainer = new GeneratedPropertyContainer(indexed);
        wrapperContainer.removeContainerProperty("id");
        setContainerDataSource(wrapperContainer);
        getColumn("timeStamp").setRenderer(new DateRenderer(FORMAT));
        getColumns().stream().forEach(c -> c.setSortable(false));
    }
}
```

Note the format string uses the pattern from [`Formatter`](http://docs.oracle.com/javase/7/docs/api/java/util/Formatter.html), not from [`SimpleDateFormat`](http://docs.oracle.com/javase/7/docs/api/java/text/SimpleDateFormat.html).

Multi-selection
-------------

The last task is to add the delete feature. Out-of-the-box, setting the selection mode to multi create a new column with checkboxes to select multiple lines:

```java
public class SampleGrid extends Grid {

    private static final String FORMAT = "%1$td/%1$tm/%1$tY %1$tH:%1$tM:%1$tS";

    public SampleGrid(Container.Indexed indexed) {
        GeneratedPropertyContainer wrapperContainer = new GeneratedPropertyContainer(indexed);
        wrapperContainer.removeContainerProperty("id");
        setContainerDataSource(wrapperContainer);
        getColumn("timeStamp").setRenderer(new DateRenderer(FORMAT));
        getColumns().stream().forEach(c -> c.setSortable(false));
        setSelectionMode(SelectionMode.MULTI);
    }
}
```

Then, just create a simple button that will delete all selected lines. This gets the job done. However, this is not the original design: in the above screenshot, each line provides its own dedicated button. Pro, you have to click on the selected line so you prevent most mistakes; con, you cannot batch delete.

Along with the date renderer, Vaadin 7.4 also provides a `ButtonRenderer` that accepts a `RenderClickListener` to add behavior. This only needs to be set on the `id` column.

```java
public class SampleGrid extends Grid {

    private static final String FORMAT = "%1$td/%1$tm/%1$tY %1$tH:%1$tM:%1$tS";

    public SampleGrid(Container.Indexed indexed) {
        GeneratedPropertyContainer wrapperContainer = new GeneratedPropertyContainer(indexed);
        setContainerDataSource(wrapperContainer);
        getColumn("timeStamp").setRenderer(new DateRenderer(FORMAT));
        getColumn("id").setRenderer(new ButtonRenderer(event -> {
            Object itemId = event.getItemId();
            indexed.removeItem(itemId);
        }));
        getColumns().stream().forEach(c -> c.setSortable(false));
        setSelectionMode(SelectionMode.MULTI);
    }
}
```

The downside of this approach is that you cannot change the button's label as it is taken from the underlying object with no way of changing it. In this case, the message's id will be displayed in a button and users might cluelessly delete the message. That's out of the question.

This makes it only slightly more complex as we need to add a dedicated container property that always returns the wanted button label and assign it the renderer from above. The final code looks like the following:

```java
public class SampleGrid extends Grid {

    private static final String FORMAT = "%1$td/%1$tm/%1$tY %1$tH:%1$tM:%1$tS";

    public SampleGrid(Container.Indexed indexed) {
        GeneratedPropertyContainer wrapperContainer = new GeneratedPropertyContainer(indexed);
        wrapperContainer.removeContainerProperty("id");
        setContainerDataSource(wrapperContainer);
        wrapperContainer.addGeneratedProperty("delete", new PropertyValueGenerator<String>() {
            @Override
            public String getValue(Item item, Object itemId, Object propertyId) {
                return "Delete";
            }

            @Override
            public Class<String> getType() {
                return String.class;
            }
        });
        getColumn("delete").setRenderer(new ButtonRenderer(event -> {
            Object itemId = event.getItemId();
            indexed.removeItem(itemId);
        }));
        getColumn("timeStamp").setRenderer(new DateRenderer(FORMAT));
        getColumns().stream().forEach(c -> c.setSortable(false));
    }
}
```

A new generated property is added with the `PropertyValueGenerator` always returning the string `Delete`. More advanced requirements could use a `Locale` to have a display depending on the user preferences, but this code is enough for ours. Note the button renderer is kept but assigned to the new `delete` generated column instead of the id's. Finally, the multiselection mode has to be removed.

## Header

The final touch is to remove the column headers as befits any chat application. This is simply done with `setHeaderVisible(false)`.

The final result looks like the original table, while saving some lines of code.

<div class="imagecenter">
<img src="/assets/images/grid.png" width="643" height="217" />
</div>

The code for this article can be browsed and forked on [Github](https://github.com/nfrankel/More-Vaadin/tree/master/grid-example).
