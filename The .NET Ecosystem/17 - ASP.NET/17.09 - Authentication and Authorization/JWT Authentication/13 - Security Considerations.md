---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


## 1. Always Use HTTPS

```
Authorization: Bearer eyJhbGci...
```

Tokens travel in headers. Without HTTPS, anyone on the network can intercept the token (man-in-the-middle attack).

> [!danger]
> **Never** transmit JWTs over plain HTTP. In production, enforce HTTPS at the infrastructure level and set `Secure` on any cookies carrying tokens.

## 2. Keep Tokens Short-Lived

```csharp
expires: DateTime.UtcNow.AddMinutes(15)  // 15 minutes is a good default
```

Short-lived tokens limit the window of exploitation if a token is compromised.

## 3. The Revocation Problem

JWTs cannot be revoked before their expiry. Once issued, they remain valid until `exp`. Mitigation strategies:

- **Short expiration times** -- reduce the damage window
- **Token blacklist** -- store revoked `jti` values in Redis/database (but this sacrifices statelessness)
- **Refresh token revocation** -- revoke the refresh token so no new access tokens can be obtained

## 4. Use Strong Signing Keys

```csharp
// Bad -- too short
"Jwt:Key": "mysecret"

// Good -- at least 256 bits (32+ characters)
"Jwt:Key": "this-is-a-sufficiently-long-secret-key-for-hs256-signing!"
```

## 5. Never Store Sensitive Data in the Payload

The payload is Base64Url encoded, not encrypted. Anyone can decode it:

```bash
echo "eyJzdWIiOiIxMjM0NTY3ODkwIn0" | base64 -d
# {"sub":"1234567890"}
```

> [!warning] Common Misconception
> Developers sometimes assume JWT payloads are encrypted because they "look encrypted." They are not. Base64 is an encoding, not encryption. Anyone with the token string can read every claim.

## 6. Key Rotation

Periodically rotate your signing keys. Use `IssuerSigningKeys` (plural) in `TokenValidationParameters` to accept both old and new keys during the transition period:

```csharp
IssuerSigningKeys = new[]
{
    new SymmetricSecurityKey(Encoding.UTF8.GetBytes(currentKey)),
    new SymmetricSecurityKey(Encoding.UTF8.GetBytes(previousKey))
}
```

## 7. Token Size

Every claim you add increases the token size, and the token is sent with **every request**. Keep the payload lean -- include only what is necessary for authentication and basic authorization. Fetch detailed user data from the API when needed.

> [!tip]
> A rule of thumb: if a claim is needed on nearly every request (like `sub`, `role`, `email`), include it. If it is needed rarely (like a full address or profile picture URL), leave it out and fetch it on demand.

> [!summary] Section Summary
> Key security practices: always use HTTPS, keep tokens short-lived, use strong signing keys, never store secrets in the payload, implement key rotation, and accept that JWTs cannot be easily revoked. Combine short-lived access tokens with revocable refresh tokens for the best balance.
