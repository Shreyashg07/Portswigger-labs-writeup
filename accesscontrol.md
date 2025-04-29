# Access control vulnerabilities and privilege escalation

*In this section, we describe:*

    Privilege escalation.
    The types of vulnerabilities that can arise with access control.
    How to prevent access control vulnerabilities.

**What is access control?**

Access control is the application of constraints on who or what is authorized to perform actions or access resources. In the context of web applications, access control is dependent on authentication and session management:

- Authentication confirms that the user is who they say they are.
- Session management identifies which subsequent HTTP requests are being made by that same user.
- Access control determines whether the user is allowed to carry out the action that they are attempting to perform.

Broken access controls are common and often present a critical security vulnerability. Design and management of access controls is a complex and dynamic problem that applies business, organizational, and legal constraints to a technical implementation. Access control design decisions have to be made by humans so the potential for errors is high. 

---
**Vertical access controls**

Vertical access controls are mechanisms that restrict access to sensitive functionality to specific types of users.

With vertical access controls, different types of users have access to different application functions. For example, an administrator might be able to modify or delete any user's account, while an ordinary user has no access to these actions. Vertical access controls can be more fine-grained implementations of security models designed to enforce business policies such as separation of duties and least privilege.
---

**Horizontal access controls**

Horizontal access controls are mechanisms that restrict access to resources to specific users.

With horizontal access controls, different users have access to a subset of resources of the same type. For example, a banking application will allow a user to view transactions and make payments from their own accounts, but not the accounts of any other user.
---
**Context-dependent access controls**

Context-dependent access controls restrict access to functionality and resources based upon the state of the application or the user's interaction with it.

Context-dependent access controls prevent a user performing actions in the wrong order. For example, a retail website might prevent users from modifying the contents of their shopping cart after they have made payment. 
---
**Examples of broken access controls**

Broken access control vulnerabilities exist when a user can access resources or perform actions that they are not supposed to be able to.
1. Vertical Privilege Escalation
Occurs when a user accesses functionality beyond their role.
Example: A regular user accessing an admin page to delete accounts.

2. Unprotected Functionality
Sensitive functions are accessible by URL alone without checks.
Example:
https://insecure-website.com/admin
Even if not linked in the UI, users might find it via robots.txt or brute-force wordlists.

3. Security by Obscurity
Hiding admin pages with obscure URLs doesn’t protect them.
Example:
https://insecure-website.com/administrator-panel-yb556
If the URL is leaked in public JavaScript, anyone can find and access it.

---

## LAB 1 : Unprotected admin functionality

Solution : <br>


1. Go to the lab and view robots.txt by appending /robots.txt to the lab URL. Notice that the Disallow line discloses the path to the admin panel.
2. In the URL bar, replace /robots.txt with /administrator-panel to load the admin panel.
3. Delete carlos.

--- 
## LAB 2: Unprotected admin functionality with unpredictable URL

Solution :<br>


1. Review the lab home page's source using Burp Suite or your web browser's developer tools.
2. Observe that it contains some JavaScript that discloses the URL of the admin panel.
3. Load the admin panel and delete carlos.

---
**Parameter-based access control methods**

Some applications determine the user's access rights or role at login, and then store this information in a user-controllable location. This could be:

    A hidden field.
    A cookie.
    A preset query string parameter.

The application makes access control decisions based on the submitted value. For example:
```bash 
https://insecure-website.com/login/home.jsp?admin=true
https://insecure-website.com/login/home.jsp?role=1
```
This approach is insecure because a user can modify the value and access functionality they're not authorized to, such as administrative functions. 
---
## LAB 3: User role controlled by request parameter

This lab has an admin panel at /admin, which identifies administrators using a forgeable cookie.Solve the lab by accessing the admin panel and using it to delete the user carlos. You can log in to your own account using the following credentials: wiener:peter <br>
solution:<br>


1. Browse to /admin and observe that you can't access the admin panel.
2. Browse to the login page.
3. In Burp Proxy, turn interception on and enable response interception.
4. Complete and submit the login page, and forward the resulting request in Burp.
5. Observe that the response sets the cookie Admin=false. Change it to Admin=true.
6. Load the admin panel and delete carlos.
---
## Lab 4: User role can be modified in user profile

This lab has an admin panel at /admin. It's only accessible to logged-in users with a roleid of 2. Solve the lab by accessing the admin panel and using it to delete the user carlos.You can log in to your own account using the following credentials: wiener:peter <br>
Solution:<br>


1. Log in using the supplied credentials and access your account page.
2. Use the provided feature to update the email address associated with your account.
3. Observe that the response contains your role ID.
4. Send the email submission request to Burp Repeater, add "roleid":2 into the JSON in the request body, and resend it.
5. Observe that the response shows your roleid has changed to 2.
6. Browse to /admin and delete carlos.
---
**Broken Access Control from Platform Misconfiguration**<br>
Some apps enforce access control at the platform level, restricting URLs and HTTP methods based on user roles.<br>
Example rule:<br>
DENY: POST /admin/deleteUser for managers<br>

