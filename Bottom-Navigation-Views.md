## Overview

[Bottom navigation](https://material.google.com/components/bottom-navigation.html) provide easy navigation for switching between 3 to 5 different views.  It was recently added to the [Material Design spec](https://material.google.com/components/bottom-navigation.html), which provides an alternative between [tabs](https://material.google.com/components/tabs.html) and [navigation drawers](https://material.google.com/patterns/navigation-drawer.html).  This widget was also added to v25 of the support design library.

<center><img src="https://imgur.com/MN4szO6.png" height="400"></center>

## Setup

See the [[design support library|Design-Support-Library#setup]] setup instructions for the Gradle config you need to add. 

Next, simply declare a `BottomNavigationView` instance and make sure to set `alignParentBottom=true` to ensure that the view is placed on the bottom layout:

```xml
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.design.widget.BottomNavigationView
        android:id="@+id/bottom_navigation"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:menu="@menu/menu_bottom_navigation" />
</android.support.constraint.ConstraintLayout>
```

Next, similar to how menu items in the [[Toolbar|Using-the-App-ToolBar]], declare the tabbed items and the icons that will be displayed in a `res/menu/menu_bottom_navigation.xml` file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/action_favorites"
        android:enabled="true"
        android:icon="@drawable/ic_favorite_white_24dp"
        android:title="@string/text_favorites"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/action_schedules"
        android:enabled="true"
        android:icon="@drawable/ic_access_time_white_24dp"
        android:title="@string/text_schedules"
        app:showAsAction="ifRoom" />
    <item
        android:id="@+id/action_music"
        android:enabled="true"
        android:icon="@drawable/ic_audiotrack_white_24dp"
        android:title="@string/text_music"
        app:showAsAction="ifRoom" />
</menu>
```
You can create the icons by choosing them from the `New` -> `Vector Asset`:

<img src="https://imgur.com/j9SkJOE.png" width="300"/>

Next, you can choose the `Action` and find the heart symbol:

<img src="https://imgur.com/K6IH2kG.png" width="300"/>

Save the file as `ic_favorite_white_24dp.xml`.  You can change the fill color after importing the vector drawable by modifying the `fillColor`:

```xml
<!-- change fill color to white -->
<path
   android:fillColor="#FFFFFFFF"
   android:pathData="M12,21.35l-1.45,-1.32C5.4,15.36 2,12.28 2,8.5 2,5.42 4.42,3 7.5,3c1.74,0 3.41,0.81 4.5,2.09C13.09,3.81 14.76,3 16.5,3 19.58,3 22,5.42 22,8.5c0,3.78 -3.4,6.86 -8.55,11.54L12,21.35z"/>
</path>
```

You can repeat this process for the other icons.  You can also download the icons directly from this [GitHub repo](https://github.com/hitherejoe/BottomNavigationViewSample/tree/master/app/src/main/res/drawable.) too.

Finally, you will need to set the navigation select handler manually:

```java
BottomNavigationView bottomNavigationView = (BottomNavigationView) findViewById(R.id.bottom_navigation);

bottomNavigationView.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
  @Override
  public boolean onNavigationItemSelected(@NonNull MenuItem item) {
    switch (item.getItemId()) {
      case R.id.action_favorites:
        // do something here
        return true;
      case R.id.action_schedules:
       // do something here
        return true;
      case R.id.action_music:
        // do something here
        return true;
      default: return true;
    } 
  }
});      
```

### Using with fragments

You can use the bottom navigation drawer to swap fragments:

```java
public class MainActivity extends AppCompatActivity {
   @Override
   protected void onCreate(Bundle savedInstanceState) {
     final FragmentManager fragmentManager = getSupportFragmentManager();

     // define your fragments here
     final Fragment fragment1 = new FirstFragment();
     final Fragment fragment2 = new SecondFragment();
     final Fragment fragment3 = new ThirdFragment(); 

     // handle navigation selection
     bottomNavigationView.setOnNavigationItemSelectedListener(               
        new BottomNavigationView.OnNavigationItemSelectedListener() {
          @Override
          public boolean onNavigationItemSelected(@NonNull MenuItem item) {
            Fragment fragment;
            switch (item.getItemId()) {
              case R.id.action_favorites:
                  fragment = fragment1;
                  break;
              case R.id.action_schedules:
                  fragment = fragment2;
                  break;
              case R.id.action_music:
              default:
                  fragment = fragment3;
                  break;
              }
            fragmentManager.beginTransaction().replace(R.id.flContainer, fragment).commit();
            return true;
         }
     });
     // Set default selection
     bottomNavigationView.setSelectedItemId(R.id.action_favorites);
  }
}

```
### Customizing styles

The bottom navigation view also can be customized with several different options, including:
`app:itemBackground`, `app:itemIcontint`, and `app:ItemTextColor`:
- `app:itemBackground` - The background for the bottom navigation's items
- `app:itemIconTint` - The tinting for the bottom navigation's items' icons
- `app:itemTextColor` - The text color for the bottom navigation's items' text

Example:
```xml
<android.support.design.widget.BottomNavigationView
  app:itemBackground="@color/colorPrimary"
  app:itemIconTint="@color/white"
  app:itemTextColor="@color/white" />
```

### Customizing menu icon state

<img src="https://imgur.com/wEwveJG.png">

When a bottom navigation menu item is selected, it is in a `checked` state.  You can create a [[state list drawable|Drawables#state-list]] to render different icons depending on whether the menu item is selected:

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector
    xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_checked="true" android:drawable="@drawable/instagram_home_filled_24"/>
    <item android:state_checked="false" android:drawable="@drawable/instagram_home_outline_24" />
</selector>
```
## References

* <https://medium.com/@hitherejoe/exploring-the-android-design-support-library-bottom-navigation-drawer-548de699e8e0>
