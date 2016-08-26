Today we are going to take a look at Ionic, PouchDB and CouchDB to create an offline first application.
First of all, I've been using Ionic / PouchDB for 3 weeks now during my spare time (2h max. a day). I want to share what I've learned and the best practices I've found.

We are going to build a simple application: a shopping list with a page of shopping lists and a page of items in a shopping list.

# Designing the models / data access

We're going to use PouchDB so let's design the `PouchDocument`:

```typescript
export interface PouchDocument {
    _id?: string;
    _rev?: string;
    _deleted?: boolean;
    doc_type: string;
}
```

Then we need our model, let's design simple interfaces:

```typescript
export interface ShoppingList {
    name: string;  
    completed: boolean; 
}

export interface ShoppingItem {
    name: string;
    bought: boolean;
}
```

Our documents will extends the model interface and the `PouchDocument`, for example: 

`export interface ShoppingListDocument implements PouchDocument, ShoppingList`


## Taking advantage of PouchDB `_id` field

When I first started using PouchDB, I was not thinking in a NoSQL way. I was doing complex queries with map/reduce to do simple things like get all shopping lists. I rapidly noticed that it was really slow, so I've searched for alternatives in the PouchDB blogs/documentation, and here is what I think is the best one:

Never create document with a generated random `_id`. It's a waste, the `_id` field is very powerful. When you're doing `allDocs` with PouchDB, it returns the result sorted by `_id`. That means you can use the `_id` to store the document type (like`shopping_list` for example) and use the `startkey`and `endkey` parameters of the `allDocs` function.

#### Example

The `_id` of a document for a shopping list is going to be `shopping_list:$name` where `$name` is the name of that shopping list. 
What you can do with this is: 

```typescript
db.allDocs({
    startkey: 'shopping_list:',
    endkey: 'shopping_list:\uffff'
});
```

It will returns all the document starting with `shopping_list`. No more need of an index on the `doc_type` field (or whatever you want to call it).

It even gets better because you can do `"joins"`. When you will store an item of a `shopping_list` you should not store the items inside the `shopping_list` document. It would be a pain to manage that array for inserts, delete, etc. What you can do is storing the `_id` of the shopping list in the `_id` of the shopping item: `shopping_item:$shopping_list_id:$item_name`. With this design you can query all the items of one list:

```typescript
getItems (list: ShoppingList) {
    return db.allDocs({
        startkey: `shopping_item:${list._id}:`
        endkey: `shopping_item:${list._id}:\uffff`
    });
}
```

So what happens if for example we want to change the name of a list / item ? The sorting will be broken because we cannot change the index. One solution I think would work is to delete the old document and create a new one with `bulkDocs`. `bulkDocs` allows you to create/update/delete documents in a batch.

To handle the creation of id easily, you can use a little function that I called `toId`:

```typescript
function normalizeId (id: string) {

    return id.toLowerCase();
}

export function toId (parts: any[]) {

    if (parts.length === 0) {
        throw new Error('parts must at least contain one id');
    }

    return `${normalizeId(parts.map((part: number|string) => {
      if (typeof part === 'number') {
        return `${part}`;
      }
      else if (typeof part === 'string') {
        return part;
      }
      else {
        throw new TypeError('toId: a part can only be a string or a number');
      }
    }).join(':'))}`;
}
```

You will only need to do `toId(['shopping_item', list._id])` for example.

## Taking advantage of RxJS 

The `pouchdb` package on `npm` uses promises or callback. We're working with `Ionic 2` here, so we have access to `RxJS`. We can convert these promises into `RxJS` observables.

For example:

```typescript
export class PouchAdapter { 
    
    private db: any;
    
    constructor (dbOptions: any) {
        this.db = new PouchDB(dbOptions);
    }
    
    private fromPromise (promise: Promise<any>) {
        return Observable.fromPromise(promise);
    }
    
    allDocs (options: any) { // you can use typings for the options here, for simplicity, I will use any
        return this.fromPromise(
            this.db.allDocs(options)
        );
    }
    
    changes (options: any = {}): Observable<any> {

        return Observable.create((observer: Observer<any>) => {

            const feed: any = this.db.changes(options)
                .on('change', (change) => observer.next(change))
                .on('error', (err) => observer.error(err))
                .on('complete', (info) => observer.complete());

            return () => feed.cancel();
        });
    }
    
    // All the other methods
}
```

