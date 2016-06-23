### **Lets get started thinking about availability**

> *Availability of a system is typically measured as a factor of its reliability – as reliability increases, so does availability. - Wikipedia*

This word is very important when the most important thing is offer a reliable system.  But, how to offer that kind of system, how can I create/provide something that makes me feel confortable.

------------------------

### Features of a reliable system/app

* Always available
* A huge suite of test cases
* Integrated diagnostic tools
* Real time monitoring
* Restrict access from external clients
* **Log every potential error in real time**

------------------------

### Log every potential error in real time

At this point I want to explain why log your program is an important task when you deliver an app to a production environment.  

Logging allows you to know more about the behaviour and performance of your app to take decisions and provide the neccesary fixes to solve a problem.  Likewise, logging provides you information about the broken point of your app.

Nowdays to check the results of a logging process is very hard because every log resides either into a server, virtualization, virtual server or cloud service.

### Log program using Lonline

##### Introduction

Lonline allows to log your program into the cloud and is powered by Dynamicloud service. Lonline allows you to log your program's execution into the cloud to avoid server access and disk space usage.  Lonline provides 6 levels of logging and 2 methods to execute reports. Lonline is a library to log your program through a storing service called Dynamicloud. With Dynamicloud we can store data dynamically and very easy, Lonline allows you to set more features and this way log more than only text, level and program trace.

**This documentation has the following content:**

