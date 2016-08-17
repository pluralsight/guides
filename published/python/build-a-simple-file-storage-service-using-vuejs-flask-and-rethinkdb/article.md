## Introduction 

In this tutorial, I will be showing you how to build a simple file storage service. We shall be making use of VueJS to handle the front-end interactions, Flask for the back-end, and RethinkDB for database storage. I will be introducing a number of tools as we go, so stay tuned.

In the first part of this series, I will be focusing on building out the back-end for the application. Later, I'll cover implementing some of the principles taught here in your current workflow either as a Python developer, Flask developer, or even as a programmer in general.

### Building the API

Let's start by building out our API. Using this file storage service, as a user, we should be able to:
1. Create an account
2. Login
3. Create and manage folders and subfolders
4. Upload a file into a folder
5. View file properties
6. Edit and Delete files

For the API, we should have the following endpoints:
- `POST /api/v1/auth/login` - This endpoint will be used to login users
- `POST /api/v1/auth/register` - This endpoint will be used to register users
- `GET /api/v1/user/<user_id>/files/` - This endpoint will be used to list files for a user with user id `user_id`
- `POST /api/v1/user/<user_id>/files/` - This endpoint will be used for creating new files for the user with user id `user_id`
- `GET /api/v1/user/<user_id>/files/<file_id>` - This endpoint will be used to get a single file with id `file_id`
- `PUT /api/v1/user/<user_id>/files/<file_id>` - This endpoint will be used to edit a single file with id `file_id`
- `DELETE /api/v1/user/<user_id>/files/<file_id>` - This endpoint will be used to delete a single file with id `file_id`

OK, we've figured out the API endpoints that we need to create this app. Now, we need to start the actual process of creating the endpoints. So let's get to it.

## Setup

You should start by creating a project directory. This is the recommended structure for our application:

```
-- /api
    -- /controllers
    -- /utils
    -- models.py
    -- __init__.py
-- /templates
-- /static
    -- /lib
    -- /js
    -- /css
    -- /img
-- index.html
-- config.py
-- run.py
```
The modules and packages for the API will be put into the `/api` directory with the models stored in the `models.py` module, and the controllers (mostly for routing purposes) stored as modules in the `/controllers` package.

We will add our routes and app creation function to `/api/__init__.py`. This way, we are able to use the `create_app()` function to create multiple app instances with different configurations. This is especially helpful when you are writing tests for your application.

```
from flask import Flask, Blueprint
from flask_restful import Api

from config import config

def create_app(env):
    app = Flask(__name__)
    app.config.from_object(config[env])

    api_bp = Blueprint('api', __name__)
    api = Api(api_bp)

    # Code for adding Flask RESTful resources goes here

    app.register_blueprint(api_bp, url_prefix="/api/v1")

    return app
```

From what you can see here, we have created a function `create_app()` which takes an `env` parameter. Ideally, `env` should be one of `development`, `production` and `testing`. Using the value supplied for `env`, we will be loading specific configurations. This configuration information is stored in `config.py` as a dictionary of classes.

```
class Config(object):
    DEBUG = True
    TESTING = False
    DATABASE_NAME = "papers"

class DevelopmentConfig(Config):
    SECRET_KEY = "S0m3S3cr3tK3y"

config = {
    'development': DevelopmentConfig,
    'testing': DevelopmentConfig,
    'production': DevelopmentConfig
}
```
For now we've specified only a few configuration parameters like the `DEBUG` parameter which tells Flask whether or not to run in debug mode. We have also specified the `DATABASE_NAME` parameter, which we shall be referencing in our models and a `SECRET_KEY` parameter for JWT Token generation. We have used the same settings for all the environments for now.

As you can see, we are using Flask Blueprint to version our API, just in case we figured we need to change implementation of core functionality without affecting the user, and we have also initialized `api`, a Flask-RESTful API object. In a few moments I will show you how to add new API endpoints to our app using this object.

Next, we put the code for running the server into `run.py` in the root directory. We will be using Flask-Script to create extra CLI commands for our application.

```
from flask_script import Manager

from api import create_app

app = create_app('development')

manager = Manager(app)

@manager.command
def migrate():
    # Migration script
    pass

if __name__ == '__main__':
    manager.run()
```

Here, we have used the `flask_script.Manager` class to abstract out the running of the server and enable us to add new commands for the CLI. `migrate()` will be used to automatically create all the tables required by our models. This is a simplified solution for now. If all is well, you should not have any errors.


Now, we can go to the command line and run `python run.py runserver` to run the server on the default port 5000.

## User model
Next up, we shall be adding in code for our models. For this application, we need just 2 models for starters. For this step in the tutorial, we will be creating just the user model.

We start off by connecting to RethinkDB and creating a connection object to our database.

```
import rethinkdb as r

from flask import current_app

conn = r.connect(db="papers")

class RethinkDBModel(object):
    pass
```

