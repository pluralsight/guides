In this tutorial I shall be showing you how you can build a simple file storage service. We shall be making use of VueJS for the frontend while we shall be making use of Flask and RethinkDB for the backend.

We shall start by building out our API. We want a simple service where we can create an account, create a folder, create subfolders, upload a file, edit a file's description and delete a file.

The API will have the following endpoints:
- `POST /api/v1/auth/login` - This endpoint will be used to login users
- `POST /api/v1/auth/register` - This endpoint will be used to register users
- `/api/v1/user/<user_id>/files/`
    - `GET` - This endpoint will be used to list files for a user with user id `user_id`
    - `POST` - This endpoint will be used for creating new files for the user with user id `user_id`
- `/api/v1/user/<user_id>/files/<file_id>`
    - `GET` - This endpoint will be used to get a single file with id `file_id`
    - `PUT` - This endpoint will be used to edit a single file with id `file_id`
    - `DELETE` - This endpoint will be used to delete a single file with id `file_id`

OK, we're now done with figuring out the API endpoints we need to create this app. Now we need to start the actual process of creating the endpoints. So let's get to it.

You should obviously start by creating a project directory. This is the recommended structure for our application:

```
-- /app
   -- /connectors
   -- /resources
   -- __init__.py
-- /templates
-- /static
    -- /lib
    -- /js
    -- /css
    -- /img
-- index.html
-- run.py
```
To start, we shall be creating a new Flask instance in run.py and starting it up.

```
from flask import Flask
from flask_restful import Api

app = Flask(__name__)
api = Api(app)

if __name__ == '__main__':
    app.run()
```

We shall proceed by creating the controllers required for authentication.

```
from flask_restful import Resource
from app.connectors import User

class AuthLogin(Resource):
    def post(self):
        parser = reqparse.RequestParser()
        parser.add_argument('email', type=str, help='E-mail field')
        parser.add_argument('password', type=str, help='Password field')
        
        args = parser.parse_args()
        
        email = args.get('email')
        password = args.get('password')
        token = User.validate(email, password)
        return {'token': token}

class AuthRegister(Resource):
    def post(self):
        parser = reqparse.RequestParser()
        parser.add_argument('fullname', type=str, help='Full name field')
        parser.add_argument('email', type=str, help='E-mail field')
        parser.add_argument('password', type=str, help='Password field')
        parser.add_argument('password_conf', type=str, help='Confirm password field')
        
        args = parser.parse_args()
        
        email = args.get('email')
        password = args.get('password')
        password_conf = args.get('password_conf')
        fullname = args.get('fullname')
        
        try:
            User.create(email,password,password_conf,fullname)
            return {'message': 'Successfully created your account.'}
        except ValidationError as e:
            abort('There was an error while trying to create your account -> {}'.format(e.message()))
        
```

As you can see from what has been done above, first thing we did here is ensure we followed the principle of thin controller, fat model. We are pushing all the code for validation and data access to the models. To summarize what was done, for the login controller `AuthLogin` we have created a post function which accepts the e-mail address and password, validates the fields using `reqparse` and then call the `User.validate()` function to handle all the required operations for login. Similarly, for the `AuthRegister`, we collect information from the user and call a model function. In this case, we are collection the email address, password, password confirm and full name fields. We pass all these values to the `User.create()` function and expect it to work as usual.

As you can notice from both controllers, we are ensuring to put the model function block within a try...except block. This is because within the model, we shall be using exceptions to communicate the different states we are at in the function execution.