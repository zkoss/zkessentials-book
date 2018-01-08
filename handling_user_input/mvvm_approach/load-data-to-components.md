## Create a ViewModel
ViewModel is an abstraction of View which contains the View's data,
state and behavior. It extracts the necessary data to be displayed on
the View from one or more Model classes. Those data are exposed through
getter and setter method like JavaBean's property. ViewModel is also a
"Model of the View". It contains the View's state (e.g. user's
selection, whether a component is enabled or disabled) that might change
during user interaction.

In ZK, the ViewModel can simply be a POJO which contains data to display
on the ZUL and doesn't have any components. The example application
displays 2 kinds of data: the user's profile and country list in the
*Listbox*. The ViewModel should look like the following:

**Define properties in a ViewModel**

``` java

public class ProfileViewModel implements Serializable{

    //services
    AuthenticationService authService = new AuthenticationServiceChapter3Impl();
    UserInfoService userInfoService = new UserInfoServiceChapter3Impl();

    //data for the view
    User currentUser;

    public User getCurrentUser(){
        return currentUser;
    }

    public List<String> getCountryList(){
        return CommonInfoService.getCountryList();
    }

    @Init // @Init annotates a initial method
    public void init(){
        UserCredential cre = authService.getUserCredential();
        currentUser = userInfoService.findUser(cre.getAccount());
        if(currentUser==null){
            //TODO handle un-authenticated access
            return;
        }
    }
    ...
}
```

-   Line 4,5: The ViewModel usually contains service classes that are
    used to get data from them or perform business logic.
-   Line 8, 10: We should define current user profile data and its
    getter method to be displayed in the zul.
-   Line 14: ViewModel exposes its data by getter methods, it doesn't
    have to define a corresponding member variable. Hence we can expose
    country list by getting from the service class.
-   Line 18: There is a marker annotation `@Init` for a method which
    should be at most one in each ViewModel and ZK will invoke this
    method after instantiating a ViewModel class. We should perform
    initialization in it, e.g. get user credential to initialize
    `currentUser`.