We referenced the database name from the application config. In flask, we have the `current_app` variable which holds a reference to the currently running application instance.

Why did I create a blank `RethinkDBModel` class? Well, there might be a couple of things we might want to share across model classes; this class is here in case cross-model sharing is necessary.

Our `User` class will inherit from this empty base class. In `User`, we will have a few functions which we'll use to interact with the database from the controllers.

We start with the `create()` function. This function will be called when we need to create a user document in the table.

```
class User(RethinkDBModel):
    _table = 'users'

    @classmethod
    def create(cls, **kwargs):
        fullname = kwargs.get('fullname')
        email = kwargs.get('email')
        password = kwargs.get('password')
        password_conf = kwargs.get('password_conf')
        if password != password_conf:
            raise ValidationError("Password and Confirm password need to be the same value")
        password = cls.hash_password(password)
        doc = {
            'fullname': fullname,
            'email': email,
            'password': password,
            'date_created': datetime.now(r.make_timezone('+01:00')),
            'date_modified': datetime.now(r.make_timezone('+01:00'))
        }
        r.table(cls._table).insert(doc).run(conn)
```
Here, we are making use of the `classmethod` decorator. This decorator enables us have access to the class instance from within the method body. We will be using the class instance to access the `_table` property from within the method. The `_table` stores the table name for that model.

We have also in here added code for making sure that the `password` and `password_conf` fields are the same. If this happens, a `ValidationError` will be thrown. The exceptions will be stored in the `/api/utils/errors.py` module. Find the definition of `ValidationError` below:

```
class ValidationError(Exception):
    pass
```

We're using named exceptions because they are easier to track.