Issue:<br>
Many frameworks support headers like X-Original-URL or X-Rewrite-URL that override the original URL. If the app relies solely on front-end controls, an attacker can bypass restrictions with:

```bash
POST / HTTP/1.1  
X-Original-URL: /admin/deleteUser  
```
Result: Access control is bypassed due to trust in overridden headers.
--- 
## LAB 5 : Lab: URL-based access control can be circumvented

 This website has an unauthenticated admin panel at /admin, but a front-end system has been configured to block external access to that path. However, the back-end application is built on a framework that supports the X-Original-URL header.

To solve the lab, access the admin panel and delete the user carlos. <br>

solution:<br>


1. Try to load /admin and observe that you get blocked. Notice that the response is very plain, suggesting it may originate from a front-end system.
2. Send the request to Burp Repeater. Change the URL in the request line to / and add the HTTP header X-Original-URL: /invalid. Observe that the application returns a "not found" response. This indicates that the back-end system is processing the URL from the X-Original-URL header.
3. Change the value of the X-Original-URL header to /admin. Observe that you can now access the admin page.
4. To delete carlos, add ?username=carlos to the real query string, and change the X-Original-URL path to /admin/delete.

---
 An alternative attack relates to the HTTP method used in the request. The front-end controls described in the previous sections restrict access based on the URL and HTTP method. Some websites tolerate different HTTP request methods when performing an action. If an attacker can use the GET (or another) method to perform actions on a restricted URL, they can bypass the access control that is implemented at the platform layer.
 ---
 ## LAB 6 : Lab: Method-based access control can be circumvented

 This lab implements access controls based partly on the HTTP method of requests. You can familiarize yourself with the admin panel by logging in using the credentials administrator:admin.

To solve the lab, log in using the credentials wiener:peter and exploit the flawed access controls to promote yourself to become an administrator.

<br>



1. Log in using the admin credentials.
2.  Browse to the admin panel, promote carlos, and send the HTTP request to Burp Repeater.
3.  Open a private/incognito browser window, and log in with the non-admin credentials.
4.  Attempt to re-promote carlos with the non-admin user by copying that user's session cookie into the existing Burp Repeater request, and observe that the response says  "Unauthorized".
5.  Change the method from POST to POSTX and observe that the response changes to "missing parameter".
6.  Convert the request to use the GET method by right-clicking and selecting "Change request method".
7.  Change the username parameter to your username and resend the request.

--- 
**Broken Access Control via URL-Matching Discrepancies** <br>
Web servers may handle URLs inconsistently — tolerating differences in capitalization, file extensions (e.g., /admin/deleteUser.anything matching /admin/deleteUser in Spring before 5.3), or trailing slashes. If access controls are stricter than URL matching, attackers can bypass protections by altering URL formats.

