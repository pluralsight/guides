Today we are going to take a look at Ionic, PouchDB and CouchDB to create an offline first application.
First of all, I've been using Ionic / PouchDB for 3 weeks now during my spare time (2h max. a day). I want to share what I've learned and the best practices I've found.

We are going to build a simple application: a shopping list with a page of shopping lists and a page of items in a shopping list.


# Taking advantage of PouchDB `_id` field

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

