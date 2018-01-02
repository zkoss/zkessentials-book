# MVVM Approach

Building a user interface for the example application using the MVVM
approach is very similar to building it using the MVC approach, however,
you don't need to give an `id` to components since we don't need to
identify components for wiring. For defining the ViewModel's properties,
we should analyse what data required to display on the user interface or
to be kept as a View's state. There are 4 types of data, todo item's
subject for creating a new todo item, todo item list for displaying all
todos, selected todo item for keeping user selection, and todo's
priority list for *Radiogroup* in detail editor.

```java
public class TodoListViewModel implements Serializable{


    //data for the view
    String subject;
    ListModelList<Todo> todoListModel;
    Todo selectedTodo;

    public List<Priority> getPriorityList(){
        return Arrays.asList(Priority.values());
    }

    //omit property accessor methods and others
...
}
```

-   A property is retrieved by a getter, so ViewModel doesn't have to
    declare a variable for a property.

## Read

As we discussed in previous chapter, displaying a collection of data
requires to prepare a data model in the ViewModel.

```java
public class TodoListViewModel implements Serializable{


    //services
    TodoListService todoListService = new TodoListServiceChapter4Impl();

    //data for the view
    String subject;
    ListModelList<Todo> todoListModel;
    Todo selectedTodo;

    @Init // @Init annotates a initial method
    public void init(){
        //get data from service and wrap it to model for the view
        List<Todo> todoList = todoListService.getTodoList();
        //you can use List directly, however use ListModelList provide efficient control in MVVM
        todoListModel = new ListModelList<Todo>(todoList);
    }

...
}
```

-   Line 17: Initialize `ListModelList` with `todoList` retrieved with a
    service class.

Then we can bind Listbox's `model` to prepared data model of the
ViewModel with data binding expression.

```xml

    <listbox model="@bind(vm.todoListModel)"
        selectedItem="@bind(vm.selectedTodo)" vflex="1" >
        <listhead>
            <listheader width="30px" />
            <listheader/>
            <listheader hflex="min"/>
        </listhead>
        <template name="model">
            <listitem sclass="@bind(each.complete?'complete-todo':'')">
                <listcell>
                    <checkbox checked="@bind(each.complete)"
                        onCheck="@command('completeTodo',todo=each)"/>
                </listcell>
                <listcell>
                    <label value="@bind(each.subject)"/>
                </listcell>
                <listcell>
                    <button onClick="@command('deleteTodo',todo=each)"
                        image="/imgs/cross.png" width="36px"/>
                </listcell>
            </listitem>
        </template>
    </listbox>
```

-   Line 1, 2: Set *Listbox*'s data model by binding `model` attribute
    to a property of type `ListModelList`. Binding `selecteditem` to
    `vm.selectedTodo` to keep user selection in the ViewModel.
-   Line 9: You can fill any valid EL expression In a data binding
    annotation, so that you can implement simple presentation logic with
    EL. Here we set `sclass` according to a `Todo` object's `complete`
    property.
-   Line 11, 12, 15: Use implicit variable `each` to access each `Todo`
    object in the data model.

## Create


We can create a new todo item by either clicking the button with plus
icon (![](Tutorial-ch6-plus.png "fig:Tutorial-ch6-plus.png")) or
pressing the "Enter" key, therefore we can bind these two events to the
same command method that adds a todo item.

**Command method `addTodo`**

```java
    @Command //@Command annotates a command method
    @NotifyChange({"selectedTodo","subject"}) //@NotifyChange annotates data changed notification after calling this method
    public void addTodo(){
        if(Strings.isBlank(subject)){
            Clients.showNotification("Subject is blank, nothing to do ?");
        }else{
            //save data
            selectedTodo = todoListService.saveTodo(new Todo(subject));
            //update the model, by using ListModelList, you don't need to notify todoListModel change
            //it is efficient that only update one item of the listbox
            todoListModel.add(selectedTodo);
            todoListModel.addToSelection(selectedTodo);

            //reset value for fast typing.
            subject = null;
        }
    }
```

-   Line 2: You can notify multiple properties change by filling an
    array of String. Here we specify `{"selectedTodo","subject"}`, since
    we change them in the method.

We can see that the benefits of abstraction provided by command binding
allows developers to bind different events to the same command without
affecting the ViewModel.

**Binding to `addTodo`**

```xml

                <hbox align="center" hflex="1" sclass="todo-box">
                    <textbox value="@bind(vm.subject)"
                        onOK="@command('addTodo')"
                            hflex="1" placeholder="What needs to be done?"/>
                    <button onClick="@command('addTodo')"
                        image="/imgs/plus.png" width="36px"/>
                </hbox>
```

