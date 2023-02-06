# Include a Separate Page

To complete the page, we need to put those individual pages into
corresponding area of the *BorderLayout*.

For all areas, we use *`<apply>`*, [a shadow component](/shadow_components/README.md), to combine separated pages, and it doesn't consume extra server memory. This component can inject a separated zul for you when the parent zul is loaded. This usage is presented below:

**chapter1/index.zul**
```xml
<?link rel="stylesheet" type="text/css" href="/style.css"?>
<zk>
	<custom-attributes org.zkoss.zul.image.preload="true"/>
	<borderlayout hflex="1" vflex="1">
		<north vflex="min" border="none" >
			<apply templateURI="/chapter1/banner.zul"/>
		</north>
		<west width="260px" border="none" collapsible="true" splittable="true" minsize="300">
			<apply templateURI="/chapter1/sidebar.zul"/>
		</west>
		<center id="mainContent" autoscroll="true">
			<apply templateURI="/chapter1/main.zul"/>
		</center>
		<south border="none">
			<apply templateURI="/chapter1/footer.zul"/>
		</south>
	</borderlayout>
</zk>
```
-   Line 1: This directive links a external style sheet under root
    folder.
-   Line 6, 9, 12, 15: Specify a separated zul path at `templateURI` attribute to inject a page into current page.

For CE users, you still can use [`<include>`](https://www.zkoss.org/wiki/ZK%20Component%20Reference/Essential%20Components/Include) as an alternative to `<apply>`.
Note: the `<include>` component's API is different from `<apply>`. For example, `<include>` uses the `src="..."` attribute instead of the `templateURI="..."` attribute to target the included page. Please refer to the documentation link above for more informations.
