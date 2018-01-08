# MVVM Approach

In addition to the MVC approach, ZK also allows you to design your
application in another architecture: [ *MVVM
(Model-View-ViewModel)*](http://books.zkoss.org/zk-mvvm-book/8.0/index.html).
This architecture also divides an application into 3 roles: **View**, **Model**,
and **ViewModel**. The View and Model plays the same roles as they do in
MVC. The ViewModel in MVVM acts like a special Controller for the View
which is responsible for exposing data from the Model to the View and
for providing required action and logic for user requests from the View.
The ViewModel is a View abstraction, which contains a View's state and
behavior. The biggest difference from the Controller in the MVC is that
**ViewModel should not contain any reference to UI components** and knows nothing about the View's visual elements. Hence this clear separation between View and ViewModel decouples ViewModel from View and makes ViewModel more reusable and more abstract.

Since the ViewModel contains no reference to UI components, you cannot
control components directly e.g. to get value from them or set value to
them. Therefore we need a mechanism to synchronize data between the View
and ViewModel. Additionally, this mechanism also has to bridge events
from the View to the action provided by the ViewModel. This mechanism,
the kernel operator of the MVVM design pattern, is a data binding system
called "ZK Bind" provided by the ZK framework. In this binding system,
the [
**binder**](http://books.zkoss.org/wiki/ZK Developer's Reference/MVVM/DataBinding/Binder)
plays the key role to operate the whole mechanism. The binder is like a
broker and responsible for communication between View and ViewModel.

![](../images/ze-Mvvm-architecture.png)

This section we will demonstrate how to implement the same target
application under MVVM approach.