**Horizontal Privilege Escalation**<br>
This occurs when a user accesses another user's resources by manipulating identifiers, like changing id=123 to another user's ID in a URL (e.g., https://insecure-website.com/myaccount?id=124), exposing unauthorized data.

---
## LAB 7 : User ID controlled by request parameter 

 This lab has a horizontal privilege escalation vulnerability on the user account page.
To solve the lab, obtain the API key for the user carlos and submit it as the solution.
You can log in to your own account using the following credentials: wiener:peter 

<br>
solution : 

1. Log in using the supplied credentials and go to your account page.
2. Note that the URL contains your username in the "id" parameter.
3. Send the request to Burp Repeater.
4. Change the "id" parameter to carlos.
5. Retrieve and submit the API key for carlos.

---

## LAB 8 :User ID controlled by request parameter, with unpredictable user IDs .

This lab has a horizontal privilege escalation vulnerability on the user account page, but identifies users with GUIDs.
To solve the lab, find the GUID for carlos, then submit his API key as the solution.
You can log in to your own account using the following credentials: wiener:peter 

<br>
Solution:<br>



1. Find a blog post by carlos.
2.  Click on carlos and observe that the URL contains his user ID. Make a note of this ID.
3.  Log in using the supplied credentials and access your account page.
4.  Change the "id" parameter to the saved user ID.
5.  Retrieve and submit the API key.

---
## Lab 9 : User ID controlled by request parameter with data leakage in redirect 

This lab contains an access control vulnerability where sensitive information is leaked in the body of a redirect response.
To solve the lab, obtain the API key for the user carlos and submit it as the solution.
You can log in to your own account using the following credentials: wiener:peter 

<br>
solution:<br>


1. Log in using the supplied credentials and access your account page.
2. Send the request to Burp Repeater.
3. Change the "id" parameter to carlos.
4. Observe that although the response is now redirecting you to the home page, it has a body containing the API key belonging to carlos.
5. Submit the API key.

---
**Horizontal to Vertical Privilege Escalation**

An attacker can escalate from horizontal to vertical privileges by targeting a higher-privilege user. For example, by accessing another user's account (id=456), the attacker might compromise an admin's account. This can expose admin functions, passwords, or the ability to change credentials—resulting in full privilege escalation.

---

## LAB 10 : Lab: User ID controlled by request parameter with password disclosure

This lab has user account page that contains the current user's existing password, prefilled in a masked input.
To solve the lab, retrieve the administrator's password, then use it to delete the user carlos.
You can log in to your own account using the following credentials: wiener:peter 

<br>
Solution:<br>


1. Log in using the supplied credentials and access the user account page.
2. Change the "id" parameter in the URL to administrator.
3. View the response in Burp and observe that it contains the administrator's password.
4. Log in to the administrator account and delete carlos.

---
**Insecure Direct Object References (IDOR)**

IDORs occur when user input is used to directly access objects (e.g., files, records) without proper authorization checks. Attackers can modify the input to access unauthorized resources. It's a common access control flaw, highlighted in the OWASP Top 10 since 2007.

---
## LAB 11 : Lab: Insecure direct object references

This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.
Solve the lab by finding the password for the user carlos, and logging into their account. 

<br>
Solution:<br>


1. Select the Live chat tab.
2. Send a message and then select View transcript.
3. Review the URL and observe that the transcripts are text files assigned a filename containing an incrementing number.
4. Change the filename to 1.txt and review the text. Notice a password within the chat transcript.
5. Return to the main lab page and log in using the stolen credentials.

---


**Access Control Vulnerabilities in Multi-Step Processes** 

Many sensitive web functions are divided into multiple steps, especially when:
Multiple user inputs or options need to be collected.
The action requires user confirmation before execution (e.g., updating user details, transferring funds).

Example Flow:

Step 1: Load a form with user-specific details.

Step 2: Submit modified data.

Step 3: Review and confirm the changes.

The Issue:
Developers may enforce access control on steps 1 and 2, but not on step 3, assuming that users will follow the intended sequence. However, an attacker could skip to step 3 by crafting a direct request with valid parameters—bypassing earlier checks and gaining unauthorized access to perform restricted actions.

Key Risk:
Partial protection in multi-step processes creates a weak link that attackers can exploit by directly targeting unprotected steps.

---
## Lab 12: Multi-step process with no access control on one step 

This lab has an admin panel with a flawed multi-step process for changing a user's role. You can familiarize yourself with the admin panel by logging in using the credentials administrator:admin.
To solve the lab, log in using the credentials wiener:peter and exploit the flawed access controls to promote yourself to become an administrator. 

<br>
solution:<br>


1. Log in using the admin credentials.
2. Browse to the admin panel, promote carlos, and send the confirmation HTTP request to Burp Repeater.
3. Open a private/incognito browser window, and log in with the non-admin credentials.
4. Copy the non-admin user's session cookie into the existing Repeater request, change the username to yours, and replay it.


---
**Referer-Based Access Control**


Some websites rely on the Referer HTTP header to control access, assuming that requests coming from specific pages (e.g., /admin) are trustworthy.

Example:

- /admin is properly protected with authentication.

- /admin/deleteUser is only accessible if the request’s Referer is /admin.

The Problem:
The Referer header is user-controllable—attackers can forge requests with a fake Referer, bypassing access controls and gaining unauthorized access to sensitive pages.

Key Risk:
Referer-based access control is insecure and easily bypassed, as it depends on a client-supplied header rather than server-side validation.

---
## Lab 13: Referer-based access control 

This lab controls access to certain admin functionality based on the Referer header. You can familiarize yourself with the admin panel by logging in using the credentials administrator:admin.
To solve the lab, log in using the credentials wiener:peter and exploit the flawed access controls to promote yourself to become an administrator. <br>

Solution:<br>


1. Log in using the admin credentials.
2. Browse to the admin panel, promote carlos, and send the HTTP request to Burp Repeater.
3. Open a private/incognito browser window, and log in with the non-admin credentials.
4. Browse to /admin-roles?username=carlos&action=upgrade and observe that the request is treated as unauthorized due to the absent Referer header.
5. Copy the non-admin user's session cookie into the existing Burp Repeater request, change the username to yours, and replay it.

---
*Location-Based Access Control**

Some applications restrict access based on the user's geographical location (e.g., banking or media services). These controls are often bypassed using VPNs, proxies, or spoofed geolocation data, making them unreliable as a sole access control method.

---
**Preventing Access Control Vulnerabilities***
To secure your application, follow these defense-in-depth principles:

Don’t rely on obfuscation (e.g., hidden URLs or Referer checks) as a security measure.

Deny access by default unless a resource is explicitly meant to be public.

Centralize access control logic using a consistent, app-wide enforcement mechanism.

Require explicit access declarations in code—developers should define who can access what.

Regularly test and audit access controls to ensure they're effective and correctly implemented.

---
Thank you <br>

shreyash ghare 
guthub :shreyashg07