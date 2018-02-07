# OAuth Primer

## About this document

This document is public domain as specified in the UNLICENSE public domain dedication. You don't have to give credit, but please refer to the GitHub repository <https://github.com/awwright/oauth-primer> if you can.



## Intro

OAuth is a framework for granting third party Web services access to your resources. For example, allowing Twitter to make Facebook posts on your behalf.



## Authenticating to Web Services

The most basic web service takes a request with some sort of credential and produces a response:

0. The user makes a request, and includes a credential to identify them, such as a username and password
0. The server authenticates the credentials and makes a response.

HTTP defines the `Authorization` request-header to pass a credential that is supposed to authorize the request, and the `Authorization: Basic` type for passing a username and password. For uses aimed at Web browsers, servers may also store credentials in cookies.



## Feature: Access Tokens

Credentials are frequently very expensive to authenticate. Many authentication methods involve entering a one-time-password, responding to an SMS message, or hashing a password through a computationally-expensive PBKDF.

To avoid having to re-authenticate the credential every time a request needs to be made, Web services have a login process where the credential is exchanged for a highly-secure, limited-duration _access token_:

0. The resource owner logs into a server in with a username and password.
0. The password is authenticated, using a PBKDF for example. The server authenticates the password, then provides a temporary-duration, strong-security access token to be used from then on.
0. The resource owner uses the access token in requests to the Web server.

There are different kinds of access tokens, the most common being a "bearer token", a static, opaque string that gives the bearer access to the protected resources. Other schemes are available that rotate the credential with a nonce (number used once), or public key cryptography.

Websites typically store this token in a cookie.

In OAuth, a username and password can be exchanged for a bearer token using `grant_type=password`, and the access token is provided in the Authorization request-header as `Authorization: Bearer [Token]`, where [Token] is some arbitrary string that only the bearer would know.



## Feature: Split resource/authorization servers

Sometimes it’s useful for services to have different servers for authenticating users and hosting resources.

To accommodate this, OAuth makes a distinction between a _resource server_ and an _authorization server_. When a user needs to make a request, they acquire an authorization token from the authorization server, then pass it to a resource server in the Authorization header.



## Feature: Redirect workflow

Not all login workflows use a username and password only. Modern Web applications, and especially applications protecting valuable assets like bank accounts, will use Two-Factor Authentication, which may involve copying a number off their phone, or other device with the effect of thwarting remote access.

These Two-Factor Authentication systems must be asynchronous; you may need to give the user instructions on how to further identify themselves, and you don’t want to give out an access token until they have proven their identity.

For desktop applications, mobile applications, and other applications where the user is running trusted code, but needs to log in to a website, OAuth provides the “Implicit grant” workflow:

0. User indicates they want to sign in
0. Application opens a Web browser pointed to the authorization server’s login page
0. The user follows the prompts to login
0. The user approves the permissions to grant to the new access token, if necessary
0. The service returns an access token to the application
0. The application makes requests to the resource server using the access token



## Feature: Authorization code (Split client/user-agent)

Sometimes the user, working from their _user-agent_, wants to let a third party access resources on their behalf, through the third party's _client_. For example, perhaps the user wants to give Twitter some credentials to post to the user's Facebook account, or a user wants to allow a finance service to access their banking transactions.

However it is difficult for the user to manually generate an authorization token and enter it into the website. Further, this passes the access token between more places than necessary (since the user doesn't ever need to see the access token firsthand), and such excessive handling increases the chance it will be leaked. Ideally, we would like to send the access token only to the client that will make use of it.

To do this, OAuth can send the user-agent an _authorization code_ instead of an access token. This authorization code is a token that can be exchanged by the client for an access token.

0. Client service redirects the resource owner to the authorization server along with its identity and desired permissions
0. The user follows the authorization server prompts to authenticate, if necessary
0. The user gives their permission to create a access token with the desired permissions
0. The authorization server redirects the user to the client service, along with an authorization code (appended in the redirect URL)
0. The client service connects to the authorization server, authenticates itself using a shared secret, and exchanges the authorization code for an access token.
0. The client service makes requests to the resource server using the access token.

The access token is only usable by clients that have previously registered a secret key with the authorization server.


## Feature: Refresh Tokens

Since access tokens are used on multiple servers, it’s possible that they might get leaked, so it’s prudent to limit the amount of time that they are valid for. However, this poses a problem that users have to re-follow the authorization process every time an access token expires - often as short as a few hours.

In addition to the access token, OAuth can also pass a _refresh token_ to a client: a credential shared only between the client and authorization server, that allows the client to rotate the access token and refresh token, and obtain new ones.

Generating a new refresh token instead of re-using the authorization code helps ensure that credentials do not have to be long lived, and are rotated automatically.
