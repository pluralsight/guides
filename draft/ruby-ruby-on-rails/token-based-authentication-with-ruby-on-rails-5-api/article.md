With API-only applications gaining popularity and  Rails 5 being just around the corner, the most ubiquitous methods of authentication are now becoming token-based. In this tutorial, a short overview of token-based authentication will be given and implemented into a Rails 5 API-only application.


## What is token-based authentication?
 
 Token-based authentication is a method of handling authentication of users in applications. It is widely used in APIs - Facebook, Twitter, Github and other websites with exposed APIs handle the authentication in the token-based way. 
 Token-based authentication is an alternative to session-based authentication. The most notable difference between the two is that session-based authentication is persisted server-side i.e a record is created for each logged-in user. Token-based authentication, on the other hand, is stateless - it does not store anything on the server, but creates a unique encoded token that is checked every time a request is made. Since session-based authentication uses 


### How does it work?
The way token-based authentication works is simple: The user enters his/her credentials and sends a request to the server. If the credentials are correct, the server creates a unique HMACSHA256 encoded token also known as JSON web token (JWT). The server stores the JWT  in the database and sends it back to the client. The client stores the JWT and makes all subsequent requests to the server with the token attached. The server authenticates the user by comparing the JWT sent with the request to the one it has stored in the database. Here is a simple diagram of the process:

![token-based auth](http://i.imgur.com/xkvip2y.jpg)
### Benefits of token-based authentication
There are several benefits to using such approach. The main ones are:
Cross-domain / CORS: cookies + CORS don't play well across different domains. A token-based approach allows you to make AJAX calls to any server, on any domain because you use an HTTP header to transmit the user information.

* JWT are stateless there is no need to keep a session store, the token is a self-contanined entity that conveys all the user information. 


* Decoupling: you are not tied to a particular authentication scheme. The token might be generated anywhere, hence the API can be called from anywhere with a single way of authenticating all the calls.

* Mobile ready: Cookies are a problem when it comes to storing user information on native mobile applications. Adopting a token-based approach simplifies the process significantly.

* CSRF: Because the application does not rely on cookies for authentication, it is invulnerable cross site request attacks.

* Performance: In terms of server-side load, a network roundtrip (e.g. finding a session on database) is likely to take more time than calculating an HMACSHA256 code to validate a token and parsing its contents.

# Setting up a token-based authentication with Rails 5