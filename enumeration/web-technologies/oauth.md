# OAuth

## Authentication

### What is it?

Authentication is the process by which a system confirms the identity of a user or application. It's essentially all about **who you are**.

Targeting authentication mechanisms allow us to to impersonate users, admins, or systems and gain unauthorized access. Often, we look to attack logic issues and lack of brute-force protection.

Common targets in authentication attacks include:

* Passwords or passphrases
* Multi-Factor Authentication (MFA)
* Session tokens
* Cookies
* Recovery questions and answers

For more details on specific authentication attack techniques, see the relevant child pages.

## Attacking password-based authentication

### What is it?

Password-based authentication generally allows to register an account and set a password, or sometimes an account will be assigned to them by an administrator. Password-based authentication tends to be suseptible to brute-force attacks, account lockouts and credential stuffing attacks.

**A simple example**

* A vulnerable web application allows users to sign up and set a password.
* After 10 failed login attempts, an account is locked.
* If an attacker uses 9 common passwords against many user accounts, they will gain access to ones that chose weak or common passwords.

Broken authentication can often lead to:

* Account takeover
* Sensitive data exposure

**Other learning resources:**

* PortSwigger: [https://portswigger.net/web-security/authentication](https://portswigger.net/web-security/authentication)

**Writeups:**

_Have a good writeup & want to share it here? Drop me a message on LinkedIn._

### Checklist

* [ ] Can we enumerate user accounts?
  * [ ] Registration page
  * [ ] Login page
  * [ ] Password reset page
* [ ] Is there any brute-force protection?
  * [ ] Check for account lockouts
  * [ ] Check for rate limiting
  * [ ] Check for CAPTCHA
  * [ ] Check for MFA
* [ ] What is the password policy?
  * [ ] Check the strength requirements
  * [ ] Is the password stored securely? (E.g. if we reset, will it send us the cleartext password)
  * [ ] Is the password reset token sufficiently unique?
* [ ] Are credentials predictable?
  * [ ] Check for default credentials
  * [ ] Check for username conventions (E.g. firstname.lastname)
* [ ] Is autocomplete enabled on password fields?
* [ ] Check the password reset functionality
  * [ ] Knowledge-based questions
  * [ ] Token leakage via Referrer
  * [ ] Token predictability
* [ ] Is authentication happening client-side?
* [ ] Are there any backups or leaked files with creds?
* [ ] Is there remember me or auto login functionality?
  * [ ] Are the tokens for this predictable?
  * [ ] How long does the token remain valid?
* [ ] Are tokens or credentials passed via the URL?
* [ ] Are there CSRF tokens?

## Attacking MFA

### What is it?

Multi-Factor Authentication (MFA) is a method of confirming a user's identity by using multiple pieces of evidence (factors), typically something they know (like a password), something they have (like a physical token or a mobile device), and something they are (like biometric data).

**A simple example**

A web application requests a password (first factor - something the user knows), then a one-time password sent to a mobile device (second factor - something the user has). An attacker could attempt to bypass MFA by stealing both the user's password and the OTP, or by exploiting vulnerabilities in the MFA implementation.

Common MFA bypass techniques can include:

* Phishing attacks to collect both factors
* Exploiting insecure backup/recovery methods
* Man-in-the-middle attacks
* Exploiting implementation weaknesses

**Other learning resources:**

* OWASP: [https://owasp.org/www-community/controls/Multi-Factor\\\\\_Authentication](https://owasp.org/www-community/controls/Multi-Factor/_Authentication)
* Duo Security: [https://duo.com/docs/duosec-v1](https://duo.com/docs/duosec-v1)
* Google Authenticator: [https://github.com/google/google-authenticator](https://github.com/google/google-authenticator)

### Checklist

* [ ] Understand the MFA implementation
  * [ ] What factors are used?
  * [ ] What backup/recovery methods exist?
  * [ ] Is there a fall-back option to less secure methods?
* [ ] Go through the MFA processes
  * [ ] Initial enrollment process
  * [ ] Login process with MFA
  * [ ] Recovery/Backup process
  * [ ] Deactivation process
* [ ] Are there any implementation weaknesses?
  * [ ] Does the application allow "remember me" functionality?
  * [ ] Can OTPs be predicted or intercepted?
  * [ ] Are session tokens securely handled?
  * [ ] Is there a secure lockout mechanism after multiple failed attempts?
* [ ] Can we bypass MFA?
  * [ ] Can we bruteforce the token?
  * [ ] Exploiting insecure backup/recovery methods
  * [ ] Can a new device be added without proper verification?
  * [ ] Is there any notification on registration of a new device?
  * [ ] Can the notification be suppressed?
* [ ] Are there any backdoors?
  * [ ] Is there an alternative login flow that bypasses MFA?
  * [ ] Is there a less secure service that doesn't require MFA but grants similar access?
  * [ ] Are there any APIs or resources that do not enforce MFA?

### Explanation

```shellscript
# OAuth 2.0
https://oauth.net/2/
https://oauth.net/2/grant-types/authorization-code/

Flow:

1. MyWeb tried integrate with Twitter.
2. MyWeb request to Twitter if you authorize.
3. Prompt with a consent.
4. Once accepted Twitter send request redirect_uri with code and state.
5. MyWeb take code and it's own client_id and client_secret and ask server for access_token.
6. MyWeb call Twitter API with access_token.

Definitions:

- resource owner: The resource owner is the user/entity granting access to their protected resource, such as their Twitter account Tweets
- resource server: The resource server is the server handling authenticated requests after the application has obtained an access token on behalf of the resource owner . In the above example, this would be https://twitter.com
- client application: The client application is the application requesting authorization from the resource owner. In this example, this would be https://yourtweetreader.com.
- authorization server: The authorization server is the server issuing access tokens to the client application after successfully authenticating the resource owner and obtaining authorization. In the above example, this would be https://twitter.com
- client_id: The client_id is the identifier for the application. This is a public, non-secret unique identifier.
- client_secret: The client_secret is a secret known only to the application and the authorization server. This is used to generate access_tokens
- response_type: The response_type is a value to detail which type of token is being requested, such as code
- scope: The scope is the requested level of access the client application is requesting from the resource owner
- redirect_uri: The redirect_uri  is the URL the user is redirected to after the authorization is  complete. This usually must match the redirect URL that you have  previously registered with the service
- state: The state  parameter can persist data between the user being directed to the  authorization server and back again. It’s important that this is a  unique value as it serves as a CSRF protection mechanism if it contains a  unique or random value per request
- grant_type: The grant_type parameter explains what the grant type is, and which token is going to be returned
- code: This code is the authorization code received from the authorization server which will be in the query string parameter “code” in this request. This code is used in conjunction with the client_id and client_secret by the client application to fetch an access_token
- access_token: The access_token is the token that the client application uses to make API requests on behalf of a resource owner
- refresh_token: The refresh_token allows an application to obtain a new access_token without prompting the user
```

### Bugs

```shellscript
# Weak redirect_uri
1. Alter the redirect_uri URL with TLD aws.console.amazon.com/myservice -> aws.console.amazon.com
2. Finish OAuth flow and check if you're redirected to the TLD, then is vulnerable
3. Check your redirect is not to Referer header or other param

https://yourtweetreader.com/callback?redirectUrl=https://evil.com
https://www.target01.com/api/OAUTH/?next=https://www.target01.com//evil.com/
https://www.target01.com/api/OAUTH?next=https://www.target01.com%09.evil.com
https://www.target01.com/api/OAUTH/?next=https://www.target01.com%252e.evil.com
https://www.target01.com/api/OAUTH/?next=https://www.target01.com/project/team
http://target02.com/oauth?redirect_uri=https://evil.com[.target02.com/
https://www.target01.com/api/OAUTH/?next=https://yourtweetreader.com.evil.com
https://www.target.com/endpoint?u=https://EVILtwitter.com/

ffuf -w words.txt -u https://www.target.com/endpoint?u=https://www.FUZZ.com/ 

# Path traversal: https://yourtweetreader.com/callback/../redirect?url=https://evil.com

# HTML Injection and stealing tokens via referer header
Check referer header in the requests for sensitive info
   
# Access Token Stored in Browser History
Check browser history for sensitive info

# Improper handling of state parameter
Check lack of state parameter and is in url params and is passed to all the flow
Verifying State entropy
Check state is not reused
Remove state and URI and check request is invalid

# Access Token Stored in JavaScript

# Lack of verification
If not email verification is needed in account creation, register before the victim.
If not email verification in Oauth signing, register other app before the victim.

# Access token passed in request body
If the access token is passed in the request body at the time of allocating the access token to the web application there arises an attack scenario. 
An attacker can create a web application and register for an Oauth framework with a provider such as twitter or facebook. The attacker uses it as a malicious app for gaining access tokens. 
For example, a Hacker can build his own facebook app and get victim’s facebook access token and use that access token to login into victim account.

# Reusability of an Oauth access token
Replace the new Oauth access token with the old one and continue to the application. This should not be the case and is considered as a very bad practice.
```

### OAuth resources

```sh
https://owasp.org/www-pdf-archive/20151215-Top_X_OAuth_2_Hacks-asanso.pdf
https://medium.com/@lokeshdlk77/stealing-facebook-mailchimp-application-oauth-2-0-access-token-3af51f89f5b0
https://medium.com/a-bugz-life/the-wondeful-world-of-oauth-bug-bounty-edition-af3073b354c1
https://gauravnarwani.com/misconfigured-oauth-to-account-takeover/
https://medium.com/@Jacksonkv22/oauth-misconfiguration-lead-to-complete-account-takeover-c8e4e89a96a
https://medium.com/@logicbomb_1/bugbounty-user-account-takeover-i-just-need-your-email-id-to-login-into-your-shopping-portal-7fd4fdd6dd56
https://medium.com/@protector47/full-account-takeover-via-referrer-header-oauth-token-steal-open-redirect-vulnerability-chaining-324a14a1567
https://hackerone.com/reports/49759
https://hackerone.com/reports/131202
https://hackerone.com/reports/6017
https://hackerone.com/reports/7900
https://hackerone.com/reports/244958
https://hackerone.com/reports/405100
https://ysamm.com/?p=379
https://www.amolbaikar.com/facebook-oauth-framework-vulnerability/
https://medium.com/@godofdarkness.msf/mail-ru-ext-b-scope-account-takeover-1500-abdb1560e5f9
https://medium.com/@tristanfarkas/finding-a-security-bug-in-discord-and-what-it-taught-me-516cda561295
https://medium.com/@0xgaurang/case-study-oauth-misconfiguration-leads-to-account-takeover-d3621fe8308b
https://medium.com/@rootxharsh_90844/abusing-feature-to-steal-your-tokens-f15f78cebf74
http://blog.intothesymmetry.com/2014/02/oauth-2-attacks-and-bug-bounties.html
http://blog.intothesymmetry.com/2015/04/open-redirect-in-rfc6749-aka-oauth-20.html
https://www.veracode.com/blog/research/spring-social-core-vulnerability-disclosure
https://medium.com/@apkash8/oauth-and-security-7fddce2e1dc5
https://xploitprotocol.medium.com/exploiting-oauth-2-0-authorization-code-grants-379798888893
```

### OAuth scheme

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MMy6O6BnEn-0f2S_MJe%2F-MMy7cLlAshQPIGv5juw%2Fimagen.png?alt=media\&token=5aa368a5-31e5-485e-bb10-31baf02c5129)

### Code grant flow

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-MMy6O6BnEn-0f2S_MJe%2F-MMy7xfgLCrBzKMM3w5I%2Fimagen.png?alt=media\&token=0a5727f0-e51f-4676-bb29-2cfde8f6d2f6)

### OAuth Attack mindmap

![](https://1729840239-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-M5x1LJiRQvXWpt04_ee%2F-M9HNRDiAQWw3ZHqe5mR%2F-M9HNlKjBz6mJ5Avk7Ye%2Fphoto_2020-06-08_07-24-17.jpg?alt=media\&token=1333371c-80d5-419a-8128-46a3a745d39e)
