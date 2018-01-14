The purpose of this article is to facilitate android development process with helpful tips and to give insights into handy android development tips regarding android API's to avoid common mistake.

let's get started with tips first

### Data Structures With ArrayAdapter 
Data structures are extremely significant and heavily used while creating adapters to display list of data to users. The parameterized constructors of [ArrayAdapter](zhttps://developer.android.com/reference/android/widget/Adapter.htmlhttps://developer.android.com/reference/android/widget/ArrayAdapter.html) either accepts array or ArrayList but if you look at the source code of ArrayAdapter then internally, it only use arrayList create from passed array by converting the array into an immutable ArrayList as 

```
public ArrayAdapter(@NonNull Context context, @LayoutRes int resource, @NonNull T[] objects) {
    this(context, resource, 0, Arrays.asList(objects));
}

public ArrayAdapter(@NonNull Context context, @LayoutRes int resource,
        @IdRes int textViewResourceId, @NonNull T[] objects) {
    this(context, resource, textViewResourceId, Arrays.asList(objects));
}
```

The immutable list will not allow modification in the resultant so to level up your android development skills, you should consider to look at the source code of android APIs available [on github](https://github.com/aosp-mirror/platform_frameworks_base).


#### Things to be considered while using DataStructures

- Ordering : Often, use of `HashMap` or `HashSet` as data source with CursorAdapter or customize adapter cause issues while implementing search mechanism because they does not support ordering, instead use `LinkedHashMap` or `HashSet`.
- Synchronization : HashSet, TreeSet, and LinkedHashSet are synchronized and should not be modified by concurrent thread, instead use 

```Set dataSet = Collections.synchronizedSet(new HashSet(...));
Set dataSet = Collections.synchronizedSet(new LinkedHashSet(...));
Set dataSet = Collections.synchronizedSet(new TreeSet(...));```

- Performance : TreeSet is bit slower because it requires sorting with every modification to maintain the natural order of data.

### Array 
Arrays often used for being memory efficient but when the requirement is to store key-value pairs (for multi-selection list) then `HashMap` is considered as the first choice for being popular and effective but android also provides memory efficient family of `SparseArray` to store keys as `int` or `long` with values as `Object`, `Boolean`, `Integer`, `Long` as

```
API                 |  Key-Value pairs    | HashMap |
------------------- | ------------------- | ---------
ArrayMap            |  <K,V>              | HashMap< K, V>
SparseArray         |  <Integer, Object>  | HashMap<Integer, Object>
SparseBooleanArray  |  <Integer, Boolean> | HashMap<Integer, Boolean>
SparseIntArray      |  <Integer, Integer> | HashMap<Integer, Integer>
SparseLongArray     |  <Integer, Long>    | HashMap<Integer, Long>
LongSparseArray     |  <Long, Object>     | HashMap<Long, Object>
LongSparseLongArray |  <Long, Long>       | HashMap<Long, Long>
```

A `SparseIntArray` has only 3 data members to store key-value and size, on the other hand `HashMap` is quite heavy, compared to `Sparse` family though `HashMap`s are more suitable for large list.
- When to use SpareArray
  - Size of data is in range of hundreds mean ideal when size < 1000 
  - To avoid auto boxing for performance
  - Keys are obtained in ascending order

e.g 
```
SparseArray sparseArray = new SparseArray(); // API 1
sparseArray.put(1, “Value_1”);

SparseIntArray sparseIntArray = new SparseIntArray(); // API 1
sparseIntArray.put(1, 12);

SparseBooleanArray sparseBooleanArray = new SparseBooleanArray(); // API 1
sparseBooleanArray.put(1, true);

SparseLongArray sparseLongArray = new SparseLongArray(); // API 18
sparseLongArray.put(1, 5L);

```

> [ArrayMap](https://developer.android.com/reference/android/util/ArrayMap.html) can also be used as a replacement for `HashMap` for memory efficiency, but it's only available on API 19 and above.

### Objects class 
 `Objects` is a utility class which contains many `static` helpful null-safe or null-tolerant methods for
- Computing the hash code 
- Returning a string for an object
- Comparing two objects

##### Traditional approach of Generating hash code

  ```@Override
    public int hashCode() {
        int result = id != null ? id.hashCode() : 0;
        result = 31 * result + (number != null ? number.hashCode() : 0);
        result = 31 * result + (firstName != null ? firstName.hashCode() : 0);
        result = 31 * result + (lastName != null ? lastName.hashCode() : 0);
        result = 31 * result + (externalCustomerId != null ? externalCustomerId.hashCode() : 0);
        return result;
    }```

- With Object.hash
  
```@Override
public int hashCode() {
    return Objects.hash(number,firstName,,lastName,externalCustomerId);
}```

You can use other utility methods of `Objects` class though you have to be careful while using it because 
- The `hash` method accepts `Object...` varargs as parameter so if you have too many `primitives` fields and heavy use of `hashcode` calculations then `autoboxing` can cause more performance damage
- `Objects` class was introduced in API 19(kitkat) without any backward compatibility so cannot be used for older version.

### Application and Context 
The common pattern of inheritance in android is inheriting Application class or UI component classes.
In android, a single Application instance represents the process for an applications mean the application objects is kept in the memory as long as the application is alive.The lifespan of a custom application's object is equivalent to static data members which can be used to provide a single source of data(like pojo instance) where the context of application class is accessible.

All activities receives their context from application object and can also retrieve the application object using `(YourApplicationName)getApplicationContext()` to access the data members and method of customize application class. A customize application class is an easy way to crate a singleton to 
- Keep the heavy object like an image, accessible with context.
- To inject behavior in application life cycle via injection. e.g. dagger.
- To place boilerplate code for cloud services.

> The customize application class needs to be registered in the manifest as 

```
<application ...
		android:name = ".YourApplicaitonClassName">```

Context of an application and activity are different in terms of their lifespan so
- Use Activity's context to create objects which has short lifespan like adapter, dialogs etc.
- Every view in UI hierarchy has also access to context which can be fetched from view instance using `view.getContext()`, where the context is of host activity.

Generic : Generic provides the type-safety often used with collection framework like `ArrayList<T>`.

```
ArrayList list = new ArrayList(); // raw type list mean Object
ArrayList<String> list2 = new ArrayList<>(); String type list```

The [wildcards](https://docs.oracle.com/javase/tutorial/java/generics/wildcards.html) can also be used to add more flexibility

```
// To read objects from collection - use "extends"
int addListValues(List<? extends Number> numList) {
    int tot=0;
    for (Number num : numList) {
        tot = tot + num.intValue();
    }
    return tot;
}

// To add objects to collection - use "super"
void addMoreValues(List<? super Number> numList) {
    numList.add(11);
    numList.add(1.1);
}
    ```

- Generic plays a crucial role during parsing and conversion of json or string response to POJO objects
- Network libraries like retrofit uses many 3rd party parsing libraries which uses reflection to retrieve the object type information.
- Generic are only compile time, the type info is not available at runtime.

## Bonus
Access android developer site off-line : Yes, the official android developer site offline version comes along with the sdk which is available under `your_SDK_Location/Android/sdk/docs/index.html`.

By default the sdk location is `C:\Users\<User_Name>\AppData\Local\Android\sdk\docs\index.html` where `AppData` is a hidden directory.

# Android Studio Tools 

### Java-8 features 
Android studio 3.0 provides the inbuilt support for [java 8 libraries and language features](https://developer.android.com/studio/write/java8-support.html#supported_features) for API 24 without jack compiler support.

```
// lambda expression 
button.setOnClickListener(
        (v)-> Toast.makeText(context, "Click", Toast.LENGTH_SHORT).show()
);
```

To enable java-8 features, add following dependencies in `build.gradle` app module

```
android {
    //.. other dependencies 

    compileOptions {
        targetCompatibility 1.8
        sourceCompatibility 1.8
    }
}```

Android Studio 3.0 and later extends support for [`try-with-resources`](https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html) to all Android API levels

### Profiler
Profile is a collection of tools to measure the app's cpu, memory and network usage in real-time. You can measure the method execution duration, capturing heap dump, memory consumption and network transmission.

> Dalvik Debug Monitor Server also known as DDMS, has been deprecated so from android 3.0 and higher, Use Android Profiler to profile your app's CPU, memory, and network usage.

The native system processes can also be tracked using [`systrace`](https://developer.android.com/studio/command-line/systrace.html) command tool to track frame drops, CPU scheduling, disk activity and threads.

Profiler is available under "click View > Tool Windows > Android Profile" option


### Device File Explorer

Device file explorer allow you to inspect the device's file system and file transmission between device and computer.
The default DDMS has a [**quite popular bug**](https://issuetracker.google.com/issues/37123176) with android nougat and above emulators where the file system is not completely accessible, hence it has been replaced with Device File Explorer.

To open, click "View > Tool Windows > Device File Explorer"

### APK Analyzer
Often the build apk file can contain surprising behavior regarding the size, code shrinking and obfuscation.
To analyze your apk, just drag and drop apk into editor screen of studio  or 

Select "Build > Analyze APK" in the menu bar and then select your APK

APK Analyzer can accomplish the followings

- View the absolute and relative size of files in the APK, such as the DEX and Android resource files.
- Understand the composition of DEX files.
- Quickly view the final versions of files in the APK, such as the AndroidManifest.xml file.
- Perform a side-by-side comparison of two APKs.

### Layout-Inspector 
A dynamically created hierarchy of views in layout is often time consuming and requires many attempts to set the desired layout positioning constraints. Layout inspector captures a snapshot and store it as `.li` and display it along with the attributes values of views on screen as shown below 

![image](https://developer.android.com/studio/images/debug/layout-inspector-callouts_2x.png)

1. View Tree: The hierarchy of views in the layout
2. Screenshot: The device screenshot with visible boundaries for each view
3. Properties Table: The layout properties for the selected view

### Android Asset Studio 
Every android application requires basic drawable resources for icons, shape shifters for back press icon, an alpha channel background icon with white overlay, nine patch icons for flexible shapes for view etc, can simply be created with [Android Asset Studio](http://romannurik.github.io/AndroidAssetStudio/)

### Webp converter 
Webp is an image file format which provides lossy and transparency with better compression.

Lossy WebP images are supported in Android 4.0 (API level 14) and higher, and lossless and transparent WebP images are supported in Android 4.3 (API level 18) and higher

To create Webp images
- Select image and right click then click Convert to WebP
- Select Lossy(offers preview as well) or lossless encoding
- Click OK to apply compression

# 3rd party tools

  ### Image Loading Libraries
  There are some awesome libraries to ease-up the task of loading images from network, resources or from storage.
  - [Picasso](http://square.github.io/picasso/) is fully featured and most simplified image loading library to support recycling and download cancellation.
  - [Glide](https://github.com/bumptech/glide) is focused on smooth scrolling of image incentive list while supporting cache and Gifs.
  - [Fresco](https://github.com/facebook/fresco) is a image loading library to support webp and gif while supporting two level cache, memory and internal storage.

### Stetho 
Stetho is a debug bridge for Android applications from Facebook that integrates with the Chrome desktop browser's Developer Tools. With Stetho you can easily inspect your application, most notably the network traffic. It also allows you to easily inspect and edit SQLite databases and the shared preferences in your app. You should, however, make sure that Stetho is only enabled in the debug build and not in the release build variant.

Another alternative is Chuck which, although offering slightly more simplified functionality, is still useful for testers as the logs are displayed on the device, rather than in the more complicated connected Chrome browser setup that Stetho requires.

### EventBus 
[EventBus](https://github.com/greenrobot/EventBus) solves issues like communication between distant Activities, Fragments, Services, Threads etc.It is more suitable when you have the requirement to inform any component which resides deep in the hierarchy stack.

### LeakCanary 
[LeakCanary](https://github.com/square/leakcanary) helps to detect memory leaks mean the allocated space of memory which no longer referenced from scope and cannot be garbage collected. A common example is inner-class threads inside activities where user can go out of the app while the thread execution continues and expects to notify the activity with updated content at the end of its execution.

### Timber 
[Timer](https://github.com/JakeWharton/timber) is a logging library which is small, extensible and also provides utility on top of Android's normal Log class.

### Json 2 Pojo 
Writing POJO classes for json is time consuming and boring so there are couple of online tools like [jsonschema2pojo](http://www.jsonschema2pojo.org/) which can write those POJO classes in seconds, all you need to do is paste json structure and select the options and finally press OK.