This way, this feels more consistent with `Angular 2`.

## Create the DB and shopping service

This is pretty straightforward to do. We need to create an injectable service that I will call `LocalDb` and a `ShoppingService` that will handle data access to the shopping lists/items via `LocalDb`.

```typescript
@Injectable()
export class LocalDb extends PouchDbAdapter {

  constructor () {
    super({ name: 'your_db_name' });
  }
}
```

I will not copy paste all the service code here, I'm sure you got the idea with only this:

```typescript
@Injectable()
export class ShoppingService {

  private static DOC_TYPE: string = 'shopping_list';

  constructor (private db: LocalDb) {

  }

  listsChanges (): Observable<any> {

    return this.db.changes({
      include_docs: true,
      live: true,
      filter: function (doc: any) {
        return doc.doc_type === ShoppingService.DOC_TYPE;
      },
      since: 'now'
    });
  }

  createList (name: string): Observable<any> {

    return this.post({
      name: name,
      completed: false,
      date: new Date()
    });
  }

  private post (doc: ShoppingListDocument): Observable<any> {

    doc.doc_type = ShoppingService.DOC_TYPE;
    doc._id = toId([doc.doc_type, doc.name]);
    return this.db.put(doc);
  }

  put (doc: ShoppingListDocument): Observable<any> {

    return this.db.put(doc);
  }

  delete (doc: ShoppingListDocument): Observable<any> {

    return this.db.delete(doc);
  }

  getAll (): Observable<any> {

    return this.db.all({
      startkey: `${ShoppingService.DOC_TYPE}:`,
      endkey: `${ShoppingService.DOC_TYPE}:\uffff`
    }).map((x) => {
      x.rows = x.rows.map((row: any) => row.doc);
      return x;
    });
  }

  getOne (id: string): Observable<PouchDocument> {

    return this.db.one(id);
  }
}
```

# Creating the list

So, let's start using `Ionic 2` and `Angular 2`. First we need to create the shopping list page.

```
$ ionic g page shopping-list
```

Let's update the `ShoppingListPage`:

```typescript
constructor (private shopping: ShoppingService) {}

ionViewWillEnter () {
    this.getAll();   
}

getAll () {
    this.loading = true;
    this.findSubscription = this.shopping.getAll().subscribe((response) => {
      this.lists = response.rows;
      this.loading = false;
    }, (err) => {
      // Handle error here
    });   
}
```

Now whenever we enter the view, the lists will be fetched from the local database.

Now let's create a simple template to display our lists:

```html
<ion-list>
  <ion-item-sliding *ngFor="let list of lists">
      <ion-item (click)="onClick(list)">
        {{ list.name }}
      </ion-item>
    <ion-item-options side="right">
      <button secondary (click)="onComplete(list)">
        <ion-icon name="checkmark"></ion-icon>
      </button>
      <button danger (click)="onDelete(list)">
        <ion-icon name="trash"></ion-icon>
      </button>
    </ion-item-options>
  </ion-item-sliding>
</ion-list>
```

We can also add our handlers to the class:

```typescript
onClick (list: ShoppingListDocument) {
    // Navigate to list
    // this.nav.push(ShoppingItemsPage)   
}

onComplete (list: ShoppingListDocument) {
    
    // assign here is a shortcut for Object.assign({}, list, { completed: true });
    // we can't update the list yet if we're not sure the operation will succeed
    this.shopping.put(assign(list, { completed: true })).subscribe();
    // I'm not handling error here, but you should
}

onDelete (list: ShoppingListDocument) {
    this.shopping.delete(assign(list, { _deleted: true })).subscribe();   
}
```

So, if we're not handling the subscribe function, how do we update the view? Well, that's the topic for the next chapter.

## Using the changes feed to update the view

Whenever you're doing a change to an item in a list, you may want to update the view in the callback of that update. But what happens if the update comes from the `sync` that is going on in background? You would need to write the same function that handles the changes. What you can do instead is updating the document and handling any errors that the database could return and let the changes feed update your view. 

