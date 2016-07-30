In this guide we are going to learn about [MongoDB](https://www.mongodb.com/), Open-sourced document-oriented NoSql database, Installation, Basic operations on database and will develop a simple [NodeJs](https://nodejs.org/en/) application using `mongodb` driver.

## MongoDB

MongoDB is a cross-platform, powerful, document-oriented database which is flexible, high performant and easily scalable data store. It also has built-in support for MapReduce-style Aggregation and Geospatial indexes.

MongoDB is made up of databases which contain multiple _collections_. Each collection is made up of _documents_. Each document is made up of _fields_ which are '_named:value_' pairs(_JSON Object_). Collections can be indexed, which improves lookup and sorting performance.

#### Quick comparision between SQL and MongoDB

![Sql-Mongodb Correspondance](https://raw.githubusercontent.com/pluralsight/guides/master/images/1c7d6087-d8ed-471f-b15b-5474946699db.PNG)

##### Note :
> Some common features of RDB's are not present in MongoDB, like
_Joins_ and complex multirow _Transactions_. These are architectural decisions to allow
for scalability, because both of these features are difficult to provide efficiently in a
distributed system.

But recent announcement form CTO of MongoDB is that, joins would be coming with version __3.2__ release in the form of brand new pipeline operator called __$lookup__. [Joins and other Aggregation](https://www.mongodb.com/blog/post/joins-and-other-aggregation-enhancements-coming-in-mongodb-3-2-part-1-of-3-introduction)


## Installation

Here we will install MongoDB on Ubuntu, But MongoDB can be installed on multiple platforms. [Supported platforms](https://docs.mongodb.com/manual/installation/)

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


MongoDB stores its data files under __/var/lib/mongodb__ and its log files under  __/var/log/mongodb__ by default, and runs using the mongodb user account. However you can specify alternate location for data and log files in mongodb configuration file __/etc/mongod.conf__.


## Basic Operations using Mongo Shell

The mongo shell is an interactive JavaScript interface to MongoDB. You can use the mongo shell to query and update data as well as perform administrative operations.

Once you have installed and have started MongoDB, connect the mongo shell to your running MongoDB server. By default Mongo server will start on port 27017.

Connect to MongoDB instance from cmd prompt : **```$ mongo```**

![Using mongo shell](https://raw.githubusercontent.com/pluralsight/guides/master/images/3b4e3b8d-8f73-4aa1-beab-2ae723dbd060.JPG)


#### ** 1. Creating & Deleting Database **

** ```$ use Database_Name```** commad is used to create database. This command will create a new database if it doesn't exist else it will return the existing database.



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7260bb4f-68f1-4d00-ad4a-a483487f7736.JPG)


** ``` $ db ```** commad is used to check current database and ** ```$ show dbs```** commad will list all the existing databases.

** ``db.dropDatabase()``** command is used to delete the database.
```
{ "dropped" : "local", "ok" : 1 }
```

#### ** 2. Creating & Deleting Collections **

MongoDB collections are created using the below syntax :
```
db.createCollection(name, options)
```
where _name_ is name if the collection to be created and _options_(optional) to specify options about indexing and memory limit.

Collections are deleted using the below syntax :
```
db.COLLECTION_NAME.drop()
```

#### ** 3. CRUD operations on documents in a collection **

###### ** 1. Inserting Documents **
    
`insert()` or `save()` methods are used to insert documents into collections. While inserting a document if collection doesn't exist, new collection will be created else document is inserted into existing document.    

```
db.employees.insert({
    "employee_id":1,
    "name":"employee1"
});

WriteResult({ "nInserted" : 1 })
```
Here _employees_ is the collection which holds employee documents.

##### ** 2. Finding an existing document **

`find()` method is used to query a document from collection, `pretty()` method can be used along with find() to beautify/formatt the queried document.

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
Here you can notice `_id` field in a document, which is a 12-bytes Hexadecimal number assures the uniqueness of every document.
 
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

`remove()` method is used to remove document from the collection. It accepts two parameters, One is _deletion_ criteria and second is _justOne_ flag.



```
db.employees.remove({"_id":ObjectId("579c6ecab87b4b49be12664c")})
WriteResult({ "nRemoved" : 1 })
```

These are some basic CRUD operations we covered, to explore more about complex operations with aggregation and indexing plese refer official documentation. [CRUD Operations](https://docs.mongodb.com/master/crud/).


## Simple NodeJs API to perform basic operations

We have seen how to interact with MongoDB instance via mongo shell, now we will build simple NodeJs API to interact with the same Instace via `mongodb` driver. Which helps us in establishing a connection between Node App and MongoDB instance.

This API will perform basic CRUD operations on the _employees_ collection under _corporate_ database.

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

Config object is a javascript Object which holds _database_ name, _port_ on which MongoDB instance is running. If MongoDB has configured with any Authentication it can hold _username_ and _password_ for the same.

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

##### ** Starting application  **

Application is configured and invoked inside `server.js` file.

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
Which holds the utility to work with  MongoDB
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
Here we will be looking into one express route. For complete source code please visit github repo [MongoDB-DemoApp](https://github.com/SanjeevMurthy/mongolab_mongoclient).


Express route to get all employee documents from employee collection.
```

routes.js

//DAO objects which holds business logic to perform CRUD operations
var employeeDAO=require('../business/employeeDAO.js');

//express route which call DAO object to get all employee documents
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

DAO object performs all CRUD related operations on the connected database instance. 

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

##### ** Demo **

Testing Node API on HTTP REST client. Below screenshot describes single API call to get all employee documents form _employee_ collection.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/76ee1692-6282-46fb-adb3-d03e4bc9520c.JPG)

This completes out journey in getting started with MongoDB. To summarise again we got to know what MongoDB is ? Its terminologies ,Comparision with SQL, basic CRUD operations and we built simple Node api using mongodb driver. [Github repo](https://github.com/SanjeevMurthy/mongolab_mongoclient)

__  __

Please feel free to share your feedback. If you found this informative please favourite this Guide !!