1. [Dependencies](#dependencies)
2. [Settings](#settings)
  1. [Installation](#installation)
  2. [Dynamicloud account](#dynamicloud-account)
3. [How to use](#how-to-use)
  1. [Log using the six levels](#log-using-the-six-levels)
  2. [Additional data](#additional-data)
  3. [Execute reports](#execute-reports)

##### **Dependencies**
**Lonline has one dependendency:** [Dynamicloud Nodejs API](https://github.com/dynamicloud/dynamicloud_nodejs_api "Dynamicloud Nodejs API")

[Dynamicloud](https://www.dynamicloud.org "Dynamicloud") provides APIs for Java and Ruby language, so you can use Loline in Java or Ruby programs.

##### **Installation**

You can install this module in your system using the npm command:
 
`npm install lonline`

##### **Settings**

Lonline needs a basic settings to be configured, the settings of Lonline are within a settings file.

**These are the settings of Lonline:**

```javascript
/**
 * Options Documentation:
 */
var ops = {
     //OPTIONAL: This is the current logger object you're using in your program.
     //Lonline will call the same function in legacy object, for example:
     //lonlineLogger.error("It couldn't process the data");
     // The line above will execute the method error from legacy object passing the log
     // and the error as follow:  legacyObject.error(log, error);
     legacy: myOwnLogger
     //OPTIONAL: This attribute indicates the path to load the settings, if this attribute is present all the
     //below attributes will be skipped.
     //This file must be a compatible json file.
     fileName: '../setting.json',
     //This attribute indicates the level of lonline
     //The following table is an explanation of levels in Lonline (Descriptions from the great Java Log4j library):
     //------------------------------------------------------------------------------------------------------------
     //| Level      | Activated levels           | Description                                                    |
     //------------------------------------------------------------------------------------------------------------
     //| fatal      | Fatal                      | Designates very severe error events that will presumably lead  |
     //|            |                            | the application to abort.                                      |
     //------------------------------------------------------------------------------------------------------------
     //| error      | Fatal, Error               | Designates error events that might still allow the application |
     //|            |                            | to continue running.                                           |
     //------------------------------------------------------------------------------------------------------------
     //| warn       | Fatal, Error, Warn         | Designates potentially harmful situations.                     |
     //------------------------------------------------------------------------------------------------------------
     //| info       | Fatal, Error, Warn, Info   | Designates informational messages that highlight the progress  |
     //|            |                            | of the application at coarse-grained level.                    |
     //------------------------------------------------------------------------------------------------------------
     //| debug      | Fatal, Error, Info, Warn   | Designates fine-grained informational events that are most     |
     //|            | Debug                      | useful to debug an application.                                |
     //------------------------------------------------------------------------------------------------------------
     //| trace      | All levels                 | Traces the code execution between methods, lines, etc.         |
     //------------------------------------------------------------------------------------------------------------
     //| off        | None                       | The highest possible rank and is intended to turn off logging. |
     //------------------------------------------------------------------------------------------------------------
     level: 'error',
     //Credentials for REST APIs
     //Go to https://www.dynamicloud.org/manage and get the API keys available in your profile
     //If you don't have an account in Dynamicloud, visit https://www.dynamicloud.org/signupform
     //You can easily use a social network to sign up
     csk: 'csk#...',
     aci: 'aci#...',
     //This is the model identifier for test and development environment
     //The model contains the structure to store logs into the cloud
     //For more information about models in Dynamicloud visit https://www.dynamicloud.org/documents/mfdoc
     modelIdentifier: 000,
     //OPTIONAL: Shows every warning like rejected request from Dynamicloud and other warnings in lonline
     warning: true,
     //OPTIONAL: report_limit indicates how many records lonline will execute to fetch data from Dynamicloud.
     //If you need to increase this value, please think about the available requests per month in your account.
     //Dynamicloud uses a limit of records per request, at this time the max records per request is 20.  So,
     //report_limit=100 are 5 request.
     reportLimit: 100,
     //Send the backtrace (the ordered method calls) of the log.  If you want to avoid this sets to false
     backtrace: true,
};
```

**Lonline allows you to put the settings into a file.  The file's path must be passed through the attribute `fileName`**

```javascript
 //OPTIONAL: This attribute indicates the path to load the settings, if this attribute is present all the
 //below attributes will be skipped.
 //This file must be a compatible json file.
 fileName: '../settings.json'
```

`settings.json`
```json
{
  "csk": "csk#...",
  "aci": "aci#...",
  "modelIdentifier": 000,
  "backtrace": true,
  "level": "trace"
}
```

##### **Dynamicloud account**

Lonline needs API credentials from a Dynamicloud account, these credentials allow Lonline to access your account's structure (Model).  The mandatory model in your account should be composed for a model with at least three fields.  For further information about models and fields in Dynamicloud visit its documentation at [Models & Fields](https://www.dynamicloud.org/documents/mfdoc "Dynamicloud documentation")

**We are going to explain step by step how to setup your account in Dynamicloud, trust us it's very easy:**

1. Sign up in Dynamicloud (You can use either Google, Linkedin or Github account to speed up the registration)
2. Click on **Add Field** link at your **Real time Dashboard**.  Here you need to add three fields:
  
a. **Fields:**
  
| Field identifier | Field label | Field comments | Field type | Is a required field in form? |
| --- | --- | --- | --- | --- |
| `lonlinetext` | Log text | Contains the trace of this log | Textarea | **Yes** |
| `lonlinelevel` | Log level | Contains the log level | Combobox | **Yes** |
| `lonlinetrace` | Complete Trace | Contains the complete trace of the log | Textarea | **No** |  
  
b. **`lonlinelevel` is a combobox.  You need to add the following options:**
  
| Value | Text |
| --- | --- |
| `fatal` | Fatal |
| `error` | Error |
| `warn` | Warn |
| `info` | Info |
| `debug` | Debug |
| `trace` | Trace |

**To add these options follow the steps below:**

1. Click on **Global Fields** link at your **Real time Dashboard**
2. In the table of fields find the row with the identifier `lonlinelevel`
3. Click on **Manage items** link.  An empty list of items will be shown, there click on **Add new** button and fill the value and text field
4. The step number three should be executed six times according to the available levels (Fatal, Error, Warn, Info, Debug and Trace).

c. **Add model**
  
A model is the cointainer of these fields, to add a model follow the next steps:

1. Click on **Add model** link at your **Real time Dashboard**
2. Fill the mandatory field name and set a description (Optional)
3. Press button Create
4. In the list of Models the Model box has a header with the model Id, copy that Id and put it in your `settings.json` file.

```json
{
  "modelIdentifier": "Enter_Model_Id"
}
```

##### The last step is to copy the API credentials (CSK and ACI keys) to put them in your `settings.json` file.

1. Click on **Your name link at left top of your account**
2. Copy the CSK and ACI keys and put them into your `settings.json` file.

```json
{
  "csk": "Enter_Client_Secret_Key",
  "aci": "Enter_API_Client_Id"
}
```

At this moment you have the necessary to start to log your program into the cloud.

#How to use
Lonline is easy to use, one line of code logs and stores into the cloud.

##### **Log using the six levels**

```javascript
var lonline = require('lonline');
var logger = lonline.getLogger({fileName: 'settings.json'});

logger.trace('Calling method Y');
logger.debug('Creating new object');
logger.info('Process has finished');
logger.warn("It could'n create this object, the process will try later.");
logger.error('Unable to load setting file');
logger.fatal('The app has crashed and its state is unavailable');
```

##### **Additional data**
Lonline allows you to log further data.  If you want to log more information (For example: The module in your application who has executed the log.) just pass a LonlineLog object with the attributes and values.  Remember that these additional attributes must match with the fields in your model, so you need to add these fields before to log.

**To log additional information, follow the code below:**

```javascript
var lonline = require('lonline');
var logger = lonline.getLogger({fileName: 'settings.json'});

logger.trace('Calling method Y', null/*This is the error object*/, {
    "module": "financialModule"
});
```

##### **Execute reports**
Lonline allows you to execute reports about the executed logs and count how many logs have been created so far.

```javascript
var lonline = require('lonline');

var today = new Date();

var fromDate = new Date(today.getFullYear(), today.getMonth() - 1, today.getDate(), 0, 0, 1, 0);
var tomorrow = new Date(today.getFullYear(), today.getMonth(), today.getDate() + 1, 23, 59, 59, 0);

var reporter = lonline.getReporter({fileName: './settings.json'});
reporter.fetch(reporter.ERROR, new Date(), new Date(), function(logs) {
    _.each(logs, function(log) {
        console.log(log['lonlinelevel']);
        console.log(log['lonlinetext']);
        console.log(log['lonlinetrace']);
        console.log(log['module']);
        .
        .
        .
        .
    });
});

reporter.count(reporter.FATAL, fromDate, tomorrow, function (count) {
    console.log(count > 0);
});
```
