---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


A JWT consists of three parts separated by dots (`.`):

```
Header.Payload.Signature
```

Each part is **Base64Url** encoded. Here is a visual breakdown:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwicm9sZSI6IkFkbWluIiwiZXhwIjoxNzE4NzQwMDAwLCJpYXQiOjE3MTg3MzY0MDB9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
|___________________________|  |______________________________________________|  |___________________________________|
         Header                               Payload                                      Signature
```

## Header

The header describes the token type and the signing algorithm:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- `alg` -- the signing algorithm. `HS256` (HMAC-SHA256) uses a shared secret. `RS256` (RSA-SHA256) uses a public/private key pair.
- `typ` -- always `"JWT"` for JSON Web Tokens.

## Payload

The payload contains **claims** -- statements about the user and additional metadata:

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "role": "Admin",
  "email": "john@example.com",
  "exp": 1718740000,
  "iat": 1718736400
}
```

- `sub` -- subject, typically the user ID
- `name` -- a custom claim with the user's name
- `role` -- a custom claim for authorization
- `exp` -- expiration time as a Unix timestamp
- `iat` -- issued-at time as a Unix timestamp

> [!warning] Common Misconception
> The payload is **encoded**, not **encrypted**. Anyone can decode the Base64Url string and read the claims. Never store sensitive information like passwords, credit card numbers, or secrets in the payload.

## Signature

The signature ensures the token has not been tampered with:

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

The server uses the secret key to generate the signature when creating the token. On every incoming request, the server recalculates the signature using the same secret and compares it to the signature in the token. If they match, the token is valid.

> [!tip]
> Think of the signature like a wax seal on a letter. Anyone can read the letter (the payload is not encrypted), but only the person with the seal (the secret key) can produce a valid seal. If the seal is broken or different, you know the letter was tampered with.

> [!summary] Section Summary
> A JWT has three Base64Url-encoded parts: the Header (algorithm and type), the Payload (claims about the user), and the Signature (cryptographic proof of integrity). The payload is readable by anyone -- it is encoded, not encrypted.