Notice how we used `datetime.now(r.make_timezone('+01:00'))` here? There were issues faced when I used `datetime.now()` without the timezone. **RethinkDB requires that time zone information be set on date fields in documents.** The Python function does not supply this for us by default unless we specify this as a parameter to the `now()` function (See [here](https://www.rethinkdb.com/docs/dates-and-times/python/) for more). Using the `r.make_timezone('+01:00')` we are able to create a timezone object that we can use for the `datetime.now()` function.

If all goes well and no exceptions are encountered, we call the `insert()` method on the table object that `r.table(table_name)` returns. This method takes a dictionary containing the data. This data will be stored as a new document in the table selected.

We have made a call to the `hash_password()` method in our class. This method makes use of the `hash.pbkdf2_sha256` module in the `passlib` package to hash the password fairly securely. In addition to that, we will need to create a method for verifying passwords.

```
from passlib.hash import pbkdf2_sha256

class User(RethinkDBModel):
    _table = 'users'

    @classmethod
    def create(cls, **kwargs):
        fullname = kwargs.get('fullname')
        email = kwargs.get('email')
        password = kwargs.get('password')
        password_conf = kwargs.get('password_conf')
        if password != password_conf:
            raise ValidationError("Password and Confirm password need to be the same value")
        password = cls.hash_password(password)
        doc = {
            'fullname': fullname,
            'email': email,
            'password': password,
            'date_created': datetime.now(r.make_timezone('+01:00')),
            'date_modified': datetime.now(r.make_timezone('+01:00'))
        }
        r.table(cls._table).insert(doc).run(conn)
    
    @staticmethod
    def hash_password(password):
        return pbkdf2_sha256.encrypt(password, rounds=200000, salt_size=16)

    @staticmethod
    def verify_password(password, _hash):
        return pbkdf2_sha256.verify(password, _hash)

```
The `pbkdf2_sha256.encrypt()` method is called with the password and values for `rounds` and `salt_size`. See [here](http://pythonhosted.org/passlib/lib/passlib.hash.pbkdf2_digest.html) for details on how you can customize your encryption and how the library works. Just to give some context on the decision to use `PBKDF2`: 

> Security-wise, PBKDF2 is currently one of the leading key derivation functions, and has no known security issues.

-- _Quotes from the passlib documentation_

The `verify_password` method will be called using the password string and a hash. It will return true or false if the password is valid.

We will now move on the the `validate()` function. This function will be called in the login method with the email address and password. The function will check that the document exists using the `email` field as index and then compare the password hash against the password supplied.

In addition to that, since we're going to be making using of JWT (JSON Web Token) for token-based authentication, we will be generating a token if the user supplies valid information. This is how the entire `models.py` will look like when we're done adding in logic for that.

```
import os
import rethinkdb as r
from jose import jwt
from datetime import datetime
from passlib.hash import pbkdf2_sha256

from flask import current_app

from api.utils.errors import ValidationError

conn = r.connect(db="papers")

class RethinkDBModel(object):
    pass


class User(RethinkDBModel):
    _table = 'users'

    @classmethod
    def create(cls, **kwargs):
        fullname = kwargs.get('fullname')
        email = kwargs.get('email')
        password = kwargs.get('password')
        password_conf = kwargs.get('password_conf')
        if password != password_conf:
            raise ValidationError("Password and Confirm password need to be the same value")
        password = cls.hash_password(password)
        doc = {
            'fullname': fullname,
            'email': email,
            'password': password,
            'date_created': datetime.now(r.make_timezone('+01:00')),
            'date_modified': datetime.now(r.make_timezone('+01:00'))
        }
        r.table(cls._table).insert(doc).run(conn)

    @classmethod
    def validate(cls, email, password):
        docs = list(r.table(cls._table).filter({'email': email}).run(conn))

        if not len(docs):
            raise ValidationError("Could not find the e-mail address you specified")

        _hash = docs[0]['password']

        if cls.verify_password(password, _hash):
            try:
                token = jwt.encode({'id': docs[0]['id']}, current_app.config['SECRET_KEY'], algorithm='HS256')
                return token
            except JWTError:
                raise ValidationError("There was a problem while trying to create a JWT token.")
        else:
            raise ValidationError("The password you inputted was incorrect.")
    
    @staticmethod
    def hash_password(password):
        return pbkdf2_sha256.encrypt(password, rounds=200000, salt_size=16)

    @staticmethod
    def verify_password(password, _hash):
        return pbkdf2_sha256.verify(password, _hash)
```

A couple of things to take note of in the `validate()` method. Firstly, the `filter()` function was used here on the table object. This command takes in a dictionary that is used to search through the table. This function in some cases can also take a predicate. This predicate can be a lambda function and will be used similarly to what is done with the python `filter` function or any other function that takes a function as an argument. The function returns a cursor that can be used to access all the documents that are returned by the query. The cursor is iterable and as such we can iterate the cursor object using the `for...in` loop. In this case, we have chosen to convert the iterable object to a list using the Python list function.

As you would expect, we basically do two things here. We want to know if the email address exists at all and then if the password is correct. For the first part, we basically count the collection. If this is empty, we raise an error. For the second part, we call the `verify_password()` function to compare the password supplied with the hash in the database. We raise an exception if these don't match.

Also noteworthy is how we have used `jwt.encode()` to create a JWT token and return it to the controller. This method is fairly straightforward and you can see the documentation [here](https://github.com/mpdavis/python-jose).

That does it for the model. Let's move on to the controllers. We have, in this model, tried to obey the principle of having **Fat models and Slim controllers**. Most of the logic is in the models. This way, our controllers only focus on routing and error reporting to the API end user.

## Authentication Controller

For the authentication controller, we need to add in Flask RESTful resource sub classes. Django web development is similar to class-based views. It's simply created as a subclass of the `flask_restful.Resource` class. Your subclasses will have methods that map to respective HTTP verbs. For instance, if we wanted to implement a GET action, we will be creating a `get()` method in our Resource subclass. The process is completed by mapping URLs to the respective classes using the `api.add_resource()` method.

Let us now add in two classes; one to take care of POST action to the login route and one to take care of POST action to the register route.

Let's start by creating the required classes. These should be stored in the `/api/controllers/auth.py` file.

```
from flask_restful import Resource

class AuthLogin(Resource):
    def post(self):
        pass

class AuthRegister(Resource):
    def post(self):
        pass
```

Next up, we will be creating these routes and referencing the respective classes in our `/api/__init__.py` file.

```
from flask import Flask, Blueprint
from flask_restful import Api

from api.controllers import auth
from config import config

def create_app(env):
    app = Flask(__name__)
    app.config.from_object(config[env])

    api_bp = Blueprint('api', __name__)
    api = Api(api_bp)

    api.add_resource(auth.AuthLogin, '/auth/login')
    api.add_resource(auth.AuthRegister, '/auth/register')

    app.register_blueprint(api_bp, url_prefix="/api/v1")

    return app
```

Now lets head back to the controller file to add in some logic. The logic required here is similar to what you will typically do with authentication systems in general.

```
from flask_restful import reqparse, abort, Resource

from api.models import User
from api.utils.errors import ValidationError

class AuthLogin(Resource):
    def post(self):
        parser = reqparse.RequestParser()
        parser.add_argument('email', type=str, help='You need to enter your e-mail address', required=True)
        parser.add_argument('password', type=str, help='You need to enter your password', required=True)
        
        args = parser.parse_args()
        
        email = args.get('email')
        password = args.get('password')
        
        try:
            token = User.validate(email, password)
            return {'token': token}
        except ValidationError as e:
            abort(400, message='There was an error while trying to log you in -> {}'.format(e.message))

class AuthRegister(Resource):
    def post(self):
        parser = reqparse.RequestParser()
        parser.add_argument('fullname', type=str, help='You need to enter your full name', required=True)
        parser.add_argument('email', type=str, help='You need to enter your e-mail address', required=True)
        parser.add_argument('password', type=str, help='You need to enter your chosen password', required=True)
        parser.add_argument('password_conf', type=str, help='You need to enter the confirm password field', required=True)
        
        args = parser.parse_args()
        
        email = args.get('email')
        password = args.get('password')
        password_conf = args.get('password_conf')
        fullname = args.get('fullname')
        
        try:
            User.create(
                email=email,
                password=password,
                password_conf=password_conf,
                fullname=fullname
            )
            return {'message': 'Successfully created your account.'}
        except ValidationError as e:
            abort(400, message='There was an error while trying to create your account -> {}'.format(e.message))
        
```

As mentioned earlier, the majority of the logic and database interaction has been pushed to the model. The controller logic is relatively simple.

To summarize what was done, for the login controller `AuthLogin`, we created a `post()` function which accepts the e-mail address and password, validates the fields using `reqparse`, and calls `User.validate()` which validates the information sent and returns a token. If an error occurs, we catch it and respond with an error message.

Similarly, for the `AuthRegister`, we collect information from the user and call a model `create()` function. In this case, we create a collection for the email address, password, password confirm, and full name fields. We pass all these values to the `User.create()` function and as before, this function will throw an error if anything goes wrong.

All things being equal, everything should work just fine. Run the server using `python run.py runserver` to test it out. You should be able to access the two endpoints that we've created here, and it should work very well.

Next up, we'll be creating the models for our files.

## File and Folder models

We'll be creating simple models for working with the files and folders similar to what we did with the User model. We shall be creating a `Folder` model as a child of the `File` model.

### File model

What you will notice as we proceed is that the fact files are stored in a flat manner in the filesystem. All users have a folder where all their files are stored but the structure of the data is logical and stored in the database. This way, we have minimal writes on the file system. To do this we will be employing some pretty neat techniques that will probably be useful to you for future projects.

Create the base models in `/api/models.py`

```
class File(RethinkDBModel):
    _table = 'files'
    
class Folder(File):
    pass
```
We start out by creating the `create()` method for the File model. This method will be called when we make a POST request to the `/users/<user_id>/files/<file_id>` endpoint used to create a file.

```
@classmethod
def create(cls, **kwargs):
    name = kwargs.get('name')
    size = kwargs.get('size')
    uri = kwargs.get('uri')
    parent = kwargs.get('parent')
    creator = kwargs.get('creator')

    # Direct parent ID
    parent_id = '0' if parent is None else parent['id']

    doc = {
        'name': name,
        'size': size,
        'uri': uri,
        'parent_id': parent_id,
        'creator': creator,
        'is_folder': False,
        'status': True,
        'date_created': datetime.now(r.make_timezone('+01:00')),
        'date_modified': datetime.now(r.make_timezone('+01:00'))
    }

    res = r.table(cls._table).insert(doc).run(conn)
    doc['id'] = res['generated_keys'][0]

    if parent is not None:
        Folder.add_object(parent, doc['id'])

    return doc
```
Here we first collect all the information we need from the keyword arguments dictionary like name, size, file URI, creator and so on. We have collected a parameter which we are calling `parent`. This field points to the `id` of the folder in which we want to store this file. We can choose not to pass this parameter if we want to store the file in the root folder. As we go on, you will understand how we can make use of this to create complex nested folder structures.

Notice here how having a `parent` of `None` makes the `parent_id` field `0`. This takes care of cases when a file is created with no parent. This assumes that we are storing the file in the root folder which will have an id of 0.

So we collect all this information about the file into a dictionary and call the `insert()` function to store it in the database. The returned dictionary from calling the insert function contains the IDs of the newly generated documents. After inserting the dictionary, we populate the ID information in the dictionary so that we can return it to the users.

In the last 3 lines of this method, we've added in a check to see if the `parent` is `None`. Since this file manager implementation has folders, whenever we create a file in a folder, we would have to logically add each newly created object to a folder. We do this by adding the object ID into an `objects` list in the corresponding record for the folder we're trying to store it in. We do this by calling a method which we will create in the Folder model called `add_object`.

Next up we will go back to our base `RethinkDBModel` class to create a number of useful methods which we may or may not override in the child classes.

```
class RethinkDBModel(object):
    @classmethod
    def find(cls, id):
        return r.table(cls._table).get(id).run(conn)

    @classmethod
    def filter(cls, predicate):
        return list(r.table(cls._table).filter(predicate).run(conn))

    @classmethod
    def update(cls, id, fields):
        status = r.table(cls._table).get(id).update(fields).run(conn)
        if status['errors']:
            raise DatabaseProcessError("Could not complete the update action")
        return True
    
    @classmethod
    def delete(cls, id):
        status = r.table(cls._table).get(id).delete().run(conn)
        if status['errors']:
            raise DatabaseProcessError("Could not complete the delete action")
        return True
```

Here we created wrapper methods for the RethinkDB `get()`, `filter()`, `update()` and `delete()` functions. This way, subclasses can leverage on those functions for much complex interactions.

The next method we will be creating in our File model is a function that will be used to move files between folders.

```
@classmethod
def move(cls, obj, to):
    previous_folder_id = obj['parent_id']
    previous_folder = Folder.find(previous_folder_id)
    Folder.remove_object(previous_folder, obj['id'])
    Folder.add_object(to, obj['id'])
```

The logic here is fairly simple. We call this method when we want to move a file `obj` into folder `to`.

We start by getting the current folder id for the current parent directory of the file. This is stored in the `parent_id` field of `obj`. We call the Folder model `find` function to obtain the folder object as a dictionary called `previous_folder`. After getting this object, we do two things. We remove the file object from the previous folder `previous_folder`, and add the file object to the new folder `to`. We achieve this by calling the `remove_object()` and `add_object()` methods of the Folder class. These methods remove the file ID from and add the file ID to the `objects` list in the Folder document, respectively. I will be showing what the implementations for these look like in a bit.

We're now done with modeling the files. We can carry out basic interactions on files like creating, editing, deleting from the database, and more. 

Next we move on to the logic for the `Folder` model which is very similar to what we did for the files.

### Folder model

```
@classmethod
def create(cls, **kwargs):
    name = kwargs.get('name')
    parent = kwargs.get('parent')
    creator = kwargs.get('creator')

    # Direct parent ID
    parent_id = '0' if parent is None else parent['id']

    doc = {
        'name': name,
        'parent_id': parent_id,
        'creator': creator,
        'is_folder': True,
        'last_index': 0,
        'status': True,
        'objects': None,
        'date_created': datetime.now(r.make_timezone('+01:00')),
        'date_modified': datetime.now(r.make_timezone('+01:00'))
    }

    res = r.table(cls._table).insert(doc).run(conn)
    doc['id'] = res['generated_keys'][0]

    if parent is not None:
        cls.add_object(parent, doc['id'], True)
    
    cls.tag_folder(parent, doc['id'])
    
    return doc

@classmethod
def tag_folder(cls, parent, id):
    tag = id if parent is None else '{}#{}'.format(parent['tag'], parent['last_index'])
    cls.update(id, {'tag': tag})
```

The `create()` method here is very similar to what we have for files with a few modifications. The first thing to notice, obviously, is the fact that we only need the name and the creator of the folder to create a folder. We have used similar logic for determining the parent folder and at the end also called the `add_object()` method to add the folder to the parent folder.

We have included the `is_folder` field here which defaults to `True` for folders and if you had noticed earlier, `False` for files.

You will notice here that we have called a `tag_folder()` method. We will be needing this later on for dealing with moving folders. To summarise, folders are tagged according to their position on the File tree. The indexes are based on their level on the tree. Any folder stored at the root level will have a tag of `<id>` where id is the folder's id. Any folder stored below that folder will have an id of `<id>-n` where n is an integer that increments continuously. Subsequently nested folders will follow the same pattern and have ids of `<id>-n-m` etc. `n` will change as we add more folders and we store the data required for this in the `last_index` field of each folder which defaults to `0`. As we add folders to this folder, we will be incrementing the value for `last_index`. **The `tag_folder()` method takes care of all this.**


We insert the dictionary we created to house all this data into the database by calling the `insert()` function.

Next, we will be overriding the find method of the File class to include functionality to show listing information for folders. This will be very useful for our Front end later on.

```
@classmethod
def find(cls, id, listing=False):
    file_ref = r.table(cls._table).get(id).run(conn)
    if file_ref is not None:
        if file_ref['is_folder'] and listing and file_ref['objects'] is not None:
            file_ref['objects'] = list(r.table(cls._table).get_all(r.args(file_ref['objects'])).run(conn))
    return file_ref
```

Here, we start by getting the object using it's id. We show a folder's listings on fulfilling 3 conditions:
- `listing` is set to True. We use this variable to know if we actually want information about the files contained in a folder.
- The `file_ref` object is actually a folder. We determine this by checking the `is_folder` field of the document.
- We have objects within the `objects` list of the folder document.

If all these conditions are fulfilled, we get all the nested objects by calling the  `get_all` method on the file table. This method accepts multiple keys and returns all the objects with the respective keys. We use the `r.args` method to convert the list of objects to multiple arguments for the `get_all` method. We replace the `objects` field of the document with the list returned. This list contains the details of each of these nested files/folders.

Next, we move on to create the `move` method for the folder. This is going to be very similar to the `move` method we created for files earlier on with the inclusion of logic for working with tags and ensuring we can actually move the folder.
```
@classmethod
def move(cls, obj, to):
    if to is not None:
        parent_tag = to['tag']
        child_tag = obj['tag']

        parent_sections = parent_tag.split("#")
        child_sections = child_tag.split("#")

        if len(parent_sections) > len(child_sections):
            matches = re.match(child_tag, parent_tag)
            if matches is not None:
                raise Exception("You can't move this object to the specified folder")

    previous_folder_id = obj['parent_id']
    previous_folder = cls.find(previous_folder_id)
    cls.remove_object(previous_folder, obj['id'])

    if to is not None:
        cls.add_object(to, obj['id'], True)
```
Here we first ensure that the folder we're moving to was in fact specified and is not `None`. This is because the assumption here is that if this is not specified we are actually moving this folder to the root folder.

We get the tag of the folder we're trying to move. We also get the tag of the folder we're trying to move it to. We compare the number of sections in their tags. This is how we know the folder's level in the file tree. There is only one case where moving is not so straightforward: when there are more parent sections than child sections. (Parent in this case refers to the folder we're trying to move this folder to). We can move a folder to any folder on it's level and above but if the `parent_sections` is more the `child_sections`, we know that it is possible that the folder to which we're trying to move this folder might be nested in its own folder. We are very careful about this because as mentioned earlier, folder structure is purely logical and we need to ensure we don't have errors with this.

For the case that the folder we're moving to is below the folder we're moving in the file tree, we must ensure that the former folder is not nested in the latter. This can simply be done by ensuring the `child_tag` of the folder we're moving, does not begin the `parent_tag` string. We use regex to implement this and raise an exception if this happens.

We're almost done now! Finally we will create the `add_object()` and `remove_object()` methods I had referred to earlier.

```
@classmethod
def remove_object(cls, folder, object_id):
    update_fields = folder['objects'] or []
    while object_id in update_fields:
        update_fields.remove(object_id)
    cls.update(folder['id'], {'objects': update_fields})

@classmethod
def add_object(cls, folder, object_id, is_folder=False):
    p = {}
    update_fields = folder['objects'] or []
    update_fields.append(object_id)
    if is_folder:
        p['last_index'] = folder['last_index'] + 1
    p['objects'] = update_fields
    cls.update(folder['id'], p)
```
As mentioned earlier, we will be doing the add and remove operations by modifying the `objects` list in the folder object. When adding subfolders, we put in a constraint to also update the `last_index` variable of the folder.

We're now done with the models. Moving over to the controllers!

## File Controller

The File controllers will be used for working with both Files and folders, hence, there will be slightly more logic here than in our previous controller. We start by creating a boilerplate for our controller in `/api/controllers/files.py` module.

```
import os

from flask import request, g
from flask_restful import reqparse, abort, Resource
from werkzeug import secure_filename

from api.models import File

BASE_DIR = os.path.abspath(
    os.path.dirname(os.path.dirname(os.path.dirname(__file__)))
)


class CreateList(Resource):
    def get(self, user_id):
        pass

    def post(self, user_id):
        pass

class ViewEditDelete(Resource):
    def get(self, user_id, file_id):
        pass

    def put(self, user_id, file_id):
        pass

    def delete(self, user_id, file_id):
        pass

```

The `CreateList` class, as the name implies, will be used for creating and listing files for a logged in user. The `ViewEditDelete` class, also as the name implies, will be used for viewing, editing and deleting files. The methods we're using in the classes correspond with the appropriate HTTP actions.

We will start the implementation by creating a bunch of decorators which we will be using on the methods in our Resource classes. You want to separate this out into the `/api/utils/decorators.py` module.

```
from jose import jwt
from jose.exceptions import JWTError
from functools import wraps

from flask import current_app, request, g
from flask_restful import abort

from api.models import User, File

def login_required(f):
    '''
    This decorator checks the header to ensure a valid token is set
    '''
    @wraps(f)
    def func(*args, **kwargs):
        try:
            if 'authorization' not in request.headers:
                abort(404, message="You need to be logged in to access this resource")
            token = request.headers.get('authorization')
            payload = jwt.decode(token, current_app.config['SECRET_KEY'], algorithms=['HS256'])
            user_id = payload['id']
            g.user = User.find(user_id)
            if g.user is None:
               abort(404, message="The user id is invalid")
            return f(*args, **kwargs)
        except JWTError as e:
            abort(400, message="There was a problem while trying to parse your token -> {}".format(e.message))
    return func

def validate_user(f):
    '''
    This decorate ensures that the user logged in is the actually the same user we're operating on
    '''
    @wraps(f)
    def func(*args, **kwargs):
        user_id = kwargs.get('user_id')
        if user_id != g.user['id']:
            abort(404, message="You do not have permission to the resource you are trying to access")
        return f(*args, **kwargs)
    return func

def belongs_to_user(f):
    '''
    This decorator ensures that the file we're trying to access actually belongs to us
    '''
    @wraps(f)
    def func(*args, **kwargs):
        file_id = kwargs.get('file_id')
        user_id = kwargs.get('user_id')
        file = File.find(file_id, True)
        if not file or file['creator'] != user_id:
            abort(404, message="The file you are trying to access was not found")
        g.file = file
        return f(*args, **kwargs)
    return func
```

- The `login_required` decorator is used to validate that users are actually logged in before accessing the method's functionality. We use this decorator to protect certain endpoints by decoding the token to ensure it's validity. We get the `id` field stored in the token and try to retrieve the corresponding user object. We also store this object in `g.user` for access by within the method definition.
- Similarly, we create the `validate_user` decorator which ensures that no other logged in user can access URL patterns labelled with another user's ID. This validation is purely based on the information in the URL.
- Finally, the `belongs_to_user` decorator ensures that the only the user who created a file can access it. This decorator actually checks the `creator` field in the file document against the `user_id` supplied.

Here are the views for creation of new files and listing of files.

```
class CreateList(Resource):
    @login_required
    @validate_user
    @marshal_with(file_array_serializer)
    def get(self, user_id):
        try:
            return File.filter({'creator': user_id, 'parent_id': '0'})
        except Exception as e:
            abort(500, message="There was an error while trying to get your files --> {}".format(e.message))

    @login_required
    @validate_user
    @marshal_with(file_serializer)
    def post(self, user_id):
        try:
            parser = reqparse.RequestParser()
            parser.add_argument('name', type=str, help="This should be the folder name if creating a folder")
            parser.add_argument('parent_id', type=str, help='This should be the parent folder id')
            parser.add_argument('is_folder', type=bool, help="This indicates whether you are trying to create a folder or not")
            
            args = parser.parse_args()

            name = args.get('name', None)
            parent_id = args.get('parent_id', None)
            is_folder =  args.get('is_folder', False)

            parent = None

            # Are we adding this to a parent folder?
            if parent_id is not None:
                parent = File.find(parent_id)
                if parent is None:
                    raise Exception("This folder does not exist")
                if not parent['is_folder']:
                    raise Exception("Select a valid folder to upload to")

            # Are we creating a folder?
            if is_folder:
                if name is None:
                    raise Exception("You need to specify a name for this folder")

                return Folder.create(
                    name=name,
                    parent=parent,
                    is_folder=is_folder,
                    creator=user_id
                )
            else:
                files = request.files['file']

                if files and is_allowed(files.filename):
                    _dir = os.path.join(BASE_DIR, 'upload/{}/'.format(user_id))

                    if not os.path.isdir(_dir):
                        os.mkdir(_dir)

                    filename = secure_filename(files.filename)
                    to_path = os.path.join(_dir, filename)
                    files.save(to_path)
                    fileuri = os.path.join('upload/{}/'.format(user_id), filename)
                    filesize = os.path.getsize(to_path)

                    return File.create(
                        name=filename,
                        uri=fileuri,
                        size=filesize,
                        parent=parent,
                        creator=user_id
                    )
                raise Exception("You did not supply a valid file in your request")
        except Exception as e:
            abort(500, message="There was an error while processing your request --> {}".format(e.message))
```

The listing method is pretty straightforward, all we do is filter the table for all files created by a certain user and stored in the root directory. We return this data for this endpoint and throw an exception if there are any errors.

For the create action, we do a bunch of things. For this tutorial, we're assuming that files and folders will be created with the same endpoint. For files, we will need to supply the file as well as a `parent_id` if we are uploading it in a folder. For folders, we will need a name, and a `parent_id` value, again if we are creating this within another folder. For folders, we also need to send an `is_folder` field with our request to specify that we are creating a folder.

If we are going to store this within a folder, we have to ensure that the folder exists and is a valid folder. We also ensure that we are supplying a name field if we are creating a folder.

For file creation, we upload the file into a folder specifically named for the different users as mentioned before. In our case, we are using the pattern `/upload/<user_id>` for the different user file directories. We have also used the file information to populate the document we're going to be storing in the table.

We conclude by calling the respective methods for file and folder creation - `File.create()` and `Folder.create()` respectively.

Notice that we have used the `marshal_with` decorator available with Flask-RESTful. This decorator is used to format the response object and to indicate the different field names and types that we'll be returning. See the definition of the `file_array_serializer` and `file_serializer` below:

```
file_array_serializer = {
    'id': fields.String,
    'name': fields.String,
    'size': fields.Integer,
    'uri': fields.String,
    'is_folder': fields.Boolean,
    'parent_id': fields.String,
    'creator': fields.String,
    'date_created': fields.DateTime(dt_format=  'rfc822'),
    'date_modified': fields.DateTime(dt_format='rfc822'),
}

file_serializer = {
    'id': fields.String,
    'name': fields.String,
    'size': fields.Integer,
    'uri': fields.String,
    'is_folder': fields.Boolean,
    'objects': fields.Nested(file_array_serializer, default=[]),
    'parent_id': fields.String,
    'creator': fields.String,
    'date_created': fields.DateTime(dt_format='rfc822'),
    'date_modified': fields.DateTime(dt_format='rfc822'),
}
```

This can be added at the top of the `/api/controllers/files.py` module or in a separate `/api/utils/serializers.py` module.

The difference between both serializers is that the file serializer includes the objects array in the response. We use the `file_array_serializer` for list responses while we use the `file_serializer` for object responses.

We have also made use of a function called `is_allowed()` to help ensure that we support all the files that we are uploading. We created a list called `ALLOWED_EXTENSIONS` to contain the list of all the allowed extensions.

```
ALLOWED_EXTENSIONS = set(['txt', 'pdf', 'png', 'jpg', 'jpeg', 'gif'])

def is_allowed(filename):
    return '.' in filename and \
           filename.rsplit('.', 1)[1] in ALLOWED_EXTENSIONS
```

Finally, we conclude by adding in the resource class for `ViewEditDelete` also in the `/api/controllers/files.py` module.

```
class ViewEditDelete(Resource):
    @login_required
    @validate_user
    @belongs_to_user
    @marshal_with(file_serializer)
    def get(self, user_id, file_id):
        try:
            should_download = request.args.get('download', False)
            if should_download == 'true':
                parts = os.path.split(g.file['uri'])
                return send_from_directory(directory=parts[0], filename=parts[1])
            return g.file
        except Exception as e:
            abort(500, message="There was an while processing your request --> {}".format(e.message))

    @login_required
    @validate_user
    @belongs_to_user
    @marshal_with(file_serializer)
    def put(self, user_id, file_id):
        try:
            update_fields = {}
            parser = reqparse.RequestParser()

            parser.add_argument('name', type=str, help="New name for the file/folder")
            parser.add_argument('parent_id', type=str, help="New parent folder for the file/folder")

            args = parser.parse_args()

            name = args.get('name', None)
            parent_id = args.get('parent_id', None)

            if name is not None:
                update_fields['name'] = name

            if parent_id is not None and g.file['parent_id'] != parent_id:
                if parent_id != '0'
                    folder_access = Folder.filter({'id': parent_id, 'creator': user_id})
                    if not folder_access:
                        abort(404, message="You don't have access to the folder you're trying to move this object to")

                if g.file['is_folder']:
                    update_fields['tag'] = g.file['id'] if parent_id == '0' else '{}#{}'.format(folder_access['tag'], folder['last_index'])
                    Folder.move(g.file, folder_access)
                else:
                    File.move(g.file, folder_access)

                update_fields['parent_id'] = parent_id

            if g.file['is_folder']:
                Folder.update(file_id, update_fields)
            else:
                File.update(file_id, update_fields)

            return File.find(file_id)
        except Exception as e:
            abort(500, message="There was an while processing your request --> {}".format(e.message))

    @login_required
    @validate_user
    @belongs_to_user
    def delete(self, user_id, file_id):
        try:
            hard_delete = request.args.get('hard_delete', False)
            if not g.file['is_folder']:
                if hard_delete == 'true':
                    os.remove(g.file['uri'])
                    File.delete(file_id)
                else:
                    File.update(file_id, {'status': False})
            else:
                if hard_delete == 'true':
                    folders = Folder.filter(lambda folder: folder['tag'].startswith(g.file['tag']))
                    for folder in folders:
                        files = File.filter({'parent_id': folder['id'], 'is_folder': False })
                        File.delete_where({'parent_id': folder['id'], 'is_folder': False })
                        for f in files:
                            os.remove(f['uri'])
                else:
                    File.update(file_id, {'status': False})
                    File.update_where({'parent_id': file_id}, {'status': False})
            return "File has been deleted successfully", 204
        except:
            abort(500, message="There was an error while processing your request --> {}".format(e.message)) 
```

We created a `get()` method which returns a single file or folder object based on the id. For folders, it includes listing information. You can see how this is done if you look at the `belongs_to_user` decorator. For files, we have included a query parameter `should_download` to be set to `true` if we want to download a file.

The `put()` method takes care of updating file and folder information. This also includes moving files and folders. Moving files is triggered by updating the `parent_id` field for a file/folder. The logic for both have been covered in the `move()` methods for the file and folder models.

The `delete()` method also comes a query parameter which specifies whether or not we want to perform a hard delete. For hard delete, records are removed from the database and the files are deleted from the file system. For soft delete, we only update the file `status` field to false.

We have created new methods here called `update_where()` and `delete_where()` in the `RethinkDBModel` class for deleting and updating a filtered set from the table:

```
@classmethod
def update_where(cls, predicate, fields):
    status = r.table(cls._table).filter(predicate).update(fields).run(conn)
    if status['errors']:
        raise DatabaseProcessError("Could not complete the update action")
    return True

@classmethod
def delete_where(cls, predicate):
    status = r.table(cls._table).filter(predicate).delete().run(conn)
    if status['errors']:
        raise DatabaseProcessError("Could not complete the delete action")
    return True
```

And that's it! We're done with our file storage API. Run the API to see it in action.

My next tutorial will cover front-end development on our file storage API using VueJS.

You can check out the codebase for the project [here](https://github.com/andela-cnnadi/papers). Show some love by starring the repository.

Feel free to send me your feedback and thoughts about this tutorial via email or comments. If you found it instructive and handy for learning Flask, Python, or RethinkDB, please favorite this tutorial as well!

