---
layout: post
title : Bean Validation and Vaadin comprehensive example
tags  : [vaadin 7, server]
---

One of Vaadin 7 key features is its tight JSR 303, also known as Bean Validation, integration. This article will detail on how to achieve such a validation.

For starters, let's have a simple use-case: the user fills in a form. When there's an error, the user is informed visually about the error. Submitting the form is only possible when all errors have been corrected. This a common scenario and we'll implement it with Vaadin.

Prerequisites
-------------
There are a couple of prerequisites to address before going further:

1. First, we'll need a JSR 303 implementation. In this case, we'll use Hibernate Validator (which is also JSR-303 Reference Implementation)
1. The next step is to define the underlying bean. In our case, we'll keep it simple (and stupid): a Person and its associated attributes. The first name and the last name must not be null while the email, when it exists, has to respect an email pattern. 
{% highlight java linenos %}
import java.util.Date;
 
import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;
 
import org.hibernate.validator.constraints.Email;
 
public class Person {
 
    private long id;
 
    private Gender gender;
 
    @Size(max = 250)
    @NotNull
    private String firstName;
 
    @Size(max = 250)
    @NotNull
    private String lastName;
 
    @Size(max = 250)
    @Email
    private String email;
 
    private Date birthdate;
 
    // Getters and setters
}
{% endhighlight %}
Note that `NotNull` and `Size` constraints come from the JSR itself (`javax.validation.constraints` package) while `Email` comes from Hibernate Validator (`org.hibernate.validator.constraints` package). Despite the coupling, considering the added value, I don't have any scruples using it.

Validating
----------
Once the bean is annotated, it's only a matter of:

+ Creating a new bean: this is pretty self-explanatory 
{% highlight java %}
BeanItem<Person> item = new BeanItem<Person>(new Person());
{% endhighlight %}
+ Vaadin 7 having deprecated the `Form` class, we need to put the bean into its replacement, `FieldGroup`. 
{% highlight java %}
FieldGroup group = new FieldGroup(item);
{% endhighlight %}
+ Then, we need to get a reference on each displayable field (as opposed to Form where there was a default and we could customize it). The `FieldGroup` has methods to create each field and to bind it to the underlying bean property. 
{% highlight java %}
Field<?> gender = group.buildAndBind("Gender", "gender", Select.class);
Field<?> firstName = group.buildAndBind("First name", "firstName");
Field<?> lastName = group.buildAndBind("Last name", "lastName");
Field<?> email = group.buildAndBind("email");
Field<?> birthdate = group.buildAndBind("Birth date", "birthdate");
{% endhighlight %}
+ Each field is then set its validator (if needed)
{% highlight java %}
firstName.addValidator(new BeanValidator(Person.class, "firstName"));
lastName.addValidator(new BeanValidator(Person.class, "lastName"));
email.addValidator(new BeanValidator(Person.class, "email"));
{% endhighlight %}
+ Finally, each field has to be set on our layout the way we want
{% highlight java %}
layout.addComponent(gender);
layout.addComponent(firstName);
layout.addComponent(lastName);
layout.addComponent(email);
layout.addComponent(birthdate);
{% endhighlight %}

Finishing touches
-----------------
There are a couple of finishing touches necessary to achieve a polished user interface.

### No preliminary validation

If all previous code snippets take place in the init() method, users will see validation errors on first and last names when the page is displayed, since all bean attributes are initially empty.

Thus, it's higly desirable to install validators only after the page has been displayed for the first time without errors. This could be done when the field loses focus and yet, it has to be done even if the field never receives focus. Thus, we have to set it in two places:
+ In each validated field blur listener
{% highlight java %}
firstName.addListener(new InstallPersonValidatorBlurListener(firstName, "firstName"));
lastName.addListener(new InstallPersonValidatorBlurListener(lastName, "lastName"));
email.addListener(new InstallPersonValidatorBlurListener(email, "email"));
 
