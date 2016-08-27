# Critical Security Areas
In this guide will talk about why Web security is so important and will understand primary risks that exposes website to attackers. [Daily DDos attacks](http://www.digitalattackmap.com/#anim=1&color=0&country=ALL&list=0&time=17033&view=map). Will look into OWASP Top-10 security risks and preventive measures to mitigate attacks.

### About OWASP

The __Open Web Application Security Project__ (OWASP) is a _non-profit_ educational
charity dedicated to enabling organizations to design, develop, acquire, operate, and maintain
secure software.[OWASP-Org](https://www.owasp.org)

OWASP is not affiliated with any technology company. Similar to many open­source software projects, OWASP produces many types of materials in a collaborative and open way. All OWASP tools, documents, forums, and chapters are free and open to
anyone interested in improving application security.

### Introduction to Websecurity risk

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/2bb93931-db94-4cf4-ac6d-82eb3dd560f4.jpg)

Websites and Web-Servers are prone to security risks and [threats](https://en.wikipedia.org/wiki/Web_threat) due to insecure software and lack of security principles. In recent years a range of different companies have suffered online attacks, including finance, defense, healthcare and other critical organisations worldwide. [Live Online-attacks](http://map.norsecorp.com/#/)

The goal of the OWASP  is to raise awareness about web application security by describing the most important areas of concern that software developers must be aware of.




### What are the [Top-10](https://www.owasp.org/index.php/Top_10_2013-Top_10) Vulnerablities ?

1. SQL-Injection
2. Broken Authentication and Session Management
3. Cross-Site Scripting (XSS)
4. Insecure Direct Object References
5. Security Misconfiguration
6. Sensitive Data Exposure
7. Missing Function Level Access Control
8. Cross-Site Request Forgery (CSRF)
9. Using Components with Known Vulnerabilities
10. Unvalidated Redirects and Forwards

We will cover each and every risk in detail below.

### __1. SQL-Injection__


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/c73b8611-3b98-4932-a81a-eedfa0fcaa05.png)


SQL injection (SQLi) refers to an injection attack wherein an attacker can execute malicious SQL statements (_malicious payload_) that control a web application's database server.

A successful SQL injection can read sensitive data from the database, modify database, execute administration operations on the database (such as shutdown the DBMS), recover the content of a given file present on the DBMS file system and in some cases can issue commands to the operating system.


Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
EASY | COMMON   | AVERAGE |SEVERE

**Example Scenario :**
The application uses untrusted data in the construction of the following vulnerable SQL call:
```
String query = "SELECT * FROM accounts WHERE custID='" + request.getParameter("id") + "'";
```
In this case the attacker modifies the ‘id’ parameter value in browser to send `' '` or `'1'='1`
For ex :
```
http://example.com/app/accountView?id=' or '1'='1
```
This changes the meaning of both query to return all the records from the accounts table.

**Preventive Measures **