-   Line 2\~6: The `onOK` and `onClick` can invoke the same command
    method.


# Update

How do we achieve the feature that selecting a todo item then detail
editor becomes visible under MVVM? Simple, just determine editor's
visibility upon selected todo item is null or not.

```xml
        <east visible="@bind(not empty vm.selectedTodo)" width="300px"
        border="none" collapsible="false" splittable="true"
        minsize="300" autoscroll="true">
        <!-- todo item detail editor-->
        </east>
```

-   Line 1: ZK bind monitors all binding properties. If one property
    changes, ZK bind re-evaluates those binding expressions that bind to
    the changed property.

In order to make selected todo item's properties display in the detail
editor, we just bind input components in the detail editor to the
corresponding `selectedTodo`'s properties.

**Binding input components to selected item's properties**

```xml

    <vlayout
    form="@id('fx') @load(vm.selectedTodo)
        @save(vm.selectedTodo, before='updateTodo')">
        <hbox align="center"  hflex="1">
            <checkbox checked="@bind(fx.complete)"/>
            <textbox value="@bind(fx.subject)" hflex="1" />
        </hbox>
        <grid hflex="1">
            <columns>
                <column align="right" hflex="min"/>
                <column/>
            </columns>
            <rows>
                <row>
                    <cell sclass="row-title">Priority :</cell>
                    <cell>
                        <radiogroup model="@bind(vm.priorityList)"
                            selectedItem="@bind(fx.priority)">
                            <template name="model">
                                <radio label="@bind(each.label)"/>
                            </template>
                        </radiogroup>
                    </cell>
                </row>
                <row>
                    <cell sclass="row-title">Date :</cell>
                    <cell><datebox value="@bind(fx.date)" width="200px"/>
                    </cell>
                </row>
                <row>
                    <cell sclass="row-title">Description :</cell>
                    <cell>
                        <textbox value="@bind(fx.description)" multiline="true"
                            hflex="1" height="200px" />
                    </cell>
                </row>
            </rows>
        </grid>
        <hlayout>
            <button onClick="@command('updateTodo')" label="Update"/>
            <button onClick="@command('reloadTodo')" label="Reload"/>
        </hlayout>
    </vlayout>
```

-   Line 2,3: Here we create a form binding at `form` attribute and give
    the middle object's id fx. Specify `@load(vm.selectedTodo)` makes
    the binder load selected todo's properties to the middle object and
    `@save(vm.selectedTodo, before='updateTodo')` makes the binder save
    middle object's data back to `vm.selectedTodo` before executing the
    command `updateTodo`, bound in line 36.
-   Line 5,6,17,27,32,33: Binding each input field to each property of
    the middle object through `fx`.
-   Line 17: Binding model of *Radiogroup* to `vm.priorityList` to
    display 3 priority levels.

After modifying item's detail, you can click the "Update" button to save
the modification or "Reload" to revert back original data. These two
functions are implemented in command methods:

```java

    @Command
    @NotifyChange("selectedTodo")
    public void updateTodo(){
        //update data
        selectedTodo = todoListService.updateTodo(selectedTodo);

        //update the model, by using ListModelList, you don't need to notify todoListModel change
        //by reseting an item , it make listbox only refresh one item
        todoListModel.set(todoListModel.indexOf(selectedTodo), selectedTodo);
    }

    //when user clicks the update button
    @Command @NotifyChange("selectedTodo")
    public void reloadTodo(){
        //do nothing, the selectedTodo will reload by notify change
    }
```

-   Line 9: `ListModelList` can update its change to the client
    automatically, you don't have to notify change of `todoListModel`.

then we can invoke them by command binding:

```xml
        <hlayout>
            <button onClick="@command('updateTodo')" label="Update"/>
            <button onClick="@command('reloadTodo')" label="Reload"/>
        </hlayout>
```

### Input Validation

Under MVVM approach, ZK provides a **validator** to help developers
perform user input validation. Validator is a reusable element that
performs validation. If you bind a component's attribute to a validator,
binder will use it to validate attribute's value automatically before
saving to a ViewModel or to a middle object. Here we implement a
validator to avoid empty value of todo's subject.

**Define a validator in the ViewModel**

```java
    //the validator is the class to validate data before set ui data back to todo
    public Validator getTodoValidator(){
        return new AbstractValidator() {

            public void validate(ValidationContext ctx) {
                //get the form that will be applied to todo
                Form fx = (Form)ctx.getProperty().getValue();
                //get filed subject of the form
                String subject = (String)fx.getField("subject");

                if(Strings.isBlank(subject)){
                    Clients.showNotification("Subject is blank, nothing to do ?");
                    //mark the validation is invalid, so the data will not update to bean
                    //and the further command will be skipped.
                    ctx.setInvalid();
                }
            }
        };
    }
```

