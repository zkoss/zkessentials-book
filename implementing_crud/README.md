Target Application
==================

In this chapter, we are going to build a todo list management
application with 4 basic operations, Create, Read, Update, and Delete
(CRUD). The application's user interface looks like the images below:

![ center | 600px](Tutorial-ch6-app.png  " center | 600px")

Select an Item:

![ center | 600px](Tutorial-ch6-app-selected.png  " center | 600px")

<div style="text-align:center">
**Select a Todo Item**

</div>
It is a personal todo list management system and it has following
features:

1.  List all todo items
2.  Create a todo item.

    :   Type item name in upper-left textbox and click
        ![](Tutorial-ch6-plus.png "fig:Tutorial-ch6-plus.png") or press
        "Enter" key to create a new todo item.

3.  Finish a todo item.

    :   Click the checkbox in front of a todo item to mark it as
        finished and the item name will be decorated with line-through.

4.  Modify a todo item.

    :   Click an existing item and the detail editor appears. Then you
        can edit the item's details.

5.  Delete a todo item.

    :   Click ![](Tutorial-ch6-cross.png "fig:Tutorial-ch6-cross.png")
        to delete an existing todo item.

In this chapter, we will show how to implement the target application
using both the MVC and MVVM approaches. If you are not familiar with
these two approaches, we suggest you to read [ Get ZK Up and Running
with MVC](ZK Getting Started/Get ZK Up_and_Running_with_MVC "wikilink")
and [ Get ZK Up and Running with
MVVM](ZK Getting Started/Get ZK Up_and_Running_with_MVVM "wikilink").
These two approaches are mutually interchangeable. You can choose one of
them depending on your situation.

MVC Approach
============

If you have read previous chapters, constructing the user interface for
the example application should not be a big problem. Let's look at the
layout first and ignore the details.

**Layout in chapter6/todolist-mvc.zul**

``` {.xml}

<?link rel="stylesheet" type="text/css" href="/style.css"?>
<window apply="org.zkoss.essentials.chapter6.mvc.TodoListController"
    border="normal" hflex="1" vflex="1" contentStyle="overflow:auto">
    <caption src="/imgs/todo.png" sclass="fn-caption" label="Todo List (MVC)"/>
    <borderlayout>
        <center autoscroll="true" border="none">
            <vlayout hflex="1" vflex="1">
                <!-- todo creation function-->
                <!-- todo list -->
            </vlayout>
        </center>
        <east id="selectedTodoBlock" visible="false"
        width="300px" border="none" collapsible="false"
        splittable="true" minsize="300" autoscroll="true">
            <vlayout >
                <!-- detail editor -->
            </vlayout>
        </east>
    </borderlayout>
</window>
```

-   Line 5: We construct the user interface with a *Border Layout* to
    separate user interface into 2 areas.
-   Line 6: The center area contains a todo creation function and a todo
    list.
-   Line 12, 13: The east area is a todo item detail editor which is
    invisible if no item selected.

Read
----

As we talked in previous chapters, we can use *Template* to define how
to display a data model list with implicit variable `each`.

**Display a ToDo List**

``` {.xml}
...
    <listbox id="todoListbox" vflex="1">
        <listhead>
            <listheader width="30px" />
            <listheader/>
            <listheader hflex="min"/>
        </listhead>
        <template name="model">
            <listitem sclass="${each.complete?'complete-todo':''}"
                value="${each}">
                <listcell>
                    <checkbox forward="onCheck=todoListbox.onTodoCheck"
                        checked="${each.complete}"/>
                </listcell>
                <listcell>
                    <label value="${each.subject}"/>
                </listcell>
                <listcell>
                    <button forward="onClick=todoListbox.onTodoDelete"
                        image="/imgs/cross.png" width="36px"/>
                </listcell>
            </listitem>
        </template>
    </listbox>
...
```

-   Line 8: The default value for the required attribute `name` is
    "model".
-   Line 10: The `${each}` is an implicit variable that you can use
    without declaration inside *Template*, and it represents each object
    of the data model list. We can implement simple presentation logic
    with EL expressions. Here we apply different styles according to a
    flag `each.complete`. We also set a whole object in `value`
    attribute, and later we can get the object in the controller.
