In this tutorial, we're going to build a chat using Pusher's presence channels with the functionality to request e-signed Non-Disclosure Agreements (NDAs) to its members using the HelloSign API.

The stack will be the following:
- Java 7 or higher
- Maven as the build manager
- Spring Boot as the server-side framework
- H2 as in-memory database
- Thymeleaf as the server-side template engine
- jQuery and Handlebars for the client-side interaction

We're going to use Pusher and HelloSign webhooks to receive events from these APIs and we'll use ngrok to keep everything in a local environment.

The app's design is based on this [pen](http://codepen.io/drehimself/pen/KdXwxR) and it works in the following way. First, a user creates a chat:

![Create chat](https://raw.githubusercontent.com/pluralsight/guides/master/images/a4a6062a-a807-42c4-9720-afdff82beb11.gif)


Then, another user joins it:

![Join chat](https://raw.githubusercontent.com/pluralsight/guides/master/images/5d064ba2-a70a-4199-bad6-2a1b3c8701f5.gif)


At any time, the chat owner can send a request to sign an NDA:

![Sending NDA](https://raw.githubusercontent.com/pluralsight/guides/master/images/401eb9e9-e6aa-4468-9446-230a9724454e.gif)


When a member of the chat signs the NDA, a notification is sent with a link to view the signed document:

![Notification signed document](https://raw.githubusercontent.com/pluralsight/guides/master/images/cf249643-5131-47b5-b8fb-09720271e6ad.gif)



The entire source code of the final app is on [Github](https://github.com/).


# Requirements