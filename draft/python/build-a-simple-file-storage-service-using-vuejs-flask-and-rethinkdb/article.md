In this tutorial, I shall be showing you how you can build a simple file storage service. We shall be making use of VueJS for the frontend, Flask for the backend and RethinkDB for database storage. I will be introducing a number of tools as we go on so stay tuned.

In the first part of this tutorial series, I will be focusing on the building the backend. As we go on, you will discover how you can implement some of the principles taught here in your current workflow either as a Python developer, Flask developer or even as a programmer in general.

We shall start by building out our API. Using this file storage service, as a user, we should be able to:
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

OK, we're now done with figuring out the API endpoints we need to create this app. Now, we need to start the actual process of creating the endpoints. So let's get to it.

## Setup

You should start by creating a project directory, obviously. This is the recommended structure for our application:

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

We will put in our routes and app creation function in `/api/__init__.py`. This way, we are able to use the function to create multiple app instances with different configurations. This is especially helpful when you are writing tests for your application.

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

Here, we have used the `flask_script.Manager` class to abstract out running of the server and enable us add new commands for the CLI. The migrate command there will be used to automatically create all the tables required by our models. This is a simplified solution for now. If all is well, you should not have any errors.


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

I'm sure a lot of people are wondering why I created a blank `RethinkDBModel` class. Well, there might be a coouple of things we might want to share across model classes and so this class is here in case this is needed.

Our `User` class will inherit from this base class and in it, we will have a few functions which we shall be using to interact with the database from the controllers.

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
            'date_created': r.now(),
            'date_modified': r.now()
        }
        r.table(cls._table).insert(doc).run(conn)
```
Here, we are making use of the `classmethod` decorator. This decorator enables us have access to the class instance from within the method body. We will be using the class instance to access the `_table` property from within the method. The `_table` stores the table name for that model.

We have also in here added code for making sure that the `password` and `password_conf` fields are the same. If this happens, a `ValidationError` will be thrown. The exceptions will be stored in the `/api/utils/errors.py` module. Find the definition of `ValidationError` below:

```
class ValidationError(Exception):
    pass
```

We're using named exceptions here because it's easier to track.

If there are no issues, we hash the password and create the document as a dictionary. Notice how we used `r.now()` here? This function gets us the current timestamp. There were issues faced when I used `datetime.datetime.now()`. This issue was because since RethinkDB noticed that I was sending a date, it was automatically trying to infer the datatype for that field in the document. Unfortunately, it requires Time zone information in addition to the date and time information. The Python function does not supply this for us by default unless we specify this as a parameter to the `now()` function (See [here](https://www.rethinkdb.com/docs/dates-and-times/python/) for more). The `r.now()` method mitigates this challenge and allows us to get the correct server time with the simple method call.

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
            'date_created': r.now(),
            'date_modified': r.now()
        }
        r.table(cls._table).insert(doc).run(conn)
    
    @staticmethod
    def hash_password(password):
        return pbkdf2_sha256.encrypt(password, rounds=200000, salt_size=16)

    @staticmethod
    def verify_password(password, _hash):
        return pbkdf2_sha256.verify(password, _hash)

```
The `pbkdf2_sha256.encrypt()` method is called with the password and values for `rounds` and `salt_size`. See [here](http://pythonhosted.org/passlib/lib/passlib.hash.pbkdf2_digest.html) for details on how you can customize your encryption and how the library works. Just to give some context as to why the decision to use `PBKDF2`: 

> Security-wise, PBKDF2 is currently one of the leading key derivation functions, and has no known security issues.

-- _Quotes from the passlib documentation_

The `verify_password` method will be called using the password string and a hash. It will return true or false if the password is valid.

We will now move on the the `validate()` function. This function will be called in the login method with the email address and password. The function will check that the document exists using the `email` field as index and then compare the password hash against the password supplied.

In addition to that, since we're going to be making using of JWT for token-based authentication, we will be generating a token if the user supplies valid information. This is how the entire models.py will look like when we're done adding in logic for that.

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
            'date_created': r.now(),
            'date_modified': r.now()
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
            raise ValidationError("The password you inputed was incorrect.")
    
    @staticmethod
    def hash_password(password):
        return pbkdf2_sha256.encrypt(password, rounds=200000, salt_size=16)

    @staticmethod
    def verify_password(password, _hash):
        return pbkdf2_sha256.verify(password, _hash)
```

A couple of things to take note of in the `validate()` method. Firstly, the `filter()` function was used here on the table object. This command takes in a dictionary that is used to search through the table. This function in some cases can also take a predicate. This predicate can be a lamda function and will be used similarly to what is done with the python `filter` function or any other function that takes a function as an argument. The function returns a cursor that can be used to access all the documents that are returned by the query. The cursor is iterable and as such we can iterate the cursor object using the `for...in` loop. In this case, we have chosen to convert the iterable object to a list using the Python list function.

As you would expect, we basically do two things here. We want to know if the email address exists at all and then if the password is correct. For the first part, we basically count the collection. If this is empty, we raise an error. For the second part, we call the `verify_password()` function to compare the password supplied with the hash in the database. We raise an exception if these don't match.

Another important thing to notice here is how we have used the `jwt.encode()` method to create a JWT token and return it to the controller. This method is fairly straightforward and you can see the documentation [here](https://github.com/mpdavis/python-jose).

So this about does it for the model. Let's move on to the controllers. We have, in this model, tried to obey the principle of having Fat models and Slim controllers. Most of the logic is in the models. This way, in our controllers, we will try to focus on routing and error reporting to the API end user.

## Authentication Controller

For the authentication controller, we need to add in Flask RESTful resource sub classes. If you've ever done any Django web development, it's similar to class-based views. It's simply created as a subclass of the `flask_restful.Resource` class. Your subclasses will have methods that map to respective HTTP verbs. For instance, if we wanted to implement a GET action, we will be creating a `get()` method in our Resource subclass. The process is completed by mapping URLs to the respective classes using the `api.add_resource()` method.

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

Now lets head back to the controller file to add in some logic. The logic required here is similar to what you will probably do with most authentication systems using most programming languages.

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

As mentioned earlier, majority of the logic and database interaction has been pushed to the model. To summarize what was done, for the login controller `AuthLogin`, we have created a post function which accepts the e-mail address and password, validates the fields using `reqparse` and then calls the `User.validate()` which validates the information sent and returns a toke. If an error occurs, we are going to catch it and respond with an error message.

Similarly, for the `AuthRegister`, we collect information from the user and call a model `create()` function. In this case, we are collection the email address, password, password confirm and full name fields. We pass all these values to the `User.create()` function and as before, this function will throw an error if anything goes wrong.

All things being equal, everything should work just fine. Run the server using `python run.py runserver` to test it out. You should be able to access the two endpoints that we've created here and it should work very well.

Next up, we'll be creating the models for our files.

## File models

We'll be creating very simple models for working with the files. Things to consider here include the fact that we need to have as much information about files stored in the database. The database records will be accessing the files stored in the file system.

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

Here we have created a general boilerplate with all the methods we need for working with Files. The `CreateList` class, as the name implies, will be used for creating and listing files for a logged in user. The ViewEditDelete class, also as the name implies, will be used for viewing, editing and deleting files. The methods we're using in the classes correspond with the appropriate HTTP actions.

For the purpose of this tutorial, I will be doing a simplistic version without

Feel free to send in your feedback and thoughts about this article. In the next part of this tutorial, we shall be going over how to write scripts to handle updating the database structure and the remaining logic for handling file uploads and management.