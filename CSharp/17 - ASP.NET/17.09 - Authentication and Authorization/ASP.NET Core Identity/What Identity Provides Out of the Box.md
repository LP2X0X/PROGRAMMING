---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


Identity ships with a large surface area of functionality. Here is what you get without writing custom code:

## User Registration and Login

Identity provides `UserManager<T>` for creating user accounts with validated passwords and `SignInManager<T>` for authenticating users against stored credentials. It handles the full flow: validate input, hash the password, store the user, issue an authentication cookie.

## Password Hashing

> [!info] Definition
> **PBKDF2** (Password-Based Key Derivation Function 2) is the default password hashing algorithm used by Identity. In .NET 8+, it uses **600,000 iterations** with HMAC-SHA512, which is a significant increase from earlier versions.

Passwords are never stored in plain text. Identity hashes them using a one-way function and stores only the hash. When a user logs in, Identity hashes the provided password and compares it to the stored hash.

## Email Confirmation

Identity generates cryptographically secure tokens for email confirmation. After registration, you send the user an email with a confirmation link containing this token. When they click the link, Identity validates the token and marks their email as confirmed.

## Two-Factor Authentication (2FA)

Identity supports **TOTP-based** (Time-based One-Time Password) two-factor authentication, compatible with authenticator apps like Google Authenticator, Microsoft Authenticator, and Authy. Users scan a QR code, and the app generates rotating 6-digit codes.

## Account Lockout

After a configurable number of failed login attempts (default: 5), Identity locks the account for a configurable duration. This protects against brute-force attacks.

## Role Management

Roles are named groups that users can belong to. Identity provides `RoleManager<T>` for creating and managing roles, and supports checking role membership in authorization policies, `[Authorize(Roles = "Admin")]` attributes, and code-level checks.

## External Login Providers

Identity integrates with OAuth 2.0 and OpenID Connect providers out of the box. You can add Google, Microsoft, Facebook, GitHub, Twitter, and any generic OAuth provider. Identity handles the redirect flow, callback processing, and linking external accounts to local user records.

## Password Reset Tokens

When a user forgets their password, Identity generates a time-limited token. You send the token in a reset link, and Identity validates it before allowing the password change.

## Phone Number Confirmation

Similar to email confirmation, Identity can generate tokens to verify phone numbers via SMS. This is used for two-factor authentication via SMS (though TOTP is recommended over SMS for security).

> [!summary] Section Summary
> Identity provides user registration, login, PBKDF2 password hashing (600,000 iterations in .NET 8+), email confirmation, TOTP-based 2FA, account lockout, role management, external login providers (Google, Microsoft, etc.), password reset tokens, and phone number confirmation -- all out of the box.
