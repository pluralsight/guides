Data persistence is one of the basic requirement of most applications. SQLite, an open-source library is a part of Android framework to persist data for applications. The implementation of sqlite requires lot of boilerplate code which has its own drawbacks :
  * Syntax errors in queries
  * No compile time error detection (Time consuming)
  * Parsing is required to convert data to [pojo](https://en.wikipedia.org/wiki/Plain_old_Java_object) objects

These issues are quite common in Q&A forms so this gave rise to popular No-SQL database like Realm, GreenDAO and ROOM where Room is a persistent library to abstract away the most of the sqlite code using annotations.


#### The prime goals of this tutorials is to provide an exposure on
  * Storage options in android
  * SQL vs No-SQL
  * Developing a notepad app using Room library

# Introduction to android storage mechanism
#### Android support several storage mechanisms to store data, as mentioned below
1. Key-Value pairs : [SharedPreferences](https://developer.android.com/reference/android/content/SharedPreferences.html), An android framework API, which stores key-values pairs in a XML file under protected file system.

    - Data stored via SharedPreference can only be accessed within the app.
    - Can only store boolean, int, long, String and [Set](https://developer.android.com/reference/java/util/Set.html) of String.

2. Internal and External storage : Applications can store text or [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) files, images, videos in phone memory or inside **public directories**(kitkat or above) under SD card storage.

    - To access phone or external storage, Applications requires to implement [Requesting Permissions Model](https://developer.android.com/training/permissions/requesting.html) for marshmallow and above.
    - Applications can access all directories under external storage on API's below KitKat.
    - Accessible to other applications, No protection.

3. SQLite : [SQLite](https://sqlite.org/index.html) is a light weight [relational database](https://en.wikipedia.org/wiki/Relational_database), embedded into android OS. The [database schema](https://en.wikipedia.org/wiki/Database_schema) is mapped to tables and integrity constraints.
    - Runtime memory consumption is merely 250 [kbytes](https://en.wikipedia.org/wiki/Kilobyte).
    - Supported data types are INTEGER, READ (decimals), TEXT, [BLOB](https://developer.android.com/reference/java/sql/Blob.html)(mostly used to store images but don't do it) and null.

4. NoSQL : `NoSQL` simply means Objects or Documents. Instead of strong the data in tabular form, The data is stored as [POJO Object](https://en.wikipedia.org/wiki/Plain_old_Java_object) which is extremely suitable for semi-structured or un-structured data when there is no fix [Database Schema](https://en.wikipedia.org/wiki/Database_schema).

Features |SQL | No-SQL
----|-------
Data Stored|    In Tabular form (RDBMS) | POJO objects or Documents
Data Manipulation |  via DML,DDL | via provided API's
Structure  |    RDBMS                           |      key-value pairs
Schema  |    Fixed schema                       |      dynamic, records can be added on the fly
Scalable  |    RDBMS                           |      key-value pairs
Android Support  |  SQLite          |      Room(semi-sql), GreenDAO, Realm


Rooms : Room library acts as an abstract layer for underlying SQLite database, means Room annotations are used 
* To Database and Entities where entities are POJO classes representing table structure
* To specify operation for retrieval, updation and deletion
* To add constraints like foreign keys etc
* Support for LiveData

### There are 3 major components in Room
1. Entity : A class annotated with [`@Entity`](https://developer.android.com/reference/android/arch/persistence/room/Entity.html) annotation is mapped to a table in database. Every entity is persisted in its own table and every field in class represents the column name.
  - `tableName` attribute is used to define the name of the table
  - Every entity class must have at-least one [Primary Key](http://wiki.c2.com/?PrimaryKey) field, annotated with @PrimaryKey
  - Fields in entity class can be annotated with `@ColumnInfo(name = “name_of_column”)` annotation to give specific column names

2. DAO : [Data Access Object](https://developer.android.com/reference/android/arch/persistence/room/Dao.html) is either be an interface or an abstract class annotated with `@Doa` annotation, containing all the methods to define the operations to be performed on data. The methods can be annotated with 
    - [@Query](https://developer.android.com/reference/android/arch/persistence/room/Query.html) to retrieve data from database
    - [@Insert](https://developer.android.com/reference/android/arch/persistence/room/Insert.html) to insert data into database   
    - [@Delete](https://developer.android.com/reference/android/arch/persistence/room/Delete.html) to delete data from database
    - [@Update](https://developer.android.com/reference/android/arch/persistence/room/Update.html) to update data in database
    
      > The result of SQLite queries are composed into `cursor` object, DAO methods abstract the conversion of cursor to Entity objects and vice-versa. 
      
3. Database : Database is a container for tables. An abstract class annotated with [`@Database`](https://developer.android.com/reference/android/arch/persistence/room/Database.html) annotation is used to create a database with given name along with database version.
    - `version = intValueForDBVersion` is used to define the database version
    - `entities = {EntityClassOne.class, ....}` is used to define list of entities for database


![Room Architecture](https://raw.githubusercontent.com/pluralsight/guides/master/images/3fefe389-9a9b-467c-bcef-793da8449f05.png)

# Notes App Structure
  Hope, You are tempted enough to try room library, so let's try it. Notes App will allow the user to 
- Create and Save notes in database
- Update and Delete notes

# Let's do it!
  In this section, we will create a demo application to get started with a database oriented application, and conceptually, code implementation can further be used to create alarm, schedule, sms based applications.

  > While creating project, choose "Basic Activity"
  
  > All the code is available at [Github Repo](https://github.com/Pavneet-Sing/RoomDemo)

### Add Maven Repository
- Open `build.gradle` project and add following maven dependency

  ```
  allprojects {
      repositories {
          google()
          jcenter()
          maven {
              url "https://maven.google.com"
          }
      }
  }```

  > [Google Maven Repository](https://androidstudio.googleblog.com/2017/05/android-studio-30-canary-1-sdk-updates.html) provides all the updated support libraries, which will be downloaded by gradle instead of downloading from SDK Manager.

- Add Room dependency in `build.gradle`(Module:app), inside dependency block
 ```
 dependencies {
    //... other dependencies

    // Room dependencies
      compile 'android.arch.persistence.room:runtime:1.0.0'
      annotationProcessor 'android.arch.persistence.room:compiler:1.0.0'
  }
  ```

### Create Entity

Before creating a database, Let's create an Entity, named as `Note` and later, Objects of this class will be added to database.

- Create a `class` named `Note`
- Add @Entity annotation on the class
- Add id, content and title fields
- **Importantly** mark at-least one field with `@PrimaryKey` annotation
- Use `alt+insert` to implement constructor, getter and setter`override` and optionally override `equals` or `tostring`

```
package com.example.pavneet_singh.roomdemo.notedb.model;

import android.arch.persistence.room.Entity;
import android.arch.persistence.room.PrimaryKey;

@Entity
public class Note {

    @PrimaryKey(autoGenerate = true)
    private int note_id;

    @ColumnInfo(name = "note_content") // column name will be "note_content" instead of "content" in table
    private String content;

    private String title;

    private 

    public Note(int note_id, String content, String title) {
        this.note_id = note_id;
        this.content = content;
        this.title = title;
    }

    public int getNote_id() {
        return note_id;
    }

    public void setNote_id(int note_id) {
        this.note_id = note_id;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Note)) return false;

        Note note = (Note) o;

        if (note_id != note.note_id) return false;
        return title != null ? title.equals(note.title) : note.title == null;
    }

    

    @Override
    public int hashCode() {
        int result = note_id;
        result = 31 * result + (title != null ? title.hashCode() : 0);
        return result;
    }

    @Override
    public String toString() {
        return "Note{" +
                "note_id=" + note_id +
                ", content='" + content + '\'' +
                ", title='" + title + '\'' +
                '}';
    }
}
```

### Creating DAO's 
  DAO define all methods to access database, annotated with `@Dao` annotation. It acts as a contract to perform CRUD operations on data in database.
  - Create an interface, marked with `@Dao` annotation
  - Add methods for CURD operations

```java
package com.example.pavneet_singh.roomdemo.notedb.dao;

import android.arch.persistence.room.Dao;
import android.arch.persistence.room.Delete;
import android.arch.persistence.room.Insert;
import android.arch.persistence.room.Query;
import android.arch.persistence.room.Update;

import com.example.pavneet_singh.roomdemo.notedb.model.Note;
import com.example.pavneet_singh.roomdemo.util.Constants;

import java.util.List;

@Dao
public interface NoteDao {
  @Query("SELECT * FROM user "+ Constants.TABLE_NAME_NOTE)
  List<Note> getAll();


  /*
  * Insert the object in database
  * @param note, object to be inserted
  */
  @Insert
  void insert(Note note);

  /*
  * update the object in database
  * @param note, object to be updated
  */
  @Update
  void update(Note repos);

  /*
  * delete the object from database
  * @param note, object to be deleted
  */
  @Delete
  void delete(Note note);

  /*
  * delete list of objects from database
  * @param note, array of objects to be deleted
  */
  @Delete
  void delete(Note... note);      // Note... is varargs, here note is an array

}```


### Create Database
  Now, we have table defined as `Entity` and CRUD methods defined via `NoteDao` so the last piece of the database puzzle is the database itself.

- Create an abstract class `NoteDatabse` which extends `RoomDatabase`
- Add version and entities to database as `@Database(entities = {Note.class}, version = 1)`
- Add abstract methods to all DAO's where the returned DAO object is constructed by Room for database interactions

  > Version number is changed to update the database structure, when required in future updates

  > Database name must ends with `.db` extension

  > Creating instance of database is quite costly so we will apply [Singlton Pattern](https://en.wikipedia.org/wiki/Singleton_pattern) to create and use only single instance for every database access.

```

package com.example.pavneet_singh.roomdemo.notedb;

import android.arch.persistence.room.Database;
import android.arch.persistence.room.Room;
import android.arch.persistence.room.RoomDatabase;
import android.content.Context;

import com.example.pavneet_singh.roomdemo.notedb.dao.NoteDao;
import com.example.pavneet_singh.roomdemo.util.Constants;

@Database(entities = { Note.class }, version = 1)
public abstract class NoteDatabase extends RoomDatabase {

  public abstract NoteDao getNoteDao();


  private static NoteDatabase noteDB;

  public static NoteDatabase getInstance(Context context) {
      if (null == noteDB) {
          noteDB = buildDatabaseInstance(context);
      }
      return noteDB;
  }

  private static NoteDatabase buildDatabaseInstance(Context context) {
      return Room.databaseBuilder(context,
              NoteDatabase.class,
              Constants.DB_NAME)
              .allowMainThreadQueries().build();
  }

  public  void cleanUp(){
      noteDB = null;
  }

}
```

> Room does not allow code execution on Main thread, `allowMainThreadQueries` is used to allow the execution

> Do not do this in real app, this is just for demonstration instead use AsyncTask (or handler, rxjava)

> `AddNoteActivity.java` snippet demonstrates, how to use AsyncTask

# Implement Database Interactions
  The below snippet will demonstrate the working of insert, update and delete functionality using Room database

### Add Notes 
In `AddNoteActivity.java`

- Initialize views and database 
- Fetch data from `EditText` and create `Note` object
- Insert previously created `Note` object into database

```
package com.example.pavneet_singh.roomdemo;

import android.content.Intent;
import android.os.AsyncTask;
import android.os.Bundle;
import android.support.design.widget.TextInputEditText;
import android.support.v7.app.AppCompatActivity;
import android.view.View;
import android.widget.Button;

import com.example.pavneet_singh.roomdemo.notedb.NoteDatabase;
import com.example.pavneet_singh.roomdemo.notedb.model.Note;

import java.lang.ref.WeakReference;

public class AddNoteActivity extends AppCompatActivity {

  private TextInputEditText et_title,et_content;
  private NoteDatabase noteDatabase;
  private Note note;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_add_note);
      et_title = findViewById(R.id.et_title);
      et_content = findViewById(R.id.et_content);
      noteDatabase = NoteDatabase.getInstance(AddNoteActivity.this);
      Button button = findViewById(R.id.but_save);

      button.setOnClickListener(new View.OnClickListener() {
          @Override
          public void onClick(View view) {
            // fetch data and create note object
                note = new Note(et_content.getText().toString(),
                        et_title.getText().toString());

                // create worker thread to insert data into database
                new InsertTask(AddNoteActivity.this,note).execute();
          }
      });
  }

  private void setResult(Note note, int flag){
      setResult(flag,new Intent().putExtra("note",note));
      finish();
  }

  private static class InsertTask extends AsyncTask<Void,Void,Boolean> {

      private WeakReference<AddNoteActivity> activityReference;
      private Note note;

      // only retain a weak reference to the activity
      InsertTask(AddNoteActivity context, Note note) {
          activityReference = new WeakReference<>(context);
          this.note = note;
      }

      // doInBackground methods runs on a worker thread
      @Override
      protected Boolean doInBackground(Void... objs) {
          activityReference.get().noteDatabase.getNoteDao().insertNote(note);
          return true;
      }

        // onPostExecute runs on main thread
      @Override
      protected void onPostExecute(Boolean bool) {
          if (bool){
              activityReference.get().setResult(note,1);
          }
      }
  }
  
}```


`activity_add_note.xml`


```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:app="http://schemas.android.com/apk/res-auto"
  xmlns:tools="http://schemas.android.com/tools"
  android:layout_width="match_parent"
  android:layout_height="match_parent"
  android:layout_margin="4dp"
  tools:context="com.example.pavneet_singh.roomdemo.AddNoteActivity"
  android:orientation="vertical">

  <android.support.design.widget.TextInputLayout
      android:id="@+id/txtInput"
      android:layout_width="match_parent"
      android:layout_height="wrap_content">
      <android.support.design.widget.TextInputEditText
          android:inputType="text"
          android:id="@+id/et_title"
          android:hint="Title"
          android:singleLine="true"
          android:layout_width="match_parent"
          android:layout_height="wrap_content" />
  </android.support.design.widget.TextInputLayout>

  <LinearLayout
      android:layout_below="@+id/txtInput"
      android:orientation="vertical"
      android:layout_width="match_parent"
      android:layout_height="match_parent">

      <android.support.design.widget.TextInputLayout
          android:id="@+id/txtInput2"
          android:layout_width="match_parent"
          android:layout_weight="1"
          android:layout_height="0dp">
          <android.support.design.widget.TextInputEditText
              android:id="@+id/et_content"
              android:hint="Content"
              android:gravity="top"
              android:singleLine="false"
              android:inputType="textMultiLine|textNoSuggestions"
              android:layout_width="match_parent"
              android:layout_height="match_parent" />
      </android.support.design.widget.TextInputLayout>

      <Button
          android:id="@+id/but_save"
          android:text="Save"
          android:layout_gravity="center"
          android:layout_width="wrap_content"
          android:layout_height="wrap_content" />

  </LinearLayout>

</RelativeLayout>```

### Retrieve And Display NoteList
- Initialize Room database instance and fetch all note objects as List
- To display list, we will use RecyclerView
- Pass the list of notes to adapter and link adapter with the RecyclerView

`NoteListActivity.java`

```
package com.example.pavneet_singh.roomdemo;

import android.content.DialogInterface;
import android.content.Intent;
import android.os.AsyncTask;
import android.os.Bundle;
import android.support.design.widget.FloatingActionButton;
import android.support.v7.app.AlertDialog;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.support.v7.widget.Toolbar;
import android.view.View;
import android.widget.TextView;

import com.example.pavneet_singh.roomdemo.adapter.NotesAdapter;
import com.example.pavneet_singh.roomdemo.notedb.NoteDatabase;
import com.example.pavneet_singh.roomdemo.notedb.model.Note;

import java.lang.ref.WeakReference;
import java.util.List;

public class NoteListActivity extends AppCompatActivity implements NotesAdapter.OnNoteItemClick{

  private TextView textViewMsg;
  private RecyclerView recyclerView;
  private NoteDatabase noteDatabase;
  private List<Note> notes;
  private NotesAdapter notesAdapter;
  private int pos;


  @Override
  protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      setContentView(R.layout.activity_main);
      initializeVies();
      displayList();
  }

  private void displayList(){
    // initialize database instance
      noteDatabase = NoteDatabase.getInstance(NoteListActivity.this);
      // fetch list of notes in background thread
      new RetrieveTask(this).execute();
  }

   private static class RetrieveTask extends AsyncTask<Void,Void,List<Note>>{

      private WeakReference<NoteListActivity> activityReference;

      // only retain a weak reference to the activity
      RetrieveTask(NoteListActivity context) {
          activityReference = new WeakReference<>(context);
      }

      @Override
      protected List<Note> doInBackground(Void... voids) {
          if (activityReference.get()!=null)
              return activityReference.get().noteDatabase.getNoteDao().getNotes();
          else
              return null;
      }

      @Override
      protected void onPostExecute(List<Note> notes) {
          if (notes!=null && notes.size()>0 ){
              activityReference.get().notes = notes;

              // hides empty text view
              activityReference.get().textViewMsg.setVisibility(View.GONE);

              // create and set the adapter on RecyclerView instance to display list
              activityReference.get().notesAdapter = new NotesAdapter(notes,activityReference.get());
              activityReference.get().recyclerView.setAdapter(activityReference.get().notesAdapter);
          }
      }
  }

  private void initializeVies(){
      Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
      setSupportActionBar(toolbar);
      textViewMsg =  (TextView) findViewById(R.id.tv__empty);

      // Action button to add note
      FloatingActionButton fab = (FloatingActionButton) findViewById(R.id.fab);
      fab.setOnClickListener(listener);
      recyclerView = findViewById(R.id.recycler_view);
      recyclerView.setLayoutManager(new LinearLayoutManager(NoteListActivity.this));
  }

}```

 ### Update Note
 To update note object, the content of already created object needs to be updated while keeping the **same primary key**

- Receive the Note Object from NoteListActivity and display its content on screen
- Once the data has been updated by user then fetch data from EditText and update the note object
- Invoke update method of NoteDao with room database instance and pass the updated note object

 > In project, AddNoteActivity is being reused to perform update and insertion

  ``` 
  public class AddNoteActivity extends AppCompatActivity {

      private TextInputEditText et_title,et_content;
      private NoteDatabase noteDatabase;
      private Note note;
      private boolean update;

      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_add_note);
          et_title = findViewById(R.id.et_title);
          et_content = findViewById(R.id.et_content);
          noteDatabase = NoteDatabase.getInstance(AddNoteActivity.this);
          Button button = findViewById(R.id.but_save);
          if ( (note = (Note) getIntent().getSerializableExtra("note"))!=null ){
              getSupportActionBar().setTitle("Update Note");
              update = true;
              button.setText("Update");
              et_title.setText(note.getTitle());
              et_content.setText(note.getContent());
          }

          button.setOnClickListener(new View.OnClickListener() {
              @Override
              public void onClick(View view) {
              note.setContent(et_content.getText().toString());
              note.setTitle(et_title.getText().toString());
              noteDatabase.getNoteDao().updateNote(note);
              }
          });
      }
  }```

### Delete Note
  To delete note object, just invoke `delete` method and pass the note object for deletion 

  ```noteDatabase.getNoteDao().deleteNote(notes.get(pos));```

  > Do not forget to remove object from list which is being used in NoteListActivity to display list and also notify the adapter using `adapterObj.notifyDataSetChanged()`

Room offers many other features like [LiveData](https://developer.android.com/topic/libraries/architecture/livedata.html) for keeping the data source updated all the time and `rxAndroid` for reactive programming.

Check out the sample application [**repository here**](https://github.com/Pavneet-Sing/RoomDemo), please post your comments and suggestion, it would mean a lot.