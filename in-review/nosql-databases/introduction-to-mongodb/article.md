In this guide, we are going to learn about [MongoDB](https://www.mongodb.com/), an open-source document-oriented NoSql database. This tutorial will cover its installation and basic MongoDB operations. We will also develop a simple [NodeJs](https://nodejs.org/en/) application using the `mongodb` driver.

## MongoDB

MongoDB is a cross-platform, powerful, document-oriented database which is flexible and easily scalable in terms of data storage. It also has built-in support for MapReduce-style [Aggregation](https://docs.mongodb.com/manual/aggregation/) and [Geospatial](https://docs.mongodb.com/manual/applications/geospatial-indexes/) indexes.

MongoDB is made up of databases which contain multiple _collections_. Each collection is made up of _documents_. Each document is made up of _fields_ which are '_named:value_' pairs (JSON objects). Collections can be indexed, which improves lookup and sorting performance.

#### Quick comparision between SQL and MongoDB

![Sql-Mongodb Correspondance](https://raw.githubusercontent.com/pluralsight/guides/master/images/1c7d6087-d8ed-471f-b15b-5474946699db.PNG)

##### Note :
> Some common features of RDB's are not present in MongoDB, like
_Joins_ and complex multirow _Transactions_. These are architectural decisions to allow
for scalability, because both of these features are difficult to provide efficiently in a
distributed system.

But according to a recent announcement, [joins are coming](https://www.mongodb.com/blog/post/joins-and-other-aggregation-enhancements-coming-in-mongodb-3-2-part-1-of-3-introduction) to MongoDB with version __3.2__ release in the form of brand new pipeline operator called `$lookup`.


## Installation

In this guide, I will install MongoDB on Ubuntu. However, the database can be installed on [multiple platforms](https://docs.mongodb.com/manual/installation/)

#### Steps :

1. ** Import the public key used by the package management system ** 
   ``` 
     sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927 
        ```
2. ** Create MongoDB source list file **
     ``` 
    echo "deb http://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list 
   ```
3. ** Update the repository ** 
    ``` 
    sudo apt-get update
    ```
4. ** Install mongodb **
     ```
    sudo apt-get install -y mongodb-org
    ```
5. ** Starting Mongo Server **
    ``` 
    sudo service mongod start 
    ```


MongoDB stores its data files under `/var/lib/mongodb` and its log files under `/var/log/mongodb` by default, and it runs using the mongodb user account. However you can specify alternate locations for data and log files in the MongoDB configuration file, `/etc/mongod.conf`.


## Basic Operations using Mongo Shell

The Mongo shell is an interactive JavaScript interface to MongoDB. You can use the Mongo shell to query and update data and also perform administrative operations.

Once you have installed and have started MongoDB, connect the Mongo shell to your running MongoDB server. By default, the Mongo server will start on **port 27017**.

Connect to MongoDB instance from cmd prompt : **```$ mongo```**

![Using mongo shell](https://raw.githubusercontent.com/pluralsight/guides/master/images/3b4e3b8d-8f73-4aa1-beab-2ae723dbd060.JPG)


#### ** 1. Creating & Deleting Database **

The ** ```$ use Database_Name```** command is used to create database. This command will create a new database if one doesn't exist. Otherwise, it will return the existing database.



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7260bb4f-68f1-4d00-ad4a-a483487f7736.JPG)


The ** ``` $ db ```** command is used to check current database. The ** ```$ show dbs```** command will list all the existing databases.

The ** ``db.dropDatabase()``** command is used to delete the database.
```
{ "dropped" : "local", "ok" : 1 }
```

#### ** 2. Creating & Deleting Collections **

MongoDB collections are created using the syntax below:
```
db.createCollection(name, options)
```
where `name` is the name of the collection to be created and `options` (an optional parameter) specifies options about indexing and memory limit.

Collections are deleted using the syntax below:
```
db.COLLECTION_NAME.drop()
```

#### ** 3. CRUD operations on documents in a collection **

###### ** 1. Inserting Documents **
    
`insert()` or `save()` methods are used to insert documents into collections. While inserting a document, if the collection doesn't exist, a new collection will be created. Otherwise, the document is inserted into the existing document.    

```
db.employees.insert({
    "employee_id":1,
    "name":"employee1"
});

WriteResult({ "nInserted" : 1 })
```
Here `employees` is the collection which holds `employee` documents.

##### ** 2. Finding an existing document **

Use the `find()` method to query a document from your collection. The `pretty()` method can be used along with `find()` to beautify and format the document returned by the query.

```
> db.employees.find().pretty()
{
	"_id" : ObjectId("579c657b56c7d42812e2788c"),
	"employeeid" : 1,
	"name" : "sanju"
}
{
	"_id" : ObjectId("579c67e9b87b4b49be12664b"),
	"name" : "employee1",
	"employee_id" : 2
}
```
Notice the `_id` field in the document. This field is a 12-bytes Hexadecimal number that assures the uniqueness of every document.

##### ** 3. Updating an existing document **

Mongodb uses `update()` and `save()` methods to update an documnet in a collection. The `update()` method updates an existing documnet while `save()` method replaces the existing documet.

```
'Update' syntax : db.collection.update(<query>,<update>)

db.employees.update({"employee_id":2},{"name":"Employee2"})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

db.employees.save({"employee_id":1,"name":"Sanjeev"})
WriteResult({ "nInserted" : 1 })

 db.employees.find()
{ "_id" : ObjectId("579c6efbb87b4b49be12664d"), "employee_id" : 1, "name" : "Sanjeev" }
```

##### ** 4. Deleting a Document **

The `remove()` method is used to remove document from the collection. It accepts two parameters: the `deletion criteria` and the `justOne` flag.



```
db.employees.remove({"_id":ObjectId("579c6ecab87b4b49be12664c")})
WriteResult({ "nRemoved" : 1 })
```

These are some basic [CRUD operations](https://docs.mongodb.com/master/crud/) we covered, to explore more about complex operations with aggregation and indexing plese refer official documentation.


## Using the NodeJS API to perform basic operations

We have seen how to interact with MongoDB using the Mongo shell. Now we will build a NodeJS API to interact with the same Instace via the `mongodb` driver. This helps us in establishing a connection between Node App and MongoDB instance.

Our API will perform basic CRUD operations on the _employees_ collection under _corporate_ database.

** Technology Stack :** NodeJS + ExpressJS + mongodb 

** Folder Structure : **
```
.
|-- business
|   `-- employeeDAO.js
|-- config
|   `-- config.js
|-- db
|   `-- db.js
|-- package.json
|-- README.md
|-- routes
|   `-- routes.js
`-- server.js

Quick Overview
----------------
config      : Which holds the configuration details of database and port number
routes      : Where routes are defined for API operations
business    : Holds Business logic to perform CRUD operation on database
server.js   : Entry point of our Application

```

##### ** API's  **
+  { URI: `\employee` , Method:`GET`}  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Get all employee documents.
+  { URI: `\employee\:id` , Method:`GET`} &nbsp;Get employee document by id.
+  { URI: `\employee` , Method:`POST`} &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Create new employee document.
+  { URI: `\employee\:id` , Method:`POST`} Update existing employee document.

##### ** Config Object **

A Config object is a javascript Object which holds _database_ name, _port_ on which MongoDB instance is running. If MongoDB has configured with any Authentication, it can hold _username_ and _password_ for the same.

```
Below Object is exported from "config.js" based on Environment

   dev: {
		rootPath:rootPath,
		db:'mongodb://localhost:27017/corporate',
		port:process.env.PORT || 3030,
		uname:'*',
		password:'*'
	}
```

##### ** Starting the application  **

Let's configure and invoke the application inside the `server.js` file.

```
Server.js

//Importing config object
var config=require('./config/config.js')['dev'];

//Importing database connection object
var db = require('./db/db.js');

//Using Express Routes
require('./routes/routes.js')(app,config);

//Initializing Express Application
var app=express();
	app.use(cookieparser());
	app.use(bodyParser.urlencoded({extended:true}));
	app.use(bodyParser.json());

//Calling connect() method to establish connection form NodeJs app to MongoDB instance 
db.connect(config.db, function(err) {
    //on error terminate application
    if(err){
        console.log("Error");
        process.exit(1);
    }else{//on success start Node application 
        console.log("db connection estlablished !!");
        app.listen(config.port, function() {
            console.log("App runnin on port : "+config.port);
        });  
    }
});

```
##### ** Database Connection Object **

This object holds the utility to work with MongoDB
```
db.js

// import mongo clinet from mongodb driver
var MongoClient = require('mongodb').MongoClient

//variable to hold  connection state
var state = {
  db: null,
}

//Function which handles connection 
exports.connect = function(url, done) {
  if (state.db) return done()
  MongoClient.connect(url, function(err, db) {
    if (err) return done(err)
    //on success store db connection object
    state.db = db
    done()
  })
}

//function to get cachecd db object
exports.get = function() {
  return state.db
}

//Function to terminate connection between NOdejs app and MongoDB
exports.close = function(done) {
  if (state.db) {
    state.db.close(function(err, result) {
      state.db = null
      state.mode = null
      done(err)
    })
  }
}

```

##### ** Express Routes **

Here we will be looking into using [Express Routes for handling MongoDB](https://jefftschwartz.wordpress.com/2012/08/23/use-express-route-specific-middleware-to-access-mongodb/). For complete source code, please visit the [github repository](https://github.com/SanjeevMurthy/mongolab_mongoclient).

Let's create an express route to get all employee documents from employee collection.
```

routes.js

//DAO objects which hold business logic to perform CRUD operations
var employeeDAO=require('../business/employeeDAO.js');

//Express route which calls the DAO object to get all employee documents
app.get('/employee',function(req,res){
	employeeDAO.getAllEmployees(function(err,response){
	    if(err){
			res.end(err);
		}else{
			res.status(200);
			res.json(response);
			res.end();
		}
	});
});
	
```

##### ** Data access layer **

The DAO (Data Access Object) object performs all CRUD related operations on the connected database instance. 

```
employeeDAO.js

//import db object which handles connection
var db = require('../db/db.js')

//Exporting DAO object
module.exports={

    getAllEmployees : function(callback){
    
	    //get the cached db connection object and  define collection name
	    var collection = db.get().collection('employees');
	    //call find() method without any query to retrive all documents form employee collection
	    collection.find().toArray(function(err, result) {
	        if(err){
	            //on error 
	            callback(err,null);
	        }else{
	            //on success
	            callback(null,result);
	        }
	    });	
    }
}
```

##### Testing Node API on HTTP REST client

The screenshot below describes a single API call that gets all employee documents from the `employee` collection.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/76ee1692-6282-46fb-adb3-d03e4bc9520c.JPG)

This completes out journey in getting started with MongoDB. To summarize, we learned what MongoDB is, and got hands-on experience with its terminologies. We compared it with SQL, performed basic CRUD operationsm, and built a simple Node api using the MongoDB driver. Remember that the complete code is on the [Github repository](https://github.com/SanjeevMurthy/mongolab_mongoclient).

__  __

Please feel free to share your feedback using the comments section below. If you found this informative, please favorite this Guide!