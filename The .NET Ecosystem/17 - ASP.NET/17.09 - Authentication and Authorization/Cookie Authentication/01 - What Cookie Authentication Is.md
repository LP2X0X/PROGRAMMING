---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


> [!info] Definition
> **Cookie authentication** is an authentication scheme where the server issues an encrypted cookie after a successful login. The browser automatically stores this cookie and sends it back with every subsequent HTTP request, allowing the server to identify the user without requiring them to re-enter credentials.

Cookie authentication is the **default and recommended** authentication scheme for server-rendered web applications built with ASP.NET Core MVC or Razor Pages. Unlike token-based schemes (such as [[JWT Authentication]]), cookie authentication relies on the browser's built-in cookie storage and automatic cookie transmission behavior.

The core idea is straightforward:

1. The user proves their identity once (typically via a login form).
2. The server packages the user's identity information (claims) into an encrypted, tamper-proof cookie.
3. The browser stores this cookie and attaches it to every request to the same domain.
4. The server decrypts the cookie on each request to reconstruct the user's identity.

This approach works naturally with the stateless nature of HTTP -- each request carries everything the server needs to know about the user, encoded inside the cookie.

> [!warning] Common Misconception
> Cookie authentication does **not** store the user's password in the cookie. The cookie contains an encrypted version of the user's **claims** (name, roles, email, etc.) -- never raw credentials. Even if someone intercepted the cookie, they would not be able to extract the password from it.

> [!summary] Section Summary
> Cookie authentication is the standard scheme for server-rendered ASP.NET Core apps. The server encrypts user identity data into a cookie after login, and the browser sends it back automatically on every request.
