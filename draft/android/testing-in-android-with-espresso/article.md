One of the integral phase of application development life cycle is testing but the question is why?

The answer is to eliminate the occurrence of [Application Not Responding](link to image) (ANR) dialog or unexpected behavior in application logic. In android, all UI updation is handled by main thread so if the main thread is unable to respond to user events within 5 seconds then it cause the crash in application and gives user an option to close the application instead of keeping the user in an infinite waiting state, resulting in poor user experience.

Application Not Responding dialog is the least expected event that can occur when the main thread( aka UI thread) is blocked for too long like 
- Rendering a larger image
- Performing REST calls on main thread
- Or performing any long calculation

Other reasons are Exceptions, infinite recursive method calls, missing class definitions, [deadlocks](https://en.wikipedia.org/wiki/Deadlock) or implementation of unsupported features on older version like vector drawable etc.

# Test Driven Development
[TDD](https://en.wikipedia.org/wiki/Test-driven_development) is a recursive cycle of writing code, testing and refactoring the code to resolve bugs the moment the appears. Despite of the sleepless nights work, it is inevitable to eliminate bugs completely at the first place though TDD implies the approach to test the working of the application after every modification in code base. The objectives of TDD are
- To meets the requirements that guided the application design and development
- To determine the performance of application under different environment considering 
    - Network connectivity
    - User input
    - Server stability
    - Battery, CPU and memory optimization (memory leaks)
- To validate the application security against SQL injections, data security vulnerabilities
- To verify the working of application specific APIs

# Unit and Instrumentation Testing
**Unit Testing** : Unit tests runs on the local machine i.e. Java Virtual Machine (JVM), mean it requires no device or emulator. Unit tests tends to be fast because units test does not test any android specific dependency (Activity,SharedPreferenc) or [mock objects](https://en.wikipedia.org/wiki/Mock_object) are used to mimic the behavior of android framework dependencies.

> unit tests are located under `package-name/src/test/java/`

**Instrumentation Testing** : Instrumentation tests are specifically designed to test UI of an application and requires emulator or physical device to perform tests. Android UI component depends upon [Context](https://developer.android.com/reference/android/content/Context.html) to access the resources in application like xml, images, string and other resources. Context is provided by android operating system so for instrumentation testing, the context is provided by [Instrumentation](https://developer.android.com/reference/android/app/Instrumentation.html) API to track the interaction between android OS and application i.e. to give commands to activity life cycle, environment details etc.

> instrumentation tests are located under `package-name/src/androidTest/java/`

# Testing Terminologies

- AndroidJUnitRunner : `AndroidJUnitRunner` is a JUnit test runner that lets you run JUnit tests on Android devices while providing annotations to ease-up the testing procedure. For instrumentation test, the test class must be prefixed with @RunWith(AndroidJUnit4.class).

- Annotations : `@Test` Annotation is used to mark a method for testing and methods with `@Before`  annotation executed first to setup environment for testing and methods with `@After` annotation is executed at the end. `@LargeTest` annotation is used to indicate that the test duration can be greater than 1 second.


- Espresso : It is a testing framework for android to write tests for user interface interaction.

- Mockito : [`mockito`](https://github.com/mockito/mockito) is often used for unit testing to create dummy objects to mimic the behavior of objects.

# Instrumentation Testing With Espresso

Espresso is a testing framework by google for UI (user-interface) testing which includes every `View` component like buttons, List, Fragments etc. Espresso is collection of testing APIs, specifically designed for instrumentation/UI testing. UI test ensures that user don’t have poor interaction or encounter unexpected behavior.

Espresso  has various options to find view on screen , perform actions and verify visibility and state/content of view. e.g
```
onView(withId(R.id.my_view))       // onView() is a ViewMatcher</strong>
    .perform(click())              // click()  is a ViewAction</strong>
    .check(matches(isDisplayed()));// matches(isDisplayed()) is a ViewAssertion
```
where

1. ViewMatcher : is used to locate the view on UI hierarchy(tree structure of xml layout components) using `withId(R.id.id_of_view)`, `withText("find by text on view")`.
2. ViewActions  : is used to perform specific action or group of actions on UI views using `ViewInteraction.perform(click(),doubleClick())` or `click()`, `longClick()`, `doubleClick()`, `swipeDown()`, `swipeLeft()`, `swipeRight()`, `swipeUp()`, `typeText()`, `pressKey()`, `clearText()` etc.
3. ViewAssertion : is used to assert view’s state using `ViewInteraction.check(assertion method)` where assertion methods can be `isDisplayed()`, `isEnabled()`, `isRoot()`.

# Setup
- Add espresso dependencies in `build.gradle` app module

    ```
    dependencies {
        implementation fileTree(dir: 'libs', include: ['*.jar'])
    
        // AndroidJUnitRunner and JUnit Rules
        androidTestCompile 'com.android.support.test:runner:1.0.1'
        androidTestCompile 'com.android.support.test:rules:1.0.1'
    
        // espresso support
        androidTestImplementation('com.android.support.test.espresso:espresso-core:3.0.1', {
            exclude group: 'com.android.support', module: 'support-annotations'
        })
        androidTestCompile 'com.android.support.test.espresso:espresso-contrib:3.0.1'
        // for intent mocking
        androidTestCompile 'com.android.support.test.espresso:espresso-intents:3.0.1'
        // for network testing to track idle state
        androidTestCompile 'com.android.support.test.espresso.idling:idling-concurrent:3.0.1'
        androidTestCompile 'com.android.support.test.espresso:espresso-idling-resource:3.0.1'
    
        // other dependencies
    }
    ```
- To enable instrumentation testing, add following dependency

   ```
   testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
   ```

Complete `build.gradle`(app) file 

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 26
    buildToolsVersion "26.0.1"
    defaultConfig {
        applicationId "com.pavneet_singh.espressotestingdemo"
        minSdkVersion 15
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    // as mentioned above
}
```
> The values for library version can differ in your project due to future updates and the rest of the configuration values project dependent.

### Turning-off Default Device Animation
The default animations can cause issue during the espresso testing process so it is recommended to turn-off the animation on device and emulator before testing.
- Open Settings
- Select Developer Options (if not activated, then click seven times on build version/number in about device settings)
- Disable
    1. Window animation scale
    2. Transition animation scale
    3. Animator duration scale

# First Espresso Test with Buttons and EditText

`MainActivity` which contains Button and EdiTtext, on which we will perform the testing to verify the visibility, entered values and click operation.

### Let's setup MainActivity with the following code

`activity_main.xml` layout source code for `MainActivity`
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.pavneet_singh.MainActivity">

    <EditText
        android:layout_marginTop="30dp"
        android:id="@+id/editTextName"
        android:hint="Enter Name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:ems="10"
        android:inputType="textPersonName" />

    <Button
        android:onClick="setDefaultText"
        android:layout_marginTop="60dp"
        android:id="@+id/button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Clear" />


    <android.support.design.widget.FloatingActionButton
        android:id="@+id/fab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="@dimen/fab_margin"
        app:srcCompat="@android:drawable/ic_dialog_email" />

</LinearLayout>
```
The `onClick` attribute in xml points to a public method to handle click event in `MainActivity` as shown below

```
package com.pavneet_singh;

import android.content.Intent;
import android.os.Bundle;
import android.support.design.widget.FloatingActionButton;
import android.support.design.widget.Snackbar;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.Toolbar;
import android.view.View;
import android.widget.EditText;

import com.pavneet_singh.espressotestingdemo.FruitListActivity;
import com.pavneet_singh.espressotestingdemo.R;

public class MainActivity extends AppCompatActivity {

    private EditText editTextName;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        editTextName = (EditText)findViewById(R.id.editTextName);

        FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
        fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                startActivity(new Intent(MainActivity.this, FruitListActivity.class));
            }
        });
    }


    public void setDefaultText(View view) {
        editTextName.setText("");
    }
}
```

### Writing Espresso Test

To Add a test class corresponding to `MainActivity` 
1. Open the Java file containing the code you want to test.
2. Click the class or method you want to test, then press Ctrl+Shift+T
3. In the menu that appears, click Create New Test.
4. In the Create Test dialog, edit any fields and select any methods to generate, and then click OK.
5. In the Choose Destination Directory dialog i.e. `androidTest` for an instrumented test and Then click OK.

Add the following code into `MainActivityTest` class

```
package com.pavneet_singh;

// Note the static imports, which enhance the code clarity by reducing code length

import static android.support.test.espresso.Espresso.onView;
import static android.support.test.espresso.action.ViewActions.click;
import static android.support.test.espresso.action.ViewActions.closeSoftKeyboard;
import static android.support.test.espresso.action.ViewActions.typeText;
import static android.support.test.espresso.matcher.ViewMatchers.withHint;
import static android.support.test.espresso.matcher.ViewMatchers.withId;
import static android.support.test.espresso.assertion.ViewAssertions.matches;
import static android.support.test.espresso.matcher.ViewMatchers.withText;

import android.support.test.espresso.ViewAssertion;
import android.support.test.espresso.action.ViewActions;
import android.support.test.filters.LargeTest;
import android.support.test.rule.ActivityTestRule;
import android.support.test.runner.AndroidJUnit4;

import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;
import com.pavneet_singh.espressotestingdemo.R;

@RunWith(AndroidJUnit4.class)
@LargeTest
public class MainActivityTest {

    // To launch the mentioned activity under testing
    @Rule
    public ActivityTestRule<MainActivity> mActivityRule =
            new ActivityTestRule<>(MainActivity.class);

    @Test
    public void testHintVisibility(){
        // check hint visibility
        onView(withId(R.id.editTextName)).check(matches(withHint("Enter Name")));
        // enter name
        onView(withId(R.id.editTextName)).perform(typeText("Testing"),closeSoftKeyboard());

        onView(withId(R.id.editTextName)).check(matches(withText("Testing")));
    }

    @Test
    public void testButtonClick(){
        // enter name
        onView(withId(R.id.editTextName)).perform(typeText("Testing"),closeSoftKeyboard());
        // clear text
        onView(withText("Clear")).perform(click());

        // check hint visibility after the text is cleared
        onView(withId(R.id.editTextName)).check(matches(withHint("Enter Name")));
    }

}
```

To Run Test
- Select the `MainActivityTest` under `package-name/androidTest`
- Right click on the file
- Select run `MainActivityTest`

Or via short cut

- Windows : shift + F10
- Mac     : Control + R

`onView` also has an overloaded version which can accept powerful [hamcrest Matcher](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/Matcher.html) methods to go one step beyond to perform specific operations like 

- [AllOf](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/core/AllOf.html) : To perform multiple operations together.
    - onView(AllOf.allOf(withId(R.id.editTextName),withId(R.id.editTextSameName))).check(matches(withText("Testing")));

- [StringContains](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/core/StringContains.html) : To match a part of input string in the targeted view text.

    - onView(withId(R.id.editTextName)).check(matches(withText(Matchers.containsString("Test"))));

- [StringStartsWith](http://hamcrest.org/JavaHamcrest/javadoc/1.3/org/hamcrest/core/StringStartsWith.html) : To find the view with prefix.
    - onView(withText(startsWith("prefix"))).perform(click());

### Testing ListView or RecyclerView

ListView or Spinner uses `AdapterView` that is displays the data at run time from the adapter so as opposed to other views, `adapterview` does not display all the list items at the same time so `onView` would not find views that are not currently loaded.

`onData()` is specifically applied to bring the desired list item into focus before performing any operation on the given position.

```
// click on 3rd position
// where the type of data source is string
onData(allOf(is(instanceOf(String.class)))).atPosition(2).perform(click());

// select view with text "item 3"
onData(allOf(is(instanceOf(String.class)), is("item 3"))).perform(click());
```

To perform click on `RecyclerView`, `espresso-contrib` dependency is required which provide `RecyclerView` specific methods as shown below

```
// verify the visibility of recycler view on screen
onView(withId(R.id.news_frag_recycler_list)).check(matches(isDisplayed()));
// perform click on view at 3rd position in RecyclerView
onView(withId(R.id.news_frag_recycler_list))
        .perform(RecyclerViewActions.actionOnItemAtPosition(3, click()));
```

### Starting an Activity with Customize Intent

To start an activity, an Intent object is required by `ActivityTestRule` instance and the overloaded version of `ActivityTestRule` is applied 
``` 
ActivityTestRule(SingleActivityFactory<T> activityFactory, boolean initialTouchMode, boolean launchActivity)
```

where `launchActivity` as `false` mean the activity will not be launched automatically by the test, see the implementation

```
@Rule
public ActivityTestRule<MainActivity> activityTestRule =
        new ActivityTestRule<MainActivity>(MainActivity.class,true,false /*lazy launch activity*/){
            @Override
            protected Intent getActivityIntent() {
                /*added predefined intent data*/
                Intent intent = new Intent();
                intent.putExtra("key","value");
                return intent;
            }
        };

@Test
public void customizeIntent(){
    // note instead of null, an intent object can be passed
    activityTestRule.launchActivity(null);
}

```
### Handling Marshmallow RunTime Permission Model

To enhance data and crucial resource security, Android MarshMallow and above inform the user about the resources used by app while the application is running. Some of them are storage, contacts, location and other hardware resources like sensors, camera etc.

So for testing, runtime permissions needs to be granted first and preferably in methods annotated with `@Before` annotation as 
```
@Before
public void grantPhonePermission() {
    // In M+, trying to call a number will trigger a runtime dialog. Make sure
    // the permission is granted before running this test.
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        getInstrumentation().getUiAutomation().executeShellCommand(
                "pm grant " + getTargetContext().getPackageName()
                        + " android.permission.READ_EXTERNAL_STORAGE");
    }
}
```

### Using Context for Initialization

Often a `context` instance is required to setup libraries before their usage so the `context` instance can be retrieved from `InstrumentationRegistry` instance.

```
@Rule
public ActivityTestRule<NewsActivity> activityTestRule =
        new ActivityTestRule<>(NewsActivity.class);


@Before
public void init(){
    Context context = InstrumentationRegistry.getTargetContext();
    Fresco.initialize(context);
}
```

### Testing Toast Visibility
Toasts are floating messages, shown to user over the current screen. To the appearance of toast, use the below code and carefully pay attention to import statement. 

```
package com.pavneet_singh;

import static android.support.test.espresso.Espresso.onView;
import static android.support.test.espresso.action.ViewActions.click;
import static android.support.test.espresso.action.ViewActions.closeSoftKeyboard;
import static android.support.test.espresso.action.ViewActions.typeText;
import static android.support.test.espresso.matcher.RootMatchers.withDecorView;
import static android.support.test.espresso.matcher.ViewMatchers.isDisplayed;
import static android.support.test.espresso.matcher.ViewMatchers.withHint;
import static android.support.test.espresso.matcher.ViewMatchers.withId;
import static android.support.test.espresso.assertion.ViewAssertions.matches;
import static android.support.test.espresso.matcher.ViewMatchers.withText;

import android.support.test.espresso.contrib.RecyclerViewActions;
import android.support.test.filters.LargeTest;
import android.support.test.rule.ActivityTestRule;
import android.support.test.runner.AndroidJUnit4;

import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;

import com.pavneet_singh.espressotestingdemo.R;

@RunWith(AndroidJUnit4.class)
public class MainActivityTest {

    // To launch the mentioned activity under testing
    @Rule
    public ActivityTestRule<MainActivity> mActivityRule =
            new ActivityTestRule<>(MainActivity.class);


    @Test
    public void testButtonClick() {
        // enter name
        onView(withId(R.id.editTextName)).perform(typeText("Testing"), closeSoftKeyboard());
        // clear text
        onView(withText("Clear")).perform(click());

        // check hint visibility after the text is cleared
        onView(withId(R.id.editTextName)).check(matches(withHint("Enter Name")));
        onView(withId(R.id.editTextName)).check(matches(isDisplayed()));
        onView(withId(R.id.editTextName))
                .perform(RecyclerViewActions.actionOnItemAtPosition(3, click()));
        // check toast visibility
        onView(withText("Testing Toast")).
                inRoot(withDecorView(not(is(activity.getWindow().getDecorView())))).
                check(matches(isDisplayed()));


    }
}
```
# Check View's Visibility in ScrollView

ScrollView can host multiple views (EditText, Buttons, CheckBox etc) vertically or horizontally so it is possible that some views are not currently visible on the screen hence applying `matches(isDisplayed())` cannot be applied on views which are not visible. 

To test those views which are not visible on the current window, use `withEffectiveVisibility` as

```
// check view exists in current layout hierarchy
onView(withId(R.id.view_id)).check(matches(withEffectiveVisibility(ViewMatchers.Visibility.VISIBLE)));
```

# Test Fragment using Espresso

Your Activity (Screen) can be divided into smaller containers called Fragments and during testing you need to first add fragment into host activity which either can be done before starting the actual test or later, when it is required.

```
@Rule
public ActivityTestRule<NewsActivity> activityTestRule =
        new ActivityTestRule<>(NewsActivity.class);

 @Before
 public void yourSetUPFragment() {
    activityTestRule.getActivity()
        .getFragmentManager().beginTransaction();
}
```
Here `@Before` function will be executed before starting the test. It’s like a setup function to load the fragment into the host activity which is appropriate when test is only fragment oriented.

This transaction can also be done later whenever required.

Note : You can also create normal fragment object and add it to host activity using fragment transaction.

# Creating Customize ViewAction
One of the common practice is to change the text in `TextView` but the regular approach of typing text in `EditText` cannot be applied on `TextView` so the below approach cannot be used.

```
onView(withId(R.id.editText)).perform(typeText("my text"), closeSoftKeyboard());
```

The above will fail because input method editor (IME) is not supported by `TextView` mean user cannot input values into `TextView` via keyboard so to type text into `TextView`, a customize `ViewAction` required.

### Typing Text into TextView using customize ViewAction

`ViewAction` is an `abstract` class so to create a customize `ViewAction`, an anonymous class of `ViewAction` type is created.

```
public static ViewAction setTextInTextView(final String value){
    return new ViewAction() {
        @SuppressWarnings("unchecked")
        @Override
        public Matcher<View> getConstraints() {
            return allOf(isDisplayed(), isAssignableFrom(TextView.class));
//                                            ^^^^^^^^^^^^^^^^^^^ 
// To check that the found view is TextView or it's subclass like EditText
// so it will work for TextView and it's descendants  
        }

        @Override
        public void perform(UiController uiController, View view) {
            ((TextView) view).setText(value);
        }

        @Override
        public String getDescription() {
            return "replace text";
        }
    };
}
```

then it can be applied as

```
onView(withId(R.id.textview_id))
        .perform(setTextInTextView("text input"));
```

### Check Attributes Like FontSize

Espresso does not have any matcher to test the font size and many other attribute values though a customize matcher can be created for particular scenarios like this as shown below

- Create a customize `TypeSafeMatcher`

```
public class FontSizeMatcher extends TypeSafeMatcher<View> {

    // field to store values
    private final float expectedSize;

    public FontSizeMatcher(float expectedSize) {
        super(View.class);
        this.expectedSize = expectedSize;
    }

    @Override
    protected boolean matchesSafely(View target) {
    // stop executing if target is not textview
        if (!(target instanceof TextView)){
            return false;
        }
    // target is a text view so apply casting then retrieve and test the desired value
        TextView targetEditText = (TextView) target;
        return targetEditText.getTextSize() == expectedSize;
    }


    @Override
    public void describeTo(Description description) {
        description.appendText("with fontSize: ");
        description.appendValue(expectedSize);
    }
}
```
then apply the testing as

```
onView(withId(R.id.id_of_view)).check(matches(withFontSize(36)));
```

# Testing Intent, IntentAction and ActivityResult

Intent can be used to start an already installed app to handle tasks like open link with default browser as

```
public void onClick(View view) {
    Intent intent = new Intent(Intent.ACTION_VIEW);
    intent.setData(Uri.parse("https://www.pluralsight.com/guides/author/Pavneet-Sing"));
    if (ActivityUtils.resolveIntent(intent,getActivity()))
        startActivity(intent);
    }
```
Test to stub and verify the triggered intent

```
Intents.init(); \\ required for intent stubbing

// create a desired matcher with action as Intent.ACTION_VIEW
Matcher<Intent> expectedIntent = hasAction(Intent.ACTION_VIEW); 

// return stub result when the intent is fired
intending(expectedIntent).respondWith(new Instrumentation.ActivityResult(0, null));

// click the button to fire the `onClick` to open url in browser
onView(withText(R.string.details_story_link)).perform(click());

// validate the previously fired intent and match it against the desired intent
intended(expectedIntent);

// clears the intent state
Intents.release();
```

# Testing Third-party Network Calls

Android supports many popular network calls APIs like `retrofit`, `okhttp`,`volley` etc, to access server data. Network calls are asynchronous and during testing, espresso is unware of the idle time required to finish hence espresso cannot provide the synchronization guarantees in those situations. In order to make Espresso aware of your app's long-running processes, an `idling resource` is used to keep track of idle time to access the data from server and later to display data on screen for testing the views.

Steps :

- Create Espresso idling resource in java package (not any testing package)

    ```
    public class EspressoTestingIdlingResource {
        private static final String RESOURCE = "GLOBAL";
    
        private static CountingIdlingResource mCountingIdlingResource =
                new CountingIdlingResource(RESOURCE);
    
        public static void increment() {
            mCountingIdlingResource.increment();
        }
    
        public static void decrement() {
            mCountingIdlingResource.decrement();
        }
    
        public static IdlingResource getIdlingResource() {
            return mCountingIdlingResource;
        }
    
    }
    ```
- Now you just need to invoke `increment()` method, just before the execution of background process 
    ```
    EspressoTestingIdlingResource.increment(); 
    ```

- Invoke `decrement()` method, after the background task has been finish.

   ```
    EspressoTestingIdlingResource.decrement();
   ```

- Finally, register the `idling` resource into your test class as

    ```
        @Rule
        public ActivityTestRule<NewsActivity> activityTestRule =
                new ActivityTestRule<>(NewsActivity.class);
    
        @Before
        public void registerIdlingResource() {
            // let espresso know to synchronize with background tasks
            IdlingRegistry.getInstance().register(EspressoTestingIdlingResource.getIdlingResource());
        }
    
        @After
        public void unregisterIdlingResource() {
            IdlingRegistry.getInstance().unregister(EspressoTestingIdlingResource.getIdlingResource());
        }
    ```

> IdlingResource is not required for `AsyncTask`, [From docs](https://developer.android.com/reference/android/support/test/espresso/IdlingResource.html) By default, Espresso synchronizes all view operations with the UI thread as well as AsyncTasks.

# Espresso Test Recorder
Android studio provide an espresso test recorder which allows to record the user(tester) event on real app and then convert those event into espresso test cases.

To start recording a test with Espresso Test Recorder, proceed as follows:

1. Click Run > Record Espresso Test.
2. In the Select Deployment Target window, choose the device on which you want to record the test then Click OK.
3. A build is triggered to compile and launch the app along with an activity record panel to record user interaction for espresso test generation.

# Reference

- [List of android testing support library dependencies](https://developer.android.com/topic/libraries/testing-support-library/packages.html#atsl-dependencies)
- [Google testing samples](https://github.com/googlesamples/android-testing)
- [Espresso cheat sheet](https://google.github.io/android-testing-support-library/downloads/espresso-cheat-sheet-2.1.0.pdf)
- [Enable StrictMode to detect hidden or accidental IO or network access](https://developer.android.com/reference/android/os/StrictMode.html)
- [FIRST rule of unit testing](https://github.com/ghsukumar/SFDC_Best_Practices/wiki/F.I.R.S.T-Principles-of-Unit-Testing)

***
I hope, now testing will help you to be a stellar programmer, you can share you love by filling the heart.
