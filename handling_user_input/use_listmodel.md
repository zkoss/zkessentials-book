## Use ZK `ListModel`
To create a model object, we suggest using ZK `org.zkoss.zul.ListModel` implementation e.g. `org.zkoss.zul.ListModelList` instead of Java standard collection object like `java.util.List`. Because it optimizes the rendering performance. When you call `add()` or `remove()`, `ListModel` will notify `Listbox` the data change range, so that `Listbox` can render the differential data instead of rendering the whole list.

If you use Java collection object, then ZK component has no way to know the differential part, so only can render the whole list for each time.
