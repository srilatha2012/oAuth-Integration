# OAuth Vulnerabilities Reflection Lab

## Reflection Questions

## 1. CSRF and the state Parameter: In your own words, explain how an attacker could perform a Cross-Site Request Forgery (CSRF) attack on an OAuth flow. How does using the state parameter, as recommended, prevent this specific attack?

Click “Login with google” on a website
Normally the process:
1. The website sends the user to Google for Login
2. The user Logs in with their Google Account
3. Google sends the user back to the website with proof that the login was successful.
4. The website connects the user session with correct Google account

A CSRF attack can happen if the application does not properly verify the login request.
ex: 
1. The attacker first logs into the application using their own google account
2. Google sends back an oAuth authorization response to the attacker
3. The attacker copies that login response link and sends it to the victim
4. If the victim clicks the link, the application may think  the OAuth response belongs to the victim.
5. The victim may then unknowingly get connected to the attacker’s account
6. The victim might upload personal information or perform actions while thinking they are using their own account.

the `state` parameter will help to prevent this attack
Before sending the user to Google, the application creates a random `state` value and stores it temporarily in the user’s session.
After login, Google sends the same `state` value back to the application
The application then checks whether the returned `state` matches the original stored value.

- If the values match, the request is accepted.
- If the values do not match, the request is rejected.
This works like a temporary secret ID that helps the application confirm that the OAuth response really belongs to the correct user session and was not sent by an attacker.


## 2. Redirect URI Attacks: The article mentions that validating a redirect_uri by simply checking the domain or allowing subdomains is a common mistake. Describe a hypothetical scenario where a “leaky” redirect_uri validation (e.g., one that allows any path on a valid domain) could be exploited to steal an authorization code.

A redirect_uri is the URL where the OAuth provider (like Google) sends the user after successful login.
example: https://example.com/callback

After login, Google sends something like:
https://example.com/callback?code=ABC123

That code is very important because it can later be exchanged for an access token.

### Problem is; 
Some applications validate the redirect_uri incorrectly.
Instead of checking the exact URL, they only check:
 - the domain name (example.com)
        or
 - allow any path under the domain.

 ### Unsafe validation
 https://example.com/*      - This means ANY page under example.com is allowed.

 ### How Attacker Exploits This  
 - Imagine the website has an open redirect page like this: 
   https://example.com/redirect?url=
   This page automatically redirects users to another website.
   https://accounts.google.com/...&redirect_uri=https://example.com/redirect?url=https://evil.com

### The OAuth server sees:
Domain = example.com
looks valid
so it accetps it

### What Happens Next:
1. Victim clicks “Login with Google”.
2. Google successfully authenticates the victim.
3. Google sends the authorization code to:
https://example.com/redirect?url=https://evil.com&code=ABC123
4. The redirect page immediately forwards the victim to:
https://evil.com?code=ABC123
5. Now the attacker’s website receives the authorization code.
6. The attacker can exchange that code for an access token and potentially access the victim’s account.

### The problem is:
The application only checked the domain name (example.com) but did NOT verify the exact redirect path.

### Secure Solution:
Applications should:
- Validate the full exact redirect URI
- Avoid wildcard paths
- Prevent open redirect pages

### safe example:
https://example.com/oauth/callback


## 3. User Experience vs. Security: Adding a third-party login option like “Login with Google” is a significant user experience improvement. However, it also introduces complexity and new potential security risks. Based on the article and your own thoughts, describe one key trade-off a development team must consider when deciding to implement OAuth. (For example, think about the balance between convenience for the user and the responsibility of the application to protect user data).

### Without OAuth:
Users must create a new account.
Remember another password.
Fill registration forms.
Sometimes verify email.

### With OAuth (“Login with Google”):
User clicks one button.
Google already knows the user.
Login becomes very fast and easy.

From the user side: 
  - Better experiance
  - Faster Login
  - Less passwords to remember

But now the application becomes responsible for handling OAuth securely.

### Development must
 - Configure redirect URIs correctly
 - Protect access tokens
 - Validate state parameters
 - Prevent attacks like CSRF
 - Prevent stolen authorization codes

If developers make even a small mistake, attackers may:
 - Steal tokens
 - Access user accounts
 - Steal personal data

 ### The Trade-Off
 We are making login easier for users, but we are also increasing the security responsibility for developers.
 So the team must decide:
 Is the convenience worth the extra security complexity?
 Can we implement OAuth safely and correctly?

 OAuth improves user convenience, but developers must carefully handle security to protect user accounts and data.