-   Line 2: Returning a validator object by a getter method makes it as
    a ViewModel's property, so we can bind it to an attribute.
-   Line 3: In most case, we can create a validator by extending
    <javadoc>org.zkoss.bind.validator.AbstractValidator</javadoc> and
    override `validate()` instead of creating from scratch.
-   Line 7: Get user input from `ValidationContext`. In our example,
    because we will apply this validator to form binding, we expect
    `ctx.getProperty().getValue()` returns a `Form` object.
-   Line 9: You can get every field that middle object contains with a
    property name.
-   Line 15: Call `set Invalid()` to fail the validation then further
    command execution will be skipped.

Then apply this validator with data binding expression.

```xml

    <vlayout
    form="@id('fx') @load(vm.selectedTodo)
    @save(vm.selectedTodo, before='updateTodo')
    @validator(vm.todoValidator)">
```

Hence, if `vm.todoValidator` fails validation, ZK won't execute
`updateTodo` command. Then binder won't save value to `selectedTodo`.

### Complete a Todo

We want clicking a *Checkbox* in front of each todo item to complete a
todo item. First, we implement business logic to complete a todo item.

```java

    @Command
    //@NotifyChange("selectedTodo") //use postNotifyChange() to notify dynamically
    public void completeTodo(@BindingParam("todo") Todo todo){
        //save data
        todo = todoListService.updateTodo(todo);
        if(todo.equals(selectedTodo)){
            selectedTodo = todo;
            //for the case that notification is decided dynamically
            //you can use BindUtils.postNotifyChange to notify a value changed
            BindUtils.postNotifyChange(null, null, this, "selectedTodo");
        }
    }
```

-   LIne 3: ZK allows you to pass any object or value that can be
    referenced by EL on a ZUL to command method through command binding
    annotation. Your command method's signature should have a
    corresponding parameter that is annotated with `@BindingParam` with
    the same type and key.
-   Line 10: We demonstrate programmatic way to notify change by
    `BindUtils.postNotifyChange()`. We leave the first and second
    parameters to null as default. The third parameters is the target
    bean that is changed and the fourth parameter is the changed
    property name.

Then we bind `onCheck` to the command `completeTodo`.

```xml

    <template name="model">
        <listitem sclass="@bind(each.complete?'complete-todo':'')">
            <listcell>
                <checkbox checked="@bind(each.complete)"
                    onCheck="@command('completeTodo',todo=each)"/>
            </listcell>
            ...
        </listitem>
    </template>
```

-   Line 4,5: Command binding allows you to pass an arguments in
    key-value pairs. We pass `each` object with key `todo`.

## Delete


Implementing a delete function is very similar to "completing a todo",
we perform business logic and notify change.

```java

    @Command
    //@NotifyChange("selectedTodo") //use postNotifyChange() to notify dynamically
    public void deleteTodo(@BindingParam("todo") Todo todo){
        //save data
        todoListService.deleteTodo(todo);

        //update the model, by using ListModelList, you don't need to notify todoListModel change
        todoListModel.remove(todo);

        if(todo.equals(selectedTodo)){
            //refresh selected todo view
            selectedTodo = null;
            //for the case that notification is decided dynamically
            BindUtils.postNotifyChange(null, null, this, "selectedTodo");
        }
    }
```

-   Line 8: When you change (add or remove) items in a `ListModelList`
    object, it will automatically reflect to *Listbox*'s rendering.

Next, bind `onClick` to the command `deleteTodo` then we are done
editing this function.

```xml
    <template name="model">
        <listitem sclass="@bind(each.complete?'complete-todo':'')">
            <listcell>
                <checkbox checked="@bind(each.complete)"
                    onCheck="@command('completeTodo',todo=each)"/>
            </listcell>
            <listcell>
                <label value="@bind(each.subject)"/>
            </listcell>
            <listcell>
                <button onClick="@command('deleteTodo',todo=each)"
                    image="/imgs/cross.png" width="36px"/>
            </listcell>
        </listitem>
    </template>
```

-   Line 11,12: In order to know which `Todo` object we should delete,
    we pass the deleting `Todo` by `todo=each`. The `todo` is key and
    `each` is value.

The key development activities under MVVM approach are designing a
ViewModel, implementing command methods, and binding attributes to a
ViewModel. You can see that the relationship between ZUL and ViewModel
is relatively decoupling and only established by data binding
expressions.

After completing above steps, please visit
http://localhost:8080/zkessentials/chapter4/todolist-mvvm.zul to see the
result.