public class InstallPersonValidatorBlurListener implements BlurListener {
 
    private Field<?> field;
    private String attribute;
 
    public InstallPersonValidatorBlurListener(Field<?> field, String attribute) {
 
        this.field = field;
        this.attribute = attribute;
    }
 
    @Override
    public void blur(BlurEvent event) {
 
        ValidatorUtils.installSingleValidator(field, attribute);
    }
}
{% endhighlight %}
+ Just before the `FieldGroup` commit
{% highlight java %}
ValidatorUtils.installSingleValidator(firstName, "firstName");
ValidatorUtils.installSingleValidator(lastName, "lastName");
ValidatorUtils.installSingleValidator(email, "email");
 
group.commit();
{% endhighlight %}
*Note that since we don't keep a reference to the installed validator, we have to uninstall it before installing it otherwise it leads to twice (or more) the same validation.*

For clarity's sake, here's the `ValidatorUtils` code:
{% highlight java %}
public class ValidatorUtils {
 
    private ValidatorUtils() {}
     
    static void installSingleValidator(Field<?> field, String attribute) {
         
        Collection<Validator> validators = field.getValidators();
 
        if (validators == null || validators.isEmpty()) {
 
            field.addValidator(new BeanValidator(Person.class, attribute));
        }
    }
}
{% endhighlight %}
### The right widget for the right data type

This point has nothing to do with validation, but it brings a nice finishing touch to the user interface. Did you notice that when we built the gender field, we passed a `Select` parameter and presto, the return field was a select. Wouldn't it be better if the birthdate field would be displayed as a true `DateField`? We can try this: 
{% highlight java %}
Field<?> birthdate = group.buildAndBind("Birth date", "birthdate", DateField.class);
{% endhighlight %}
Unfortunately, this doesn't work as Vaadin loudly complains with a `com.vaadin.data.fieldgroup.FieldGroup$BindException: Unable to build a field of type com.vaadin.ui.DateField for editing java.util.Date`. This means that in contrast to the old `Form`, `FieldGroup` doesn't handle date fields. As both use field factories (albeit of a different type), this is easily corrected. We just have to create a field group factory that return `DateField` when passed Date attributes and delegates to the default field group factory otherwise. 
{% highlight java %}
public class EnhancedFieldGroupFieldFactory implements FieldGroupFieldFactory {
 
    private FieldGroupFieldFactory fieldFactory = new DefaultFieldGroupFieldFactory();
 
    @SuppressWarnings({ "rawtypes", "unchecked" })
    @Override
    public <T extends Field> T createField(Class<?> dataType, Class<T> fieldType) {
 
        if (Date.class.isAssignableFrom(dataType)) {
 
            return (T) createDateField();
        }
 
        return fieldFactory.createField(dataType, fieldType);
    }
 
    @SuppressWarnings({ "rawtypes", "unchecked" })
    protected <T extends Field> T createDateField() {
 
        DateField field = new DateField();
 
        field.setImmediate(true);
 
        return (T) field;
    }
}
{% endhighlight %}
Then, we set an instance of this factory to the field group, passing the field type we need.
{% highlight java %}
Field<?> birthdate = group.buildAndBind("Birth date", "birthdate", DateField.class);
{% endhighlight %}

### Remove initial ugly null values

Finally, since the `Person` bean is initialized with `null` values, `null` appears in the fileds when the page first loads. For a end-user, this is not particularly desirable. We have to cast the field returned by the field group to a more friendly type and call the `setNullRepresentation()` method.
{% highlight java %}
AbstractTextField firstName = (AbstractTextField) group.buildAndBind("First name", "firstName");
 
firstName.setNullRepresentation("");
{% endhighlight %}
Conclusion
----------

In this article, we used the power of Vaadin 7 to easily create a submit form, complete with validation coming from JSR 303 annotations. The full sources of this article can be found on [Github](https://github.com/nfrankel/More-Vaadin/tree/master/beanvalidation-example).

