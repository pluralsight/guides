#### In this guide I will walk you though editing a todo, and adding user authentication using Ionic 2 and Google's Firebase realtime database.
##### Note: This is part 2 in the series.

---

## Where we left off
At the end of part 1 we had our application able to connect to firebase, list todos, and create new todos. So our next step is going to be editing a todo. 

##### Note: Since writing part 1 the Angularfire 2 library has reached beta and is a good choice for any angular firebase application. However, since we have already started down the road of writing everything ourselves we will continue.

### The Dataservice
To start off we will add a new method to our data service called `update` it will take a todo object "`_todo`" and the id "`_id`" of the todo we want to update. In that method we will call our firebase refernce to the todos list, find the todo to update, update it, and return the promise.

Now in firebase there are two functions we could call to do this update; set and update. Both will update the object at `_id` the difference is that `set` is a destructive action. Meaning that it will completely rewrite the object at `_id`. Where as update will merge the old data and the new data only updating the fields that changed. Example:

```
_id_1: {
  title: 'test',
  complete: false
}

this._todosRef.child(_id_1).set({title: 'set test'}); // => _id_1: { title: 'set test' }
this._todosRef.child(_id_1).update({title: 'update test'}); // => _id_1: { title: 'update test', complete: false }
```
So we will add this to our data service class in data.ts.

```
update(_id, _todo)
{
    return this._todosRef.child(_id).update(_todo)
}
```
Next we will need access to the key firebase creates for each todo, but the way we are currently storing our todos we throw that data away. Lets change the `handleData` method to address that issue.

```

```


 
 