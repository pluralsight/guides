### Learn how to build a basic “what happened today” conversational application that provides a dynamic and personalized response.

In this example, you’ll build a basic “what happened today” conversational application that provides a dynamic and personalized response. A conversational app contains intents, entities, and responses that you want to deliver to your user. Intents are the mechanisms that pick up what your user is requesting (using entities) and that direct the app to respond accordingly.

> Conversation.one is a free build-once-deploy-anywhere platform where your app can be deployed as an Alexa skill, Google Home action, Facebook Messenger bot, or a chatbot on your website.

For simple replies that don’t include information gathered outside of the conversation, you can define textual hard-coded responses. More advanced responses can be made using your own logic and integrations. In later sections, you’ll add responses gathered from an external API call. For now, let's cover the basics.

### Create a Conversational App
A Conversation.one application represents your conversational interface through Amazon Alexa, Google Home, or Facebook Messenger bot. To create an app:

1. If you don’t already have a Conversation.one account, sign up. If you have an account, log in.
2. Click on Create New App in the navigation. If it’s your first application, you will be redirected to the Create New App wizard automatically.
3. Select your industry and enter your company name. For the purpose of this tutorial, select Other for the industry and enter What Happened Today as your company name. Uncheck Add demo data to start with an empty application. Click Next to continue.
4. Select the devices you want to support with your app. For this demo, select Messenger, Amazon Alexa, Google Home, and Chatbot. Click Next to continue.
5. Your new application is ready. Click Customize to start developing your app.

### Create an Intent
An intent maps what a user says with what your app does. This first intent will cover when the user asks for information on what happened today.

To create an intent:

1. Create a new intent named whatHappenedToday.
2. In the User Says section, enter examples of what you might expect a user to ask for. The more examples you provide, the more ways a user can ask a question and the conversational app will understand. Enter these examples:
    - What happened today in history
    - What happened on this day in the past
    - What took place today
    - What occurred today
3. Click Save to apply your changes.

### Add Response
1. In the Response section, select Text and enter You created my first conversational app.
2. Click Save to apply your changes.

### Try It Out
Now that your app can understand basic requests from the user, try out what you have so far.

In the chat console on the right, type in a request. The request can be a little different than the examples you provided in the User Says section. This can be something like “what popped up today.” After you type the request, hit Send.

The chatbot will answer, “You created my first conversational app.”

### Add Code Response
1. In the Response section, select From Code and enter the code:
```javascript
var today = new Date();
var day = today.getDate();
var month = today.getMonth() + 1;
return fetch("http://history.muffinlabs.com/date/" + month + "/" + day)
    .then(function(res) {
        return res.json();
    }).then(function(json) {
        var events = json.data.Events;
        var randomEvent = Math.floor(Math.random() * (events.length));
        var selectedEvent = events[randomEvent];
        res.say('On this day,' + selectedEvent.year + ' - ' + selectedEvent.text.replace(/(\(.*\))/g, ''));
    });
```
2. The code above is running the following steps:
    1. Find today’s date.
    2. Call the History API using the Fetch API.
    3. Select a random event and return it using the res.say command.
3. Click Save to apply your changes.

### Try It Out
Now that your app can return real historical events to the user, try out what you have so far.

In the chat console on the right, type in a request. The request can be a little different than the examples you provided in the User Says section. This can be something like “what popped up today.” After you type the request, hit Send.

The chatbot will answer with some interesting event from the past. For example, “On this day, 1974: The Rumble in the Jungle boxing match between Muhammad Ali and George Foreman takes place in Kinshasa, Zaire. Ali wins by KO in the eighth round, regaining the title of World Heavyweight Champion and causing Foreman's first professional defeat.”

### Customize Your Authorization Page
1. Open the Authorization tab from the left-hand menu.
2. Inside, you can find an HTML template for your authorization page. You can change the look and feel, add or remove form fields as you wish.
3. To preview your authorization page, click the Preview Authorization Page in the editor’s top menu
4. Let’s do some changes to our authorization page. Change the body background to blue by locating the CSS code:
```CSS
body {
    background-image: url("https://167a9.https.cdn.softlayer.net/80167A9/s1.mob-server.com/login/osman-rana-293978.jpg");
    background-size: 120% 140%;
    background-repeat: no-repeat;
    background-position: bottom bottom;
    margin-top: 30px !important;
}
body {
    background-color: blue !important;
    margin-top: 30px !important;
}
```
5. And changing it to click Save, and reopen the preview page to view the changes. To validate the user’s credentials you will need to add a call to your authorization server in the function checkLogin and to call the function checkLoginOnSuccesscheckLoginOnFail on failure.
6. To store the user’s details in Conversation.one secured resource server, call the function authorizeSucess. You can see the example code in the provided template:
```javascript
var pin = $("#user_pincode").val();
var data = {};
data.firstname = $('#user_firstname').val();
data.lastname = $('#user_lastname').val();
data.name = $("#business_fields_name").val();
data.password = $("#business_fields_pass").val();
data.email = $("#user_email").val();
data.mobile = $("#user_mobile").intlTelInput("getNumber");
data.validationtest = "1234567890";
var resource_data = JSON.stringify(data);
authorizeSucess(data.name, resource_data, pin);
```

### Fetch User Data In Your Response Code
To access the user's data, use the function sdk.conversationHelper.getBackendContext. The function will verify that account linking is enabled and will fetch the users’ data from the secured resource server. On first access, the function will ask the user to provide the user selected PIN code.

1. Make sure you have an EnterPINCode intent. If not add one.
2. Edit the code response of the whatHappenedToday intent to the code:
```javascript
var backendcontext = sdk.conversationHelper.getBackendContext(context, req, res);
if (!backendcontext) return;
var today = new Date();
var day = today.getDate();
var month = today.getMonth() + 1;
return fetch("http://history.muffinlabs.com/date/" + month + "/" + day)
    .then(function(res) {
        return res.json();
    }).then(function(json) {
        var events = json.data.Events;
        var randomEvent = Math.floor(Math.random() * (events.length));
        var selectedEvent = events[randomEvent];
        res.say("Hi " + backendcontext.firstname + ".");
        res.say('On this day,' + selectedEvent.year + ' - ' + selectedEvent.text.replace(/(\(.*\))/g, ''));
    });
```

### Try It Out
Now that your app has account linking enabled and can fetch the stored users’ data, try out what you have so far.

In the chat console on the right, type in a request. The request can be a little different than the examples you provided in the User Says section. This can be something like “what popped up today.” After you type the request, hit Send.

The chatbot will ask you to link your account. Click on the Open log-in window button and fill in your data.

You can use any value for username and password. To verify those values, connect it to your own authentication server as described above.

Now, the chatbot will ask for your pre-selected PIN code, once provided you will get a personalized response from the chatbot.

What’s Next?
In the next part, you’ll use entities to extract parameter values from natural language inputs.