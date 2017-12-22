# Introduction

In this tutorial, you'll learn to build a simple web application using the following AWS Services:
- [AWS DynamoDB](https://aws.amazon.com/dynamodb/) as the database
- [AWS Lambda](https://aws.amazon.com/lambda/) to create functions that will read and write from/to the database
- [AWS API Gateway](https://aws.amazon.com/api-gateway/) to create the REST API that the web application will use
- [AWS S3](https://aws.amazon.com/s3/) to host the web application
- [AWS CloudFront](https://aws.amazon.com/cloudfront/) to deliver the web application from a location near to the user's location.

Here's the architecture diagram:

![architecture diagram](https://raw.githubusercontent.com/pluralsight/guides/master/images/2965d6f2-9e6b-4a3c-855e-97eef7f22c43.png)

For the lambda functions, the Node.js runtime with the Javascript SDK will be used.

Everything will be made from the AWS Management Console, without external frameworks, SDK, or command-line interfaces (CLI).

The sample web app allows you to create/edit/delete courses. This is how it looks like:

![demo](https://raw.githubusercontent.com/pluralsight/guides/master/images/db216707-2ba9-4229-acb7-d2c76ed9413b.gif)

It's a Single-page application made with React and Redux. The credit for this application goes to [Aries McRae](https://github.com/ariesmcrae), I just modified his [React CRUD boilerplate](https://github.com/ariesmcrae/reactjs-crud-boilerplate) for the purposes of this tutorial.

You can clone the app from [this GitHub repository](https://github.com/eh3rrera/react-app-frontend) and test it locally with the included Express server.

This tutorial won't explain how the app is made. There's no problem if you don't know about React and Redux, this tutorial will focus on building the API and setting up all the AWS services.

The API works with two data structures, `course` and `author`:
```
Course:
{
    id: "web-components-shadow-dom",
    title: "Web Component Fundamentals",
    watchHref: "http://www.pluralsight.com/courses/web-components-shadow-dom",
    authorId: "cory-house",
    length: "5:10",
    category: "HTML5"
}

Author:
{
  id: 'cory-house',
  firstName: 'Cory',
  lastName: 'House'
}
```

Of course, you'll need an AWS account. If you don't have one, [sign up here](https://aws.amazon.com/). Registration is free, and all the services used in this tutorial are within the [AWS Free Tier](https://aws.amazon.com/free/), but you'll need to enter a credit card number in case you exceed that free tier.

The most difficult step is the verification, in which an automated system will call you, and you have to enter a PIN code. It doesn't always work and probably you'll have to ask the support team to call you to verify your account.

Once you have created your account, your AWS account is automatically signed up for all services in AWS, but it's a good practice to create an admin user instead of using your root account for day-to-day operations. Follow the steps in [this guide](http://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) to do this.

Now let's start by creating the tables to store the app information in DynamoDB.

# Storing data in DynamoDB
DynamoDB is a fully-managed NoSQL database that stores the data in key-value pairs, like a JSON object:
```
{
  "ID": 1,
  "Title": "Introduction to Angular 5",
  "Category": "web-dev"
}
```

There are no schemas so every record can have a different structure. The only restriction is that the field(s) defined as the partition key must be present in all the records.

Based on this partition key, DynamoDB store data in different drives. An efficient distribution will make accessing the data as fast as possible, so it's important to choose a good partition key.

This way, the partition key can become the primary key, but you can also use a combination of a partition key and a sort key as a primary key. For example, if you have multiple records with the same course ID (the partition key), you can add a timestamp as a sort key to form a unique combination. In addition, you can also have secondary indexes for any other field (or combination of fields) to make queries more efficient.

DynamoDB gives you a lot of options. You can learn more about it in the [developer guide](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html), but for now, let's dive into creating a database for our API.

Open the *Services* menu and choose *DynamoDB*:

![choose dynamodb](https://raw.githubusercontent.com/pluralsight/guides/master/images/fa5ffa61-7359-4148-93b0-697f05027e1c.jpg)

Make sure you're in the correct AWS region (there's a DynamoDB database per region) and click on *Create table*:

![create table](https://raw.githubusercontent.com/pluralsight/guides/master/images/d69fa092-91c9-41bb-bfd4-5cd54a1f3a4f.jpg)

Enter the following information, leave the default settings checked and click on *Create*:
- Table name: `courses`
- Primary key: `id`

![course table](https://raw.githubusercontent.com/pluralsight/guides/master/images/69d43523-55df-4c77-9ace-20d2882ae0ec.png)

If you see the following message:

![autoscaling message](https://raw.githubusercontent.com/pluralsight/guides/master/images/4a52ebad-1df4-4da9-a3ee-34f74d2bf4f6.png)

Your AWS account does not currently have the *DynamoDBAutoscaleRole*, but there's no problem if you're not going to use the auto-scaling service or if you're not sure what it is. If you want to use it, follow the documentation link to add the *AmazonDynamoDBFullAccess* and a custom inline policy to create the autoscale role automatically when you create a new table with auto scaling the first time, it's all [in the documentation](http://docs.aws.amazon.com/amazondynamodb/latest/developerguide/AutoScaling.Console.html#AutoScaling.Permissions).

It might take a few seconds, but you should see a confirmation page like the following:

![table created](https://raw.githubusercontent.com/pluralsight/guides/master/images/8c343e09-e42d-4300-ac99-9a566777cd3c.png)

Take note of the table's Amazon Resource Name (ARN), you'll need it later.

In the *Items* tab you can query your table or add items:
![IMAGE](06)

I'm going to leave it empty, I'll add items to this table when the application is finished, so I'll go ahead and create a new table for the authors with the following information:
- Table name: `authors`
- Primary key: `id`
![IMAGE](07)

After the table is created, take note of its ARN.

Now, in the *Items* tab, create some authors, for example:
```
{
  id: 'cory-house',
  firstName: 'Cory',
  lastName: 'House'
},
{
  id: 'samer-buma',
  firstName: 'Samer',
  lastName: 'Buma'
},
{
  id: 'deborah-kurata',
  firstName: 'Deborah',
  lastName: 'Kurata'
}
```
![IMAGE](08)

Now let's create the lambda functions that will use these tables.


# Creating lambda functions

AWS Lambda is a service that allows you to run functions upon certain events, for example, when data is inserted in a DynamoDB table or when a file is uploaded to S3.

In this case, a lambda function will be run whenever a request hits one of the API endpoints you'll set up in the next section.

At the time of this writing, a lambda function can be written in Node.js, Python, Java, or C#. This tutorial will use Node.js.

You can learn more about AWS Lambda [in the developer documentation](http://docs.aws.amazon.com/lambda/latest/dg/welcome.html).

We'll create six lambda functions:
- `get-all-authors` to return all the users in the database
- `get-all-courses` to return all the courses in the database
- `get-course` to return only one course
- `save-course` to create a new course
- `update-course` to update a course
- `delete-course` to delete a course

Let's start by creating the `get-all-authors` function. Open the *Services* menu and choose *Lambda*:
![IMAGE](09)

Click on :Create a function*:
![IMAGE](10)

And then, with the *Author from scratch option* selected, enter the following:
- Name: `get-all-authors`
- Runtime: `Node.js 6.10` (or a superior version)
- Role: `Create a custom role`

When selecting the option `Create a custom role`, a new window will open to allow you to create a new role.

Alternatively, you could choose the option `Create a new role from a template(s)` and choose the policy template `Simple Microservice permissions`. This template will give you permissions to read, create, update, and delete items from any table:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "dynamodb:DeleteItem",
                "dynamodb:GetItem",
                "dynamodb:PutItem",
                "dynamodb:Scan",
                "dynamodb:UpdateItem"
            ],
            "Resource": "arn:aws:dynamodb:region:accountId:table/*"
        }
    ]
} 
```

However, this is not a good practice, service should have the minimum set of permissions to operate. That's way, you'll create a custom role with only the permissions the function requires.

So in the window to create a new role, enter `get-all-authors-lambda-role` as the role name:
![IMAGE](11)

At this point, it will only give you permissions to write to the CloudWatch logs, so click on the *Edit* link and modify the policy document so it looks like this:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": "dynamodb:Scan",
            "Resource": "<YOUR_ARN_FOR_THE_AUTHORS_TABLE>"
        }
    ]
}
```

Just replace the ARN of your author's table. This will give you permissions to perform a scan operation to get all the items of the author's table only.

Finally, click on the *Allow* button.

Back to the window to create the lambda function, choose the role `get-all-authors-lambda-role` in the *Existing role* option and click on *Create function*:
![IMAGE](12)

After the lambda function is created, scroll down to the *Function code* section:
![IMAGE](13)

Enter the following code:
```
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB({region: '<YOUR_AWS_REGION_CODE>', apiVersion: '2012-08-10'});

exports.handler = (event, context, callback) => {
    const params = {
      TableName: 'authors'
    };
    dynamodb.scan(params, (err, data) => {
      if(err) {
        console.log(err);
        callback(err);
      } else {
        callback(null, data);
      }
    });
};
```
First, you require the AWS SDK and create a DynamoDB instance passing the code of the region where the database is located, for example `us-east-2` for Ohio (find out your region code [here](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions)), and the API version (`2012-08-10` is the latest at the time of writing this tutorial).

The exported function receives three parameters:
- `event`, an object that is used to pass data to the handler.
- `context`, an object that provides runtime information about the Lambda function that is executing.
- `callback`, a function to return information to the caller (if it's called, otherwise the return value is `null`). It takes two parameters:
  - error, an object that provides the result of a failed execution. When a Lambda function succeeds, you can pass null as the first parameter.
  - result, an object that provides the result of a successful execution. The result provided must be JSON.stringify compatible. If an error is provided, this parameter is ignored.

Inside the `handler` function, you execute a [scan](http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html#scan-property) operation to get all the items of the `authors` table.

If there's an error, it's passed to the callback function. Otherwise, the callback is executed passing the data returned.

Keep in mind that the `scan` operation will return up to one MB of data. If there's more data available, a `LastEvaluatedKey` value is included in the results along with the number of items exceeding the limit to continue the scan in a subsequent operation.

You can know more about the available methods of the AWS Javascript SDK for DynamoDB in the [API documentation](http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html).

Now click *Save*. Then, if you click on the *Test* button, a window to configure some input properties will open:
![IMAGE](14)

This function doesn't take any parameters so give it a name and just click on *Create* and then on *Test* one more time.

You can check the execution results at the top of the page:
![IMAGE](15)

Or in the *Execution Results* tab below the code editor:
![IMAGE](16)

This is a sample output:
```
{
  "Items": [
    {
      "id": {
        "S": "cory-house"
      },
      "firstName": {
        "S": "Cory"
      },
      "lastName": {
        "S": "House"
      }
    },
    {
      "id": {
        "S": "samer-buma"
      },
      "firstName": {
        "S": "Samer"
      },
      "lastName": {
        "S": "Buma"
      }
    },
    {
      "id": {
        "S": "deborah-kurata"
      },
      "firstName": {
        "S": "Deborah"
      },
      "lastName": {
        "S": "Kurata"
      }
    }
  ],
  "Count": 3,
  "ScannedCount": 3
}
```

As you can see, every key/value pair inside `Items` also contains the type (`S` for `String`), so let's modify the format with a `map` function to only return the key and the value for each item.

Replace the code of the `else` block inside the callback function of `scan` with the following:
```
...

exports.handler = (event, context, callback) => {
    ...
    dynamodb.scan(params, (err, data) => {
      if(err) {
        ...
      } else {
        const authors = data.Items.map(item => {
          return { id: item.id.S, firstName: item.firstName.S, lastName: item.lastName.S };
        });
        callback(null, authors);
      }
    });
};
```

If you save and then test the function once again, you'll get as result a much cleaner object:
```
[
  {
    "id": "cory-house",
    "firstName": "Cory",
    "lastName": "House"
  },
  {
    "id": "samer-buma",
    "firstName": "Samer",
    "lastName": "Buma"
  },
  {
    "id": "deborah-kurata",
    "firstName": "Deborah",
    "lastName": "Kurata"
  }
]
```

Now follow the *Functions* link at the top of the page. Your function should appear on the dashboard:
![IMAGE](17)

Next, we need to create the rest of the functions with their corresponding roles and policies. Of course, you can create one role with a full access policy and use it for all your function, but remember, that is not the recommended approach.

You can also create the policies and roles from the Identity and Access Management (IAM) console with the visual editor, first creating a policy with required permission and then creating the role that will contain that policy.

But probably it's faster to enter the policy manually and have AWS create both the policy and the role for you. So here are the policies and the code for each function.

The policy to create a new course:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": "dynamodb:PutItem",
            "Resource": "<YOUR_ARN_FOR_THE_COURSES_TABLE>"
        }
    ]
}
```

And the code for the corresponding lambda function (`save-course`):
```
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB({region: '<YOUR_AWS_REGION_CODE>', apiVersion: '2012-08-10'});

const replaceAll = (str, find, replace) => {
    return str.replace(new RegExp(find, 'g'), replace);
}

exports.handler = (event, context, callback) => {
    const id = replaceAll(event.title, ' ', '-').toLowerCase();
    const params = {
      Item: {
        "id": {
          S: id
        },
        "title": {
          S: event.title
        },
        "watchHref": {
          S: `http://www.pluralsight.com/courses/${id}`
        },
        "authorId": {
          S: event.authorId
        },
        "length": {
          S: event.length
        },
        "category": {
          S: event.category
        }
      },
      TableName: 'courses'
    };
    dynamodb.putItem(params, (err, data) => {
      if(err) {
        console.log(err);
        callback(err);
      } else {
        callback(null, { id: params.Item.id.S, title: params.Item.title.S, watchHref: params.Item.watchHref.S, authorId: params.Item.authorId.S, length: params.Item.length.S, category: params.Item.category.S });
      }
    });
};
```

To test this function, you can configure a test event with the following data:
```
{
  "title": "Web Component Fundamentals",
  "authorId": "cory-house",
  "length": "5:10",
  "category": "HTML5"
}
```

To update a course we're also going to use the `putItem` operation so the policy is the same and you can reuse the role of the previous function.

The code for the corresponding lambda function (`update-course`) is also similar, the only difference is that the `id` and `watchHref` fields are not generated:
```
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB({region: '<YOUR_AWS_REGION_CODE>', apiVersion: '2012-08-10'});

exports.handler = (event, context, callback) => {
    const params = {
      Item: {
        "id": {
          S: event.id
        },
        "title": {
          S: event.title
        },
        "watchHref": {
          S: event.watchHref
        },
        "authorId": {
          S: event.authorId
        },
        "length": {
          S: event.length
        },
        "category": {
          S: event.category
        }
      },
      TableName: 'courses'
    };
    dynamodb.putItem(params, (err, data) => {
      if(err) {
        console.log(err);
        callback(err);
      } else {
        callback(null, { id: params.Item.id.S, title: params.Item.title.S, watchHref: params.Item.watchHref.S, authorId: params.Item.authorId.S, length: params.Item.length.S, category: params.Item.category.S });
      }
    });
};
```
To test this function, you can configure a test event with the following data:
```
{
  "id": "web-component-fundamentals",
  "title": "Web Component Fundamentals",
  "authorId": "cory-house",
  "length": "5:03",
  "category": "HTML5",
  "watchHref": "http://www.pluralsight.com/courses/web-components-shadow-dom"
}
```

The policy to get all the courses:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": "dynamodb:Scan",
            "Resource": "<YOUR_ARN_FOR_THE_COURSES_TABLE>"
        }
    ]
}
```

And the code for the corresponding lambda function (`get-all-courses`):
```
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB({region: '<YOUR_AWS_REGION_CODE>', apiVersion: '2012-08-10'});

exports.handler = (event, context, callback) => {
    const params = {
      TableName: 'courses'
    };
    dynamodb.scan(params, (err, data) => {
      if(err) {
        console.log(err);
        callback(err);
      } else {
        const courses = data.Items.map(item => {
          return { id: item.id.S, title: item.title.S, watchHref: item.watchHref.S, authorId: item.authorId.S, length: item.length.S, category: item.category.S };
        });
        callback(null, courses);
      }
    });
};
```

The policy to get one course:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": "dynamodb:GetItem",
            "Resource": "<YOUR_ARN_FOR_THE_COURSES_TABLE>"
        }
    ]
}
```

And the code for the corresponding lambda function (`get-course`):
```
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB({region: '<YOUR_AWS_REGION_CODE>', apiVersion: '2012-08-10'});

exports.handler = (event, context, callback) => {
    const params = {
      Key: {
        "id": {
          S: event.id
        }
      },
      TableName: 'courses'
    };
    dynamodb.getItem(params, (err, data) => {
      if(err) {
        console.log(err);
        callback(err);
      } else {
        callback(null, { id: data.Item.id.S, title: data.Item.title.S, watchHref: data.Item.watchHref.S, authorId: data.Item.authorId.S, length: data.Item.length.S, category: data.Item.category.S });
      }
    });
};
```

To test this function, you can configure a test event with the following data:
```
{
  "id": "web-component-fundamentals"
}
```

The policy to delete a course:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "arn:aws:logs:*:*:*"
        },
        {
            "Effect": "Allow",
            "Action": "dynamodb:DeleteItem",
            "Resource": "<YOUR_ARN_FOR_THE_COURSES_TABLE>"
        }
    ]
}
```

And the code for the corresponding lambda function (`delete-course`):
```
const AWS = require('aws-sdk');
const dynamodb = new AWS.DynamoDB({region: '<YOUR_AWS_REGION_CODE>', apiVersion: '2012-08-10'});

exports.handler = (event, context, callback) => {
    const params = {
      Key: {
        "id": {
          S: event.id
        }
      },
      TableName: 'courses'
    };
    dynamodb.deleteItem(params, (err, data) => {
      if(err) {
        console.log(err);
        callback(err);
      } else {
        callback(null, data);
      }
    });
};
```

To test this function, you can configure a test event with the following data:
```
{
  "id": "web-component-fundamentals"
}
```

And these are all the functions you'll need. Now let's expose them to the world via a REST API.

# Setting up API Gateway

API Gateway is a service that allows creating a REST API fully managed by AWS that acts as the front-end for other services.

Open the *Services* menu and choose *API Gateway*:
![IMAGE](18)

Press *Get started*:
![IMAGE](19)

You can create an API with a [Swagger](https://swagger.io/) file here we're going to create the API manually, so choose to create a *New API* and give it a name:
![IMAGE](20)

This will create a new API and you'll be ready to start adding resources (URL paths) and methods (GET, POST, etc.):
![IMAGE](21)

Now open the *Actions* menu and choose *Create Resource*. Enter a resource name and check the option *Enable API Gateway CORS*:
![IMAGE](22)

By default, the URL path will be created from the resource name. [Cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (CORS) is a mechanism that allows a web page to request resources hosted in another domain (which is going to be our case here). With this mechanism, the server sends some headers to tell the application that is OK to access the resources on that different server.

Next, click on *Create Resource*:
![IMAGE](23)

An `Options` endpoint is created because the client sends a preflight request to see if the resource is available and if it contains the CORS headers:
![IMAGE](24)

Now to start adding methods to the `/courses` endpoint, select it (don't select the root `/` endpoint), open the *Actions* menu and click on *Create Method* option:
![IMAGE](25)

We're going to start with the POST method, so select this option and click on the check icon next to the menu to set up the method:
![IMAGE](26)

As the *Integration type* choose *Lambda Function*, then the region where you created your lambda functions, and enter the name of the function that will be associated to this method (`save-course`):
![IMAGE](27)

Confirm the permission to invoke the lambda function:
![IMAGE](28)

By default, API Gateway will pass the request body to the lambda function and return to the client the object returned by the lambda function. Since the client is going to send exactly what the lambda function expects and the lambda function is going to do the same, there's no need to configure any request or response [body mapping templates](http://docs.aws.amazon.com/apigateway/latest/developerguide/models-mappings.html#models-mappings-mappings).

However, probably it's a good idea to use a [model](http://docs.aws.amazon.com/apigateway/latest/developerguide/models-mappings.html#models-mappings-models) to validate the request.

So go to the *Models* section and click on *Create*:
![IMAGE](29)

Give it a name, enter `application/json` as the *Content type*, and as the model schema, enter the following:
```
{
  "$schema": "http://json-schema.org/schema#",
  "title": "CourseInputModel",
  "type": "object",
  "properties": {
    "title": {"type": "string"},
    "authorId": {"type": "string"},
    "length": {"type": "string"},
    "category": {"type": "string"}
  },
  "required": ["title", "authorId", "length", "category"]
}
```
Models use the [JSON Schema specification](http://json-schema.org/). You can learn more about it [here](https://spacetelescope.github.io/understanding-json-schema/).

Click on *Create model* and back to the POST method click on *Method Request*:
![IMAGE](30)

In *Request Validator* choose the option *Validate body*, in the *Request Body* section add the model for the `application/json` content type and confirm all these choices:
![IMAGE](31)

Now if you click on *TEST*:
![IMAGE](32)

You can test the POST endpoint by entering a request body like:
```
{
  "title": "Web Component Fundamentals",
  "authorId": "cory-house",
  "length": "5:03",
  "category": "HTML5"
}
```

Everything should work correctly:
![IMAGE](33)

If you remove one or more attributes of the request, an error should be returned:
![IMAGE](34)

Moving on, we are also going to need a path like `courses/web-component-fundamentals`, where `web-component-fundamentals` corresponds to the ID of the course we want to update, delete or select one course.

Select the `course` resource and from the *Actions* menu, create another resource, this time just adding a path variable using brackets (and enabling CORS):
![IMAGE](35)

Click on *Create resource* to create it:
![IMAGE](36)

Next, create a PUT method and link it to the `update-course` lambda function:
![IMAGE](37)

Now the thing with this operation is that the ID of the course is in the URL and the rest of the attributes are in the request body.

In this case, we'll have to use a body mapping template to send all the information as the lambda function requires it.

So select the PUT method and click on *Integration Request*:
![IMAGE](38)

Then, in the *Body Mapping Templates* section, choose the option *When there are no templates defined*, add a mapping template for the content-type `application/json`, click on the check icon next to it, and enter the following template:
```
{
  "id": "$input.params('id')",
  "title" : $input.json('$.title'),
  "authorId" : $input.json('$.authorId'),
  "length" : $input.json('$.length'),
  "category" : $input.json('$.category'),
  "watchHref" : $input.json('$.watchHref')
}
```

The mapping template uses the [Velocity Template Language](http://velocity.apache.org/engine/devel/vtl-reference-guide.html) (VTL). [Here's](http://docs.aws.amazon.com/apigateway/latest/developerguide/models-mappings.html) the link to the relevant section in the AWS documentation.

The `$input` variable gives you access to all the request data, while `$input.params()` to the request parameters from the path, query string, or headers (in that order) and `$input.json()` returns a JSON string from the request body. You can know more about the `$input` variable and their functions [here](http://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-mapping-template-reference.html#input-variable-reference).

Also, notice that the ID has to be "converted" to a string by enclosing it in quotation marks, while the rest of the data since it's already sent as a string, don't have to be converted.

Click *Save* and test it with the path variable `web-component-fundamentals` (assuming the record exists in your database) and the following request body:
```
{
  "title": "Web Component Fundamentals",
  "authorId": "cory-house",
  "length": "5:03",
  "category": "HTML5",
  "watchHref": "http://www.pluralsight.com/courses/web-components-shadow-dom"
}
```

Everything should work correctly:
![IMAGE](39)

You can optionally add a model to validate the request, just like in the case of the POST method.

Now, applying what you have learned, create a DELETE and GET method on the `/courses/{id}` resource, linking them to the `delete-course` and `get-course` functions and passing only the ID with the following template:
```
{
  "id": "$input.params('id')"
}
```

Don't forget to add a GET method on the `/courses` resource for the `get-all-courses` function.

And another resource, `/authors`, for the `get-all-authors` function.

When you're done, you should have something like this:
![IMAGE](40)

Now, we need to add the CORS header to all the methods (the CORS option you chose when creating a resource only adds the CORS headers for the preflight request). 

Select the `/authors` endpoint, click on *Actions* and choose *Enable CORS*:
![IMAGE](41)

Deselect the OPTIONS method, click on *Enable CORS and replace existing CORS headers*:
![IMAGE](42)

And confirm to add the CORS headers to the GET method:
![IMAGE](43)

Do the same for all the methods of the `/courses` and `/courses/{id}` resources (except for the OPTIONS method).

Finally, to deploy the API, go to the *Actions* menu and choose *Deploy API*:
![IMAGE](44)

Then, create a new stage, give it a name, optionally a description and click on *Deploy*:
![IMAGE](45)

After the API is deployed, you'll get the URL to invoke it:
![IMAGE](46)

For example, in my case, the root URL of my API is `https://hcqxdqvii3.execute-api.us-east-2.amazonaws.com/v1` and to get the course with ID `web-component-fundamentals`, I'll have to use the URL `https://hcqxdqvii3.execute-api.us-east-2.amazonaws.com/v1/courses/web-component-fundamentals`.

At this point, you can use [curl](https://curl.haxx.se/) or [Postman](https://www.getpostman.com/) to test the API:
![IMAGE](47)

Now, the only step missing is uploading a client app for our API to S3.

# Hosting the app in S3

Clone the app from [this GitHub repository](https://github.com/eh3rrera/react-app-frontend).

In a terminal window, go to the app directory and execute `npm install` to install the dependencies.

Next, go to `src/api/serverUrl.js`, and replace the exported value with the URL of your API.

If you want to test it locally, execute `npm start` on the terminal.

Then, execute `npm run build` to build a version of the app ready for production. You can find the output files in the `build` folder.

Now let's use Amazon S3 to host the app and go completely serverless.

Amazon S3 is a storage service that can also function as a web server.

In S3, you put your files in buckets. You can know more about S3 in [this developer guide](http://docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html), for now, let's create a bucket.

Open the *Services* menu and choose *S3*:
![IMAGE](48)

Click on the *Create bucket* button and enter a name and the region where your bucket will be created:
![IMAGE](49)

Make sure you don't put any spaces in the name. Also, the name has to be unique across all existing names in S3, so use your domain name or something unique.

Next, click on the *Create* button, we'll take the default values for the properties and permissions tabs.

Once the bucket is created, click on it, and go to the *Properties* tab:
![IMAGE](50)

Click on the *Static website hosting* option and select *Use this bucket to host a website*:
![IMAGE](51)

Since this is a single-page application, enter `index.html` as the *Index* and *Error* documents. Copy the URL that S3 gives you and click *Save*.

Next, in the *Permissions* tab, select *Bucket Policy*:
![IMAGE](52)

Enter the [following policy](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-2) to grant read-only permissions to your bucket, just change `examplebucket` by your bucket name (in my case `net.eherrera.serverless-demo`):
```
{
  "Version":"2012-10-17",
  "Statement":[
    {
      "Sid":"AddPerm",
      "Effect":"Allow",
      "Principal": "*",
      "Action":["s3:GetObject"],
      "Resource":["arn:aws:s3:::examplebucket/*"]
    }
  ]
}
```

Click on *Save* to apply the policy:
![IMAGE](53)

Now go to the *Overview* tab, click on the *Upload* button, drag the content of the `build` directory and click on *Upload*:
![IMAGE](54)

Once the upload has finished, go to your app using the URL you copied (or go back to the *Static website hosting* option in the *Properties* tab to get it):
![IMAGE](55)

Optionally, you can create another bucket to store the logs of the web server, you just have to enable the *Server access logging* option in the *Properties* tab.

However, one thing that is strongly recommended is to use CloudFront.

CloudFront is a Content Delivery Network (CDN) service that will copy your app to edge locations around the world to improve the speed in which your app is served to your users. You can learn more about it in [this developer guide](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/Introduction.html).

Open the *Services* menu and choose *CloudFront*:
![IMAGE](56)

Click on *Create Distribution* and under the *Web* section click on *Get Started*:
![IMAGE](57)

In the *Origin Domain Name* select the bucket that contains your app, *Origin ID* will be populated:
![IMAGE](58)

Scroll down and under *Distribution Settings*, set the *Default Root Object* value to `index.html` and click on *Create Distribution* at the bottom of the page.

Wait a few minutes until the deployment has finished, and then click on the ID of your distribution:
![IMAGE](59)

In the *General* tab, you'll find the new URL of your app in the *Domain Name* row:
![IMAGE](60)

And that's it, you can use the HTTPS protocol for your URL if you want.


# Conclusion

The idea of a serverless application is interesting:
- There's no need to manage any servers.
- The application can be scaled automatically and highly available.
- You pay only for the resources used and for the time the application is used.

In this tutorial, you have learned how to build a REST API using API Gateway and Lambda functions that stores and read database from a DynamoDB database, and it's consumed by a single-page application hosted in S3 and distributed over the world with CloudFront.

Of course, there's a lot more to learn, for example, how to authenticate users with [Amazon Cognito](https://aws.amazon.com/cognito/) or how to use a framework like [Serveless](https://serverless.com/) or a service like [AWS CloudFormation](https://aws.amazon.com/cloudformation/) to simplify the managing and deploying of serverless applications, but that can be material for other tutorials. Thanks for reading.