It's also a better design because you don't have to worry anymore with  "Where is this change coming from?". It all come from the changes feed.

#### Example

So, we would have something like this (in ShoppingPage):

```typescript
this.changesSubscription = this.shopping.listsChanges().subscribe(
    (change) => {
        this.lists = handleChange(this.lists, change);
    },
    () => { /* handle error */ }
);
```

So here we're calling handleChange which is a function that will update the lists and return a new list for us. 
Let's see how we can handle the changes.

* If a document has been deleted, we need to remove it from the list
* If a document has been added, we need to insert it from the list
* If a document has been updated, we need to update it in the list
* There is no differences between an added and updated document in the change feed. We need to check if the document already exists first.

How can we find a document ? The first idea that comes to mind is a simple loop that checks if the `_id` is equal to the searched document. Well, don't do that, we can do a lot better. Remember that the list is ordered by its `_id` field. That means we can use one of the first algorithm that we learn in computer sciences: `The binary search`. It's way more effective, especially if you have a lot of items in your list.

So let's see how the `handleChange` function looks like.

```typescript
export function handleChange (input: PouchDocument[], change: any) {

  if (change.deleted) {
    return removeFromList(input, change.doc, '_id');
  }

  return insertIntoOrder(input, change.doc, '_id');
}
```
 
Let's use a modified version of the binary search algorithm, we don't only need to know if the element exists but also where to insert it if it doesn't. So we need an additionnal boolean field to tell us if the element exists or not because we can't make the difference between indexes.

```typescript
export function binarySearch (input: any[], value: any, field: string): {
  exists: boolean,
  index: number
} {

  let minIndex = 0;
  let maxIndex = input.length - 1;
  let currentIndex;

  while (minIndex <= maxIndex) {
    currentIndex = (minIndex + maxIndex) / 2 | 0;
    const currentElement = input[currentIndex];

    if (currentElement[field] < value[field]) {
      minIndex = currentIndex + 1;
    }
    else if (currentElement[field] > value[field]) {
      maxIndex = currentIndex - 1;
    }
    else {
      return { exists: true, index: currentIndex };
    }
  }

  return { exists: false, index: maxIndex + 1 }; // Where to insert it if it doesn't exist
}
```

The `removeFromList` function:

```typescript
export function remove (target: any[], index?: number) {

    if (typeof index !== 'number') {
        index = target.length;
    }

    return [...target.slice(0, index), ...target.slice(index + 1)];
}

export function removeFromList (input: any[], value: any, field: string): any[] {
    const { index, exists } = binarySearch(input, value, field);
    return exists ? remove(input, index) : input;
}
```

The `insertIntoOrder` function:

```typescript
export function insert (target: any[], value: any, index?: number) {

    if (typeof index !== 'number') {
        index = target.length;
    }

    return [...target.slice(0, index), value, ...target.slice(index)];
}

export function insertIntoOrder (input: any[], value: any, field: string, binary: boolean = true): any[] {

    const orderedIndex = binarySearch(input, value, field);
    if (!orderedIndex.exists) {
        // insert
        return insert(input, value, orderedIndex.index);
    }
    else {
        // update value
        input[orderedIndex.index] = value;
        return [...input]; // clone array to have another reference
    }
}
```

Now, you have everything you need to handle change locally and from a remote database. 

I didn't write a lot about immutability, but as you can see, all these functions above are returning new object / references. 
I think it's one of the best practices that we can use in our application.

For example, without immutable data, a `pipe` in `Angular 2` must be impure and it will trigger its `transform` method every time a change detection triggers. But if we use immutable data, we can let it pure and the pipe will trigger its `transform`method every time the reference changes.

Same thing for the `NgFor`, but the `NgFor` doesn't work with the reference of the array but with the references of its elements. That means, if an object changes in that array, we need to change that reference to update the view. The `NgFor` is smart enough to update only the changed row.

# Conslusion (todo)

We've seen how to use the `_id` field at our advantages. Of course, sometimes you will need to do complex queries with map/reduce, you will not be able to avoid it. But most of the times the `allDocs` function should be enough.