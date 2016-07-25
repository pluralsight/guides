#### In this guide I will walk you though editing a todo, and adding user authentication using Ionic 2 and Google's Firebase realtime database.
##### Note: This is part 2 in the series.

---

## Where we left off
At the end of part 1 we had our application able to connect to firebase, list todos, and create new todos. So our next step is going to be editing a todo. 

##### Note: Since writing part 1 the Angularfire 2 library has reached beta and is a good choice for any angular firebase application. However, since we have already started down the road of writing everything ourselves we will continue.

### The Dataservice
To start off we will add a new method to our data service called `update` it will take a todo object "`_todo`" and the id "`_id`" of the todo we want to update. In that method we will call our firebase refernce to the todos list, find the todo to update, and update it.

Now in firebase there are two functions we could call to do this update; set and update. Both will update the object at `_id` the difference is that `set` is a destructive action. Meaning that it will completely rewrite the object at `_id`. Where as update will merge the old data and the new data only updating the fields that changed. Example:

```
_id_1: {
  title: 'test',
  complete: false
}

this._todosRef.child(_id_1).set({title: 'set test'}); // => _id_1: { title: 'set test' }
this._todosRef.child(_id_1).update({title: 'update test'}); // => _id_1: { title: 'update test', complete: false }
```

```
update(_id, _todo)
{
    this._todosRef.child(_id).update(_todo)
}
```

 This todos change however, will not be refected in our list. Why you ask? We setup our data service to listen for the `child_add` event in firebase and updating a child is not the same as adding. So we have a few options on what todo. 1. Add a new listener for `value` that triggers when the values of our list change, 2. Change our current listener to `value`, or 3. Update the object itself and not change the listener. We are going to go with option 3.
 
 