-   Line 13: The `each.complete` is a boolean variable so that we can
    assign it to `checked`. By doing this, the *Checkbox* will be
    checked if the todo item's `complete` variable is true.
-   Line 12, 19: The `forward` attribute is used to forward events to
    another component and we will talk about it in later sections.

In the controller, we should provide a data model for the *Listbox*.

``` {.java}
public class TodoListController extends SelectorComposer<Component>{


    //wire components
    ...
    @Wire
    Listbox todoListbox;

    ...

    //services
    TodoListService todoListService = new TodoListServiceChapter6Impl();

    //data for the view
    ListModelList<Todo> todoListModel;
    ListModelList<Priority> priorityListModel;
    Todo selectedTodo;


    @Override
    public void doAfterCompose(Component comp) throws Exception{
        super.doAfterCompose(comp);

        //get data from service and wrap it to list-model for the view
        List<Todo> todoList = todoListService.getTodoList();
        todoListModel = new ListModelList<Todo>(todoList);
        todoListbox.setModel(todoListModel);

        ...
    }
...
}
```

-   Line 25 \~ 27: We initialize the data model in `doAfterCompose()`.
    Get data from the service class `todoListService` and create a
    `ListModelList` object. Then set it as the data model of
    `todoListbox`.

There is a priority radiogroup in todo item detail editor appeared on
the right hand side when you select an item.

![ center](Tutorial-ch6-priority.png  " center")

<div style="text-align:center">
**Todo Item's Priority Radiogroup**

</div>
In our application, its priority labels come from an enumerating
`Priority` instead of a static text. We can still use *Template* to
define how to create each *Radio* under a *Radiogroup*. The zul looks
like as follows:

``` {.xml}
...
        <row>
            <cell sclass="row-title">Priority :</cell>
            <cell>
                <radiogroup id="selectedTodoPriority">
                    <template name="model">
                        <radio label="${each.label}"/>
                    </template>
                </radiogroup>
            </cell>
        </row>
...
```

-   Line 6 \~8: Define how to create each *Radio* with *Template* and
    assign `each.label` to `label` attribute.

We also need to provide a data model for the *Radiogroup* in the
controller:

``` {.java}
public class TodoListController extends SelectorComposer<Component>{


    //wire components
    ...
    @Wire
    Listbox todoListbox;

    ...
    @Wire
    Radiogroup selectedTodoPriority;
    ...

    //services
    TodoListService todoListService = new TodoListServiceChapter6Impl();

    //data for the view
    ListModelList<Todo> todoListModel;
    ListModelList<Priority> priorityListModel;
    Todo selectedTodo;

    @Override
    public void doAfterCompose(Component comp) throws Exception{
        super.doAfterCompose(comp);

        //get data from service and wrap it to list-model for the view
        List<Todo> todoList = todoListService.getTodoList();
        todoListModel = new ListModelList<Todo>(todoList);
        todoListbox.setModel(todoListModel);

        priorityListModel = new ListModelList<Priority>(Priority.values());
        selectedTodoPriority.setModel(priorityListModel);
    }
...
}
```

-   Line 31, 32: Create a `ListModelList` with `Priority` and set it as
    a model of `selectedTodoPriority`.

Create
------

After typing the todo item name, we can save the item by either clicking
the button with the plus icon
(![](Tutorial-ch6-plus.png "fig:Tutorial-ch6-plus.png")) or pressing
"Enter" key. Therefore, we have to listen to 2 events: `onClick` and
`onOK`. For handling other key pressing events, please refer to
[ZK\_Developer's\_Reference/UI\_Patterns/Keystroke\_Handling](ZK_Developer's_Reference/UI_Patterns/Keystroke_Handling "wikilink").

``` {.java}
public class TodoListController extends SelectorComposer<Component>{

    //wire components
    @Wire
    Textbox todoSubject;

    //services
    TodoListService todoListService = new TodoListServiceChapter6Impl();

    //data for the view
    ListModelList<Todo> todoListModel;
    ListModelList<Priority> priorityListModel;
    Todo selectedTodo;

    ...

    //when user clicks on the button or enters on the textbox
    @Listen("onClick = #addTodo; onOK = #todoSubject")
    public void doTodoAdd(){
        //get user input from view
        String subject = todoSubject.getValue();
        if(Strings.isBlank(subject)){
            Clients.showNotification("Nothing to do ?",todoSubject);
        }else{
            //save data
            selectedTodo = todoListService.saveTodo(new Todo(subject));
            //update the model of listbox
            todoListModel.add(selectedTodo);
            //set the new selection
            todoListModel.addToSelection(selectedTodo);

            //refresh detail view
            refreshDetailView();

            //reset value for fast typing.
            todoSubject.setValue("");
        }
    }
...
}
```

-   Line 18: Listen the button's `onClick` event and "Enter" key
    pressing event: `onOK`.
-   Line 19: This method adds a todo item, update the data model of
    *Listbox*, change the selection to a newly created one, then reset
    the input field of the *Textbox*.
-   Line 21: Get user input in the *Textbox* `todoSubject` by
    `getValue()`.
-   Line 23: Show a notification at the right hand side of the *Textbox*
    `todoSubject`.
-   Line 28: When you change (add or remove) items in a `ListModelList`
    object, it will automatically render in the *Listbox*'s.
-   Line 30: Call `addToSelection()` to assign a component's selection
    and it will automatically reflect to the corresponding widget's
    selection.

Update
------

To update a todo item, you should select an item first then detail
editor will appear. The following codes demonstrate how to listen a
"onSelect" event and display the item's detail.

``` {.java}
public class TodoListController extends SelectorComposer<Component>{


    //wire components
    @Wire
    Textbox todoSubject;
    @Wire
    Button addTodo;
    @Wire
    Listbox todoListbox;

    @Wire
    Component selectedTodoBlock;
    @Wire
    Checkbox selectedTodoCheck;
    @Wire
    Textbox selectedTodoSubject;
    @Wire
    Radiogroup selectedTodoPriority;
    @Wire
    Datebox selectedTodoDate;
    @Wire
    Textbox selectedTodoDescription;
    @Wire
    Button updateSelectedTodo;


    //when user selects a todo of the listbox
    @Listen("onSelect = #todoListbox")
    public void doTodoSelect() {
        if(todoListModel.isSelectionEmpty()){
            //just in case for the no selection
            selectedTodo = null;
        }else{
            selectedTodo = todoListModel.getSelection().iterator().next();
        }
        refreshDetailView();
    }

    private void refreshDetailView() {
        //refresh the detail view of selected todo
        if(selectedTodo==null){
            //clean
            selectedTodoBlock.setVisible(false);
            selectedTodoCheck.setChecked(false);
            selectedTodoSubject.setValue(null);
            selectedTodoDate.setValue(null);
            selectedTodoDescription.setValue(null);
            updateSelectedTodo.setDisabled(true);

            priorityListModel.clearSelection();
        }else{
            selectedTodoBlock.setVisible(true);
            selectedTodoCheck.setChecked(selectedTodo.isComplete());
            selectedTodoSubject.setValue(selectedTodo.getSubject());
            selectedTodoDate.setValue(selectedTodo.getDate());
            selectedTodoDescription.setValue(selectedTodo.getDescription());
            updateSelectedTodo.setDisabled(false);

            priorityListModel.addToSelection(selectedTodo.getPriority());
        }
    }
...
}
```

-   Line 29: Use `@Listen` to listen `onSelect` event of the *Listbox*
    whose id is `todoListbox`.
-   Line 30: This method checks `todoListModel`'s selection and
    refreshes the detail editor.
-   Line 35: Get user selection from data model by `getSelection()`
    which returns a `Set`.
-   Line 40: If an item is selected, it makes detail editor visible and
    pushes data into those input components of the editor by calling
    setter methods. If no item is selected, it makes detail editor
    invisible and clear all input components' value.
-   Line 53: Make the detail editor visible when `selectedTodo` is not
    null.
-   Line 60: Use `addToSelection()` to assign a component's selection
    and it will automatically reflect to the corresponding widget's
    selection.

After modifying the item's detail, you can click the "Update" button to
save the modification or "Reload" to revert back original data. The
following codes demonstrate how to implement these functions:

**Handle clicking "update" and "reload" button**

``` {.java}
    //when user clicks the update button
    @Listen("onClick = #updateSelectedTodo")
    public void doUpdateClick(){
        if(Strings.isBlank(selectedTodoSubject.getValue())){
            Clients.showNotification("Nothing to do ?",selectedTodoSubject);
            return;
        }

        selectedTodo.setComplete(selectedTodoCheck.isChecked());
        selectedTodo.setSubject(selectedTodoSubject.getValue());
        selectedTodo.setDate(selectedTodoDate.getValue());
        selectedTodo.setDescription(selectedTodoDescription.getValue());
        selectedTodo.setPriority(priorityListModel.getSelection().iterator().next());

        //save data and get updated Todo object
        selectedTodo = todoListService.updateTodo(selectedTodo);

        //replace original Todo object in listmodel with updated one
        todoListModel.set(todoListModel.indexOf(selectedTodo), selectedTodo);

        //show message for user
        Clients.showNotification("Todo saved");
    }

    //when user clicks the update button
    @Listen("onClick = #reloadSelectedTodo")
    public void doReloadClick(){
        refreshDetailView();
    }
```

-   Line 4: Validate user input and show a notification.
-   Line 9 \~ 13: Update selected `Todo` by getting user input from
    components.
-   Line 16, 19: We save the selected `Todo` object and get an updated
    one. Then, we replace the old one in the list model with the updated
    one.

### Complete a Todo

Click a *Checkbox* in front of a todo item means to finish it. To
implement this feature, the first problem is: how do we know which
*Checkbox* is checked as there are many of them. We cannot listen to a
*Checkbox* event as they are created in template using
`@Listen("onCheck = #todoListbox checkbox")`,thus are created
dynamically. Therefore, we introduce the [ "Event
Forwarding"](ZK%20Developer's%20Reference/Event%20Handling/Event%20Forwarding "wikilink")
feature to demonstrate ZK's flexibility. This feature can forward an
event from a component to another component, so we can forward an
`onCheck` event from each *Checkbox* to the *Listbox* that encloses it,
then we can just listen to the *Listbox*'s events instead of all events
of *Checkbox*.

**extracted from chapter6/todolist-mvc.zul**

``` {.xml}
...
    <listbox id="todoListbox" vflex="1">
...
        <template name="model">
            <listitem sclass="${each.complete?'complete-todo':''}" value="${each}">
                <listcell>
                    <checkbox forward="onCheck=todoListbox.onTodoCheck" checked="${each.complete}"/>
                </listcell>
                <listcell>
                    <label value="${each.subject}"/>
                </listcell>
                <listcell>
                    <button forward="onClick=todoListbox.onTodoDelete" image="/imgs/cross.png" width="36px"/>
                </listcell>
            </listitem>
        </template>
```

-   Line 7: Forward the Checkbox's `onCheck` to an event `onTodoCheck`
    of a *Listbox* whose id is `todoListbox`. The `onTodoCheck` is a
    customized forward event name, and you can use whatever name you
    want. Then we can use `@Listen` to listen this special event name.

Next, we listen to the customized event `onTodoCheck` and mark the todo
as finished.

``` {.java}
public class TodoListController extends SelectorComposer<Component>{
...

    //when user checks on the checkbox of each todo on the list
    @Listen("onTodoCheck = #todoListbox")
    public void doTodoCheck(ForwardEvent evt){
        //get data from event
        Checkbox cbox = (Checkbox)evt.getOrigin().getTarget();
        Listitem litem = (Listitem)cbox.getParent().getParent();

        boolean checked = cbox.isChecked();
        Todo todo = (Todo)litem.getValue();
        todo.setComplete(checked);

        //save data
        todo = todoListService.updateTodo(todo);
        if(todo.equals(selectedTodo)){
            selectedTodo = todo;
            //refresh detail view
            refreshDetailView();
        }
        //update listitem style
        ((Listitem)cbox.getParent().getParent()).setSclass(checked?"complete-todo":"");
    }
...
}
```

-   Line 5: Listen to the customized event name `onTodoCheck ` of a
    *Listbox* `todoListbox` for we already forward `onCheck` to the
    *Listbox* in the zul.
-   Line 6: An event listener method can have a argument, but argument's
    type depends on which event you listen. As the customized event is
    forwarded from another component, the argument should be
    <javadoc>org.zkoss.zk.ui.event.ForwardEvent</javadoc>. This method
    set the `Todo` object of the selected item as complete and decorate
    *Listitem* with line-through by changing its `sclass`.
-   Line 8: You should call `getOrigin()` to get the original event that
    is forwarded. Every event object has a method `getTarget()` that
    allows you get the target component that receives the event.
-   Line 9: Navigate the component tree by `getParent()`.
-   Line 12: Here we get `Todo` object of the selected todo item from
    `value` attribute that we assigned in the zul by
    `<listitem ... value="${each}"/>`

Delete
------

Implement deletion feature is similar to completing a todo item. We also
forward each delete button's
(![](Tutorial-ch6-cross.png "fig:Tutorial-ch6-cross.png")) `onClick`
event to the *Listbox* that encloses those buttons.

**Forward delete button's `onClick`**

``` {.xml}
    <listbox id="todoListbox" vflex="1">
...
        <template name="model">
            <listitem sclass="${each.complete?'complete-todo':''}"
                        value="${each}">
                <listcell>
                    <checkbox forward="onCheck=todoListbox.onTodoCheck"
                                checked="${each.complete}"/>
                </listcell>
                <listcell>
                    <label value="${each.subject}"/>
                </listcell>
                <listcell>
                    <button forward="onClick=todoListbox.onTodoDelete"
                        image="/imgs/cross.png" width="36px"/>
                </listcell>
            </listitem>
        </template>
...
```

-   Line 14, 15: Forward delete button's `onClick` to the *Listbox*'s as
    a custom forward event named `onTodoDelete`.

Then we can listen to the forwarded event and perform deletion.

``` {.java}

    //when user clicks the delete button of each todo on the list
    @Listen("onTodoDelete = #todoListbox")
    public void doTodoDelete(ForwardEvent evt){
        Button btn = (Button)evt.getOrigin().getTarget();
        Listitem litem = (Listitem)btn.getParent().getParent();

        Todo todo = (Todo)litem.getValue();

        //delete data
        todoListService.deleteTodo(todo);

        //update the model of listbox
        todoListModel.remove(todo);

        if(todo.equals(selectedTodo)){
            //refresh selected todo view
            selectedTodo = null;
            refreshDetailView();
        }
    }
```

-   Line 2: Listen the customized event name `onTodoDelete` of a
    *Listbox* that we forward from delete button.
-   Line 7: Since we have set each `Todo` object to each `Listitem`'s
    `value` in the zul, we can get it by `getValue()`

After completing the above steps, vist
<http://localhost:8080/essentials/chapter6/todolist-mvc.zul> to see the
result.

MVVM Approach
=============

Building a user interface for the example application using the MVVM
approach is very similar to building it using the MVC approach, however,
you don't need to give an `id` to components since we don't need to
identify components for wiring. For defining the ViewModel's properties,
we should analyse what data required to display on the user interface or
to be kept as a View's state. There are 4 types of data, todo item's
subject for creating a new todo item, todo item list for displaying all
todos, selected todo item for keeping user selection, and todo's
priority list for *Radiogroup* in detail editor.

``` {.java}
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

Read
----

As we discussed in previous chapter, displaying a collection of data
requires to prepare a data model in the ViewModel.

``` {.java}
public class TodoListViewModel implements Serializable{


    //services
    TodoListService todoListService = new TodoListServiceChapter6Impl();

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

``` {.xml}

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

Create
------

We can create a new todo item by either clicking the button with plus
icon (![](Tutorial-ch6-plus.png "fig:Tutorial-ch6-plus.png")) or
pressing the "Enter" key, therefore we can bind these two events to the
same command method that adds a todo item.

**Command method `addTodo`**

``` {.java}
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

``` {.xml}

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

Update
------

How do we achieve the feature that selecting a todo item then detail
editor becomes visible under MVVM? Simple, just determine editor's
visibility upon selected todo item is null or not.

``` {.xml}
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

``` {.xml}

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

``` {.java}

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

``` {.xml}
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

``` {.java}
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

``` {.xml}

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

``` {.java}

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

``` {.xml}

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

Delete
------

Implementing a delete function is very similar to "completing a todo",
we perform business logic and notify change.

``` {.java}

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

``` {.xml}
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
<http://localhost:8080/essentials/chapter6/todolist-mvvm.zul> to see the
result.

Source Code
===========

-   [ZUL
    pages](https://github.com/zkoss/zkessentials/tree/master/src/main/webapp/chapter6)
-   [Java](https://github.com/zkoss/zkessentials/tree/master/src/main/java/org/zkoss/essentials/chapter6)

