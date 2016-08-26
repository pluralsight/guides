Today we are going to take a look at Ionic, PouchDB and CouchDB to create an offline first application.
First of all, I've been using Ionic / PouchDB for 3 weeks now during my spare time (2h max. a day). I want to share what I've learned and the best practices I've found.

We are going to build a simple application: a shopping list with a page of shopping lists and a page of items in a shopping list.

# Designing the models / data access

... todo ...

## Taking advantage of RxJS 

... todo ...

## Taking advantage of PouchDB `_id` field

When I first started using PouchDB, I was not thinking in a NoSQL way. I was doing complex queries with map/reduce to do simple things like get all shopping lists. I rapidly noticed that it was really slow, so I've searched for alternatives in the PouchDB blogs/documentation, and here is what I think is the best one:

Never create document with a generated random `_id`. It's a waste, the `_id` field is very powerful. When you're doing `allDocs` with PouchDB, it returns the result sorted by `_id`. That means you can use the `_id` to store the document type (like`shopping_list` for example) and use the `startkey`and `endkey` parameters of the `allDocs` function.

#### Example

The `_id` of a document for a shopping list is going to be `shopping_list:$name` where `$name` is the name of that shopping list. 
What you can do with this is: 

```
db.allDocs({
    startkey: 'shopping_list:',
    endkey: 'shopping_list:\uffff'
});
```

It will returns all the document starting with `shopping_list`. No more need of an index on the `doc_type` field (or whatever you want to call it).

It even gets better because you can do `"joins"`. When you will store an item of a `shopping_list` you should not store the items inside the `shopping_list` document. It would be a pain to manage that array for inserts, delete, etc. What you can do is storing the `_id` of the shopping list in the `_id` of the shopping item: `shopping_item:$shopping_list_id:$item_name`. With this design you can query all the items of one list:

```

getItems (list: ShoppingList) {
    return db.allDocs({
        startkey: `shopping_item:${list._id}:`
        endkey: `shopping_item:${list._id}:\uffff`
    });
}

```

# Creating the list

## Using the changes feed to update the view

Whenever you're doing a change to an item in a list, you may want to update the view in the callback of that update. But what happens if the update comes from the `sync` that is going on in background? You would need to write the same function that handles the changes. What you can do instead is updating the document and handling any errors that the database could return and let the changes feed update your view. 

It's also a better design because you don't have to worry anymore with  "Where is this change coming from?". It all come from the changes feed.

#### Example