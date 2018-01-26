This tutorial contains information about the android application architecture principals and to develop an application to incorporate the MVP architecture which improves facilitates the testing process, code re-usability, scalability and separation of UI components from app logic.

The conventional approach in anroid development follows the [MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) architecture where is described as below

### MVC
- Model : Stores the data and may contain the logic
- View  : A `View` is just a dummy placeholder to represent the output
- Controller : Controller, acts as a manager to provide response to user and interact with the user-interface.

An Activity or Fragment often act as a **Controller**, which contains all the implementation logic for `Sharedprference`, network callbacks, model and database interaction while including the activity lifecycle callback as well. 

Eventually the code grows abruptly and become more rigid to maintain, scale and refactor. MVC is so tightly coupled with Android APIs that the logic implementation cannot be tested with Unit testing or requires great deal of efforts to create a mock testing environment so this bring us to the next architecture approach i.e. clean architecture

### Clean Architecture
To separate the logic from the android APIs, A clean code architecture is adapted by the community to create layered architecture; separating model, logic and user interface in various layers.


![clean architecture](https://8thlight.com/blog/assets/posts/2012-08-13-the-clean-architecture/CleanArchitecture.jpg)

### Clean architecture helps to achieve code

- Separation of UI and logic : The UI and logic implementation is maintained in separately to incorporate changes without effecting other layers.
- Adaptive data source implementation : The data source be changed any time in future by leveraging [Repository Pattern](https://msdn.microsoft.com/en-us/library/ff649690.aspx).
- Testability : The [business logic](https://en.wikipedia.org/wiki/Business_logic) can be tested independent of UI, Database and any external source.
- Re-usability : The code structure can reused by adding the existing modules to new project.

For the detailed explanation on clean architecture, read this [article](https://8thlight.com/blog/uncle-bob/2012/08/13/the-clean-architecture.html).

# MVP
To address the architectural issues, MVP separates the application into three layers where all user events are delegate to Presenter

- **Model** : Model is responsible for managing the data, irrespective of the source of data. The responsibilities of model is to cache data, validate data source either network or local data source like sqlite, files etc. A model is often implemented via [Repository Pattern](https://msdn.microsoft.com/en-us/library/ff649690.aspx) to abstract away the internal implementation of data source.

- **View** : The only job of view is to display the content, instructed by the presenter. The View interface can be implemented by Activity or Fragment or it's subtypes. View keeps a reference to presenter to delegate the user event to presenter and perform UI oriented tasks like populating list, showing dialogs or updating content on screen.

- **Presenter** : The Presenter is an intermediate layer which bridge the communication gap between Model and View. As per the user event, passed by the view, the Presenter queries the model, transforms the data and pass the updated data to View to be displayed on the screen.

Note 
- In most cases, Presenter and View are bind to each other in a one-to-one relationship.
- Interfaces are define and implemented as a contract between View and Presenter communication.
- The View [aka](https://en.wikipedia.org/wiki/Aka) [passive view](https://martinfowler.com/eaaDev/PassiveScreen.html) should never update itself from model.
- The presenter handle all the business logic, model updation and view updation calls.
- To perform unit testing, presenter should avoid any usage of Android API because running a JVM test on local machine takes less time because it does not requires any UI interactions(Android framework components) or any emulator or physical device.

# Implementing Basic MVP Pattern

The project demonstrates working of an activity which receives and displays a list of movies from a network request (which is implemented locally for demonstration).

This is the visual representation of project structure

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/dd9bd57c-c7c1-4bbd-98d2-ea62aa3c3f15.PNG)
```
package com.example.pavneet_singh.mvptestingdemo.mvp.movieslist;
----------------------------------------------------------------
.model                  : POJO and Repository pattern 
.mvp.movielist          : MVP implementation for Movie List Screen
.util                   : To imitate a network response and provide helpful classes
```
1. Implementing Passive View : A view should be implemented as passive view. `MoviesListActivity` implements the `MoviesListContract.View` interface to expose the functionalities to other components like presenter.
```
// passive view
public class MoviesListActivity extends AppCompatActivity implements MoviesListContract.View{
```
Presenter will hold a reference to view and invoke methods exposed by View interface.The process to fetch the data from the data source, will be handled by the presenter. This **passive view** approach simplifies the testing phase with the help of [mocking object](https://en.wikipedia.org/wiki/Mock_object).

2. Android API Free Presenter : The presenter handle events from UI and use interface callback mechanism to communicate with long background threads and processes. The presenter implements the `MoviesListContract.Presenter`

 ```
public final class MoviePresenter implements MoviesListContract.Presenter  {
```

 The key point to make a testable presenter is to avoid usage of android API because often the common android API like `Sharedpreference`, database requires `context` which cannot be acquired without the involvement of activity lifecycle so to conduct the local junit tests via JVM on IDE the [mockito](http://site.mockito.org/) framework is used to create dummy object.

3. A Complete Contract for view and presenter

 The `View` and `Presenter` interfaces are combined into one single interface, known as `Contract` which will be used to establish a relationship between the concrete implementations of both `View` and `Presenter`.

 `MoviesListContract.java`

 ```
public interface MoviesListContract {

    // implemented by MoviesListActivity to provide concrete implementation
    interface View {
        void showProgress(); // display progress bar
        void hideProgress(); // hide progress bar
        void showMovieList(List<Movie> movies); // receive response to display
        void showLoadingError(String errMsg); // display error 

    }

    // implemented by MoviesPresenter to handle user event
    interface Presenter{
        void loadMoviewList();
        void dropView();
    }

    // implemented by MoviePresenter to receive response from asynchronous processes
    interface OnResponseCallback{
        void onResponse(List<Movie> movies);
        void onError(String errMsg);
    }
}
```

The `View` interface simply indicates the UI behavior and a concrete implementation of `Presenter` will manage the state of UI via invoking the methods of `View`.

A presenter has more user action specific methods to perform the result oriented operation on data. Presenter also use `OnResponseCallback` interfaces to establish a communication link with background tasks and process.

The reason to keep Presenter and View in a single interface is
- `View` is also a class in android so with this, we can name our interface as `View` without creating any confusion.
- Both View and Presenter functionalities is available at a single place which enhance the clarity.
- Presenter and View, both has specific behavior to serve a single entity (activity or fragment) so it makes more sense to keep the same things together.

### Important point while making a presenter
- Don't create a lifecycle in presenter otherwise your presenter will be tightly connected with the android component where most has their own differences. This makes presenter more dependent and highly rigid to modification.

- Introducing a method in presenter which accepts the view reference requires null checks at many places so avoid it if you can instead use constructor.

- Use cache mechanism like LRUCache to retain the data and keep the presenter stateless and let it recreate with the recreation of an activity or fragment.

4. View and Presenter In Connectivity
`MoviePresenter` will receive the reference of `View` i.e. `MoviesListActivity` via constructor along with the concrete implementation of `MovieRepo` interface which will contain the logic to retrieve data from data source (network, local database, files etc).

 Below the concrete implementation of presenter i.e. `MoviePresenter.java`

```
public final class MoviePresenter implements MoviesListContract.Presenter  {

    // to keep reference to view
    private MoviesListContract.View view;
    // Repository pattern, mclient holds reference to concrete data retrieval implementation
    private MovieRepo mclient;


    public MoviePresenter(MoviesListContract.View view,MovieRepo client) {
        this.view = view;
        mclient = client;
    }

    @Override
    public void dropView() {
        view = null;
    }

    // would be triggered by MovieListActivity
    @Override
    public void loadMoviewList() {
        view.showProgress();
        mclient.getMovieList(callback);
        // required for espresso UI testing
        // to wait till response occurred
        EspressoTestingIdlingResource.increment();
    }

    // callback mechanism , onResponse will be triggered with response
    // by simulatemovieclient(or your network or database process) and pass the response to view
    private final OnResponseCallback callback = new OnResponseCallback() {
        @Override
        public void onResponse(List<Movie> movies) {
            view.showMovieList(movies);
            view.hideProgress();
            EspressoTestingIdlingResource.decrement();
        }

        @Override
        public void onError(String errMsg) {
            view.hideProgress();
            view.showLoadingError(errMsg);
            EspressoTestingIdlingResource.decrement();
        }
    };
}
```
`loadMoviewList` method would be triggered by `MoviesListActivity` when the user perform action (swipe down on screen) then a progress bar will be displayed on screen until response is received and delivered to activity by `view.showMovieList(movies);` method to display list.

```
public class MoviesListActivity extends AppCompatActivity implements MoviesListContract.View{

    private RecyclerView recyclerView;
    private MoviesAdapter moviesAdapter;
    private SwipeRefreshLayout swipeLayout;
    private MoviesListContract.Presenter presenter;
    private TextView tv_empty_msg;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_movie_list);
        initViews();
    }

    private void initViews(){
        presenter = new MoviePresenter(this, new SimulateMovieClient());
        recyclerView = (RecyclerView) findViewById(R.id.movies_recycler_list); // list
        tv_empty_msg = (TextView)findViewById(R.id.swipe_msg_tv); // empty message
        recyclerView.setLayoutManager(new LinearLayoutManager(this)); // for vertical liner list
        moviesAdapter = new MoviesAdapter(this);
        recyclerView.setAdapter(moviesAdapter);
        swipeLayout = (SwipeRefreshLayout) findViewById(R.id.swipe_container);
        swipeLayout.setOnRefreshListener(listener);
        swipeLayout.setColorSchemeColors( // colors for progress dialog
                ContextCompat.getColor(MoviesListActivity.this, R.color.colorPrimary),
                ContextCompat.getColor(MoviesListActivity.this, R.color.colorAccent),
                ContextCompat.getColor(MoviesListActivity.this, android.R.color.holo_green_light)
        );
    }

    private final OnRefreshListener listener = new OnRefreshListener() {
        @Override
        public void onRefresh() {
            presenter.loadMoviewList();
        }
    };

    // for future, to show progress
    @Override
    public void showProgress() {
        swipeLayout.setRefreshing(true);
    }

    @Override
    public void hideProgress() {
        swipeLayout.setRefreshing(false);
    }

    // toggle the visibility of empty textview or list
    // display list only when response it not empty
    private void showORHideListView(boolean flag){
        if (flag){
            tv_empty_msg.setVisibility(View.GONE);
            recyclerView.setVisibility(View.VISIBLE);
        }else {
            tv_empty_msg.setVisibility(View.VISIBLE);
            recyclerView.setVisibility(View.GONE);
        }
    }

    @Override
    public void showMovieList(List<Movie> movies) {
        if (!movies.isEmpty()){
            moviesAdapter.setList(movies);
            showORHideListView(true);
        }
    }

    @Override
    public void showLoadingError(String errMsg) {
        hideProgressAndShowErr(errMsg);
        showORHideListView(false);
    }

    private void hideProgressAndShowErr(String msg){
        tv_empty_msg.setVisibility(View.VISIBLE);
        Toast.makeText(this, msg, Toast.LENGTH_SHORT).show();
        showORHideListView(false);
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        presenter.dropView();
    }
}
```
The above implementation of View creates a reference of presenter and pass its reference using `this` along with the `SimulateMovieClient` instance, containing data retrieval logic.

`activity_movie_list.xml`
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/tasksContainer"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scrollbars="vertical">

    <TextView
        android:text="@string/No_Data_MSG"
        android:gravity="center|center_horizontal"
        android:id="@+id/swipe_msg_tv"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/swipe_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >

        <android.support.v7.widget.RecyclerView
            android:id="@+id/movies_recycler_list"
            android:visibility="gone"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>

    </android.support.v4.widget.SwipeRefreshLayout>
</RelativeLayout>
```
This is the adapter code to create list items
`MoviesAdapter.java`
```
public class MoviesAdapter extends RecyclerView.Adapter<MoviesAdapter.MovieHolder> {

    private final Context context;
    private final List<Movie> movStrings = new ArrayList<>();
    private static final String TAG = "MoviesAdapter";

    public MoviesAdapter(Context context) {
        this.context = context;
    }

    @Override
    public MovieHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context).inflate(R.layout.item_movie_model,parent,false);
        return new MovieHolder(view);
    }

    @Override
    public void onBindViewHolder(MovieHolder holder, final int position) {
        Log.e(TAG, "onBindViewHolder: "+position);
        holder.movieTitle.setText(movStrings.get(position).getTitle());
        holder.date.setText(Utility.convertMinutesToDuration(movStrings.get(position).getDurationinMinutes()));
        holder.rating.setText(Double.toString(movStrings.get(position).getRating()));
        holder.moviesLayout.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Toast.makeText(context, movStrings.get(position).toString(), Toast.LENGTH_SHORT).show();
            }
        });
    }

    public void setList(List<Movie> list){
        movStrings.clear();
        movStrings.addAll(list);
        notifyDataSetChanged();
        Log.e(TAG, "onNext: "+movStrings.size() );
    }


    @Override
    public int getItemCount() {
        return movStrings.size();
    }

    public static class MovieHolder extends RecyclerView.ViewHolder {
        TextView movieTitle;
        TextView date;
        TextView view;
        TextView rating;
        LinearLayout moviesLayout;

        public MovieHolder(View v) {
            super(v);
            moviesLayout = (LinearLayout) v.findViewById(R.id.movies_layout);
            movieTitle = (TextView) v.findViewById(R.id.title);
            date = (TextView) v.findViewById(R.id.date);
            rating = (TextView) v.findViewById(R.id.rating);
        }
    }
}
```
The layout item file `item_movie_model.xml`
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/movies_layout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:background="?android:attr/selectableItemBackground"
    android:gravity="center_vertical"
    android:minHeight="72dp"
    android:orientation="horizontal"
    android:padding="16dp">

    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:orientation="vertical">

        <TextView

            android:id="@+id/title"
            tools:text="Avengers"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="top"
            android:paddingRight="16dp"
            android:textStyle="bold"
            android:textColor="@android:color/black"
            android:textSize="16sp" />


        <TextView
            android:id="@+id/date"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:maxLines="1"
            android:paddingRight="16dp"
            android:textColor="@color/greyLight" />

    </LinearLayout>

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="35dp"
        android:orientation="horizontal">

        <ImageView
            android:id="@+id/rating_image"
            android:layout_width="15dp"
            android:layout_height="15dp"
            android:scaleType="centerCrop"
            android:src="@drawable/star"
            android:tint="@color/colorAccent" />


        <TextView
            android:id="@+id/rating"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginLeft="8dp"
            tools:text="5.0"
            android:layout_marginStart="8dp" />
    </LinearLayout>

</LinearLayout>
```

`MovieRepo` is an interface which is implemented by `SimulateMovieClient` to imitate the background thread behavior to retrieve data as shown below 

`MovieRepo.java`
```
public interface MovieRepo {
    void getMovieList(MoviesListContract.OnResponseCallback callback);
}
```
`SimulateMovieClient` contains a thread implementation which will give the response with the delay of `1500` milliseconds.
```
public final class SimulateMovieClient implements MovieRepo{

    static final Random RANDOM = new Random();


    public void getMovieList(final MoviesListContract.OnResponseCallback callback)  {
        // To imitate network request delay
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                ArrayList<Movie> movies = new ArrayList<>();
                try {
                    movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "IT", Utility.convertStringToDate("2017-10-8"), 7.6, 127, Type.HORROR));
                    // add more Movie object in list
                    callback.onResponse(movies);
                } catch (ParseException e) {
                    callback.onError("Error from network");
                }
            }
        }, 1500);

    }
}
```
Final visual output

![mvp demo](https://raw.githubusercontent.com/Pavneet-Sing/MVP-with-Testing/master/screenshots/demo.gif)

This concludes the introductory implementation of MVP and the complete code structure is available as [github repository](https://github.com/Pavneet-Sing/MVP-with-Testing) to play with.

# Testing MVP with espresso and junit
The current section is divided into two parts to test the MVP UI using instrumentation test and business logic using junit test. This section is focused on testing the project specific logic rather than testing framework working.

### Testing MoviePresenter
To test the business logic of presenter, we required to mock the UI and repository components so the primary focused will be on the testing the presenter and its interaction with other components like view and repository instance. To apply mocking, we will use mockito framework.

### Setup
The unit test classes created under `module-name/src/test/java/` package that run on local JVM. These test are quick in nature and has no android API interaction.

### Create Test

To create either a local unit test or an instrumented test, you can create a new test for a specific class or method by following these steps:

1. Open the Java file containing the code you want to test.
2. Click the class or method you want to test, then press Ctrl+Shift+T (⇧⌘T).
3. In the menu that appears, click Create New Test.
4. In the Create Test dialog, edit any fields and select any methods to generate, and then click OK.
5. In the Choose Destination Directory dialog, click the source set corresponding to the type of test you want to create: androidTest for an instrumented test or test for a local unit test. Then click OK.

### Test Presenter Functionalities
- Receive user event
- Initiate background thread to retrieve response
- Capture the response from background thread using `ArgumentCaptor`
- Verify the behavior of mocked view when response is received

### Explanation step by step
1. Mock instances like `view` and `movieRepo` using `@Mock` annotation to test the complete functionality of presenter.
2. Methods with `@Before` is used to setup the testing environment and dependencies, in this case, setup the mocking framework and initialize presenter using mocked instances.
3. Create an `argumentCaptor` instance to capture the callback interface reference and later, used to provide response to mocked view instance.
    - Create and pass a dummy list as response.
4. Verify the successful method invocation of `view` instance

`MoviePresenterTest.java`
```
public class MoviePresenterTest {

    private static final Random RANDOM = new Random();

    @Mock // 1
    private MoviesListContract.View view;

    @Mock // 1
    private MovieRepo movieRepo;

    private MoviePresenter presenter;

    @Captor // 3
    private ArgumentCaptor<MoviesListContract.OnResponseCallback> argumentCaptor;

    @Before // 2
    public void setUp(){
        // A convenient way to inject mocks by using the @Mock annotation in Mockito.
        //  For mock injections , initMocks method needs to be called.
        MockitoAnnotations.initMocks(this);

        // get the presenter reference and bind with view for testing
        presenter = new MoviePresenter(view,movieRepo);


    }

    @Test
    public void loadMoviewList() throws Exception {
        presenter.loadMoviewList();
        verify(movieRepo,times(1)).getMovieList(argumentCaptor.capture());
        argumentCaptor.getValue().onResponse(getList());
        verify(view,times(1)).hideProgress();
        ArgumentCaptor<List> entityArgumentCaptor = ArgumentCaptor.forClass(List.class);
        verify(view).showMovieList(entityArgumentCaptor.capture());

        assertTrue(entityArgumentCaptor.getValue().size() == 10);
    }

    @Test
    public void OnError() throws Exception {
        presenter.loadMoviewList();
        verify(movieRepo,times(1)).getMovieList(argumentCaptor.capture());
        argumentCaptor.getValue().onError("Error");
        ArgumentCaptor<String> argumentCaptor = ArgumentCaptor.forClass(String.class);
        verify(view,times(1)).showLoadingError(argumentCaptor.capture()); // 3
        verify(view).showLoadingError(argumentCaptor.getValue()); // 4
    }

    private List<Movie> getList() {
        ArrayList<Movie> movies = new ArrayList<>();
        try {
            movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "IT", Utility.convertStringToDate("2017-10-8"), 7.6, 127, Movie.Type.HORROR));
            movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "Jumanji 2", Utility.convertStringToDate("2018-12-20"), 8.3, 111, Movie.Type.ACTION));
            movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "The Dark Knight", Utility.convertStringToDate("2008-07-08"), 9.0, 152, Movie.Type.ACTION));
            movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "Inception", Utility.convertStringToDate("2010-07-16"), 8.8, 148, Movie.Type.ACTION));
            movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "The Green Mile", Utility.convertStringToDate("1999-12-10"), 8.5, 189, Movie.Type.DRAMA));
            movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "Transcendence", Utility.convertStringToDate("2014-04-18"), 6.3, 120, Movie.Type.FICTION));
            movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "Saving Private Ryan", Utility.convertStringToDate("1998-07-24"), 8.6, 169, Movie.Type.DRAMA));
            movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "Whiplash", Utility.convertStringToDate("2015-02-20"), 8.5, 117, Movie.Type.DRAMA));
            movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "The Raid", Utility.convertStringToDate("2011-04-13"), 7.6, 111, Movie.Type.ACTION));
            movies.add(new Movie(RANDOM.nextInt(Integer.MAX_VALUE), "Burnt", Utility.convertStringToDate("2015-10-30"), 6.6, 111, Movie.Type.DRAMA));
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return movies;
    }
}
```

### Testing MovieListActivity
To perform automated testing on UI, we will use espresso framework is used to perform instrumentation test.
Espresso is not aware of any asynchronous operation or background thread so in order to make Espresso aware of your app's long-running operations, we will use [CountingIdlingResource](https://developer.android.com/reference/android/support/test/espresso/idling/CountingIdlingResource.html).

`MoviePresent` must increment the idling resource to make espresso aware of the idle time as shown below
```
    @Override
    public void loadMoviewList() {
        view.showProgress();
        mclient.getMovieList(callback);

        // required for espresso UI testing
        EspressoTestingIdlingResource.increment();
    }
```
and decrement it when the response is received.

The idling resource is accessed in a static manner using a utility class `EspressoTestingIdlingResource`

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
> Check the presenter code for complete implementation in [repository](https://github.com/Pavneet-Sing/MVP-with-Testing/blob/master/app/src/main/java/com/example/pavneet_singh/mvptestingdemo/mvp/movieslist/MoviePresenter.java)

### Explanation step by step
1. Register the idling resource in `@Before` annotated method to make espresso aware of the asynchronous delay.
2. Write a test to verify the visibility of default empty message using.
3. Perform user action to initiate data retrieval task.
4. Check the visibility of list on screen.
5. Click on particular list item.
6. Verify the toast message, containing details of movie.

`MovieListTest.java`
```
@RunWith(AndroidJUnit4.class)
public class MovieListTest {


    @Rule
    public ActivityTestRule<MoviesListActivity> activityTestRule =
            new ActivityTestRule<>(MoviesListActivity.class);

    /**
     * Register IdlingResource resource to tell Espresso when your app is in an
     * idle state. This helps Espresso to synchronize test actions.
     */

    @Before // 1
    public void registerIdlingResource() {
        IdlingRegistry.getInstance().register(EspressoTestingIdlingResource.getIdlingResource());
    }

    /**
     * Unregister your Idling Resource so it can be garbage collected and does not leak any memory.
     */
    @After
    public void unregisterIdlingResource() {
        IdlingRegistry.getInstance().unregister(EspressoTestingIdlingResource.getIdlingResource());
    }

    @Test // 2
    public void checkStaticView() {
        // verify default empty text message
        onView(withId(R.id.swipe_msg_tv)).check(matches(isDisplayed()));
        //                                |--------------------------|
        //|----------------------------| find a view | using swipe_msg_tv id
        //check visibility of view on screen <-------|
    }

    @Test
    public void checkRecyclerViewVisibility() {
        // 3. perform swipe
        onView(withId(R.id.swipe_msg_tv)).check(matches(isDisplayed()));
        onView(withId(R.id.swipe_container)).perform(swipeDown());

        // verify swipe is displayed
        onView(withId(R.id.swipe_msg_tv)).check(matches(not(isDisplayed())));

        // 4 verify recycler view is displayed
        onView(withId(R.id.movies_recycler_list)).check(matches(isDisplayed()));
        // 5 perform click on item at 0th position
        onView(withId(R.id.movies_recycler_list))
                .perform(RecyclerViewActions.actionOnItemAtPosition(0, click()));

        // 6 verify the toast text
        MoviesListActivity activity = activityTestRule.getActivity();
        onView(withText("Title : 'IT' Rating : '7.6'")).
                inRoot(withDecorView(not(is(activity.getWindow().getDecorView())))).
                check(matches(isDisplayed()));
    }
}
```
# Conclusion
MVP provides remarkable support for testing, scalability and [SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)) principals. One downside can be the boilerplate code but eventually it's worth your time to create complex applications and decoupled modules.

# References
[Dagger 2](https://github.com/google/dagger) : To supply the dependencies to modules like presenter rather than receiving it from constructor. This make our module more adaptive to future changes.

[RxJava](https://github.com/ReactiveX/RxJava) : For reactive programming

[Android Architecture Blueprints](https://github.com/googlesamples/android-architecture) : Google repository to demonstrate working of various android architecture.


-------
I hope you find this article helpful. If you have any feedback, kindly feel free to comment below, i would be glad to talk to you.