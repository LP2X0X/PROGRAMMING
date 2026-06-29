---
tags: [csharp, asp-net-core, http, status-codes, web-api, fundamentals]
---


Every HTTP response -- ==every single one== -- includes a three-digit **status code** in its first line. This code tells the client (browser, mobile app, API consumer) what the server did with the request:

```
HTTP/1.1 200 OK
Content-Type: application/json

{"id": 1, "name": "Widget"}
```

The `200` is the status code. The `OK` is the **reason phrase** -- a human-readable label that accompanies the code. The code is what matters to software; the reason phrase is informational.

Status codes are defined in [RFC 9110](https://www.rfc-editor.org/rfc/rfc9110) (formerly RFC 7231) and are split into five classes based on their first digit:

| First Digit | Class | Meaning |
|---|---|---|
| **1xx** | Informational | Request received, processing continues |
| **2xx** | Success | Request was successfully received, understood, and accepted |
| **3xx** | Redirection | Further action is needed to complete the request |
| **4xx** | Client Error | The request contains bad syntax or cannot be fulfilled |
| **5xx** | Server Error | The server failed to fulfill a valid request |

> [!ad-tip] The First Digit Is the Category
> You do not need to memorize every code. If you see a status code you have never encountered, the first digit tells you the general outcome. A `418` you have never seen? It is a 4xx, so something is wrong with the request (in this case, it is the famous joke code "I'm a teapot" -- not used in production, but it illustrates the principle).

> [!summary] Section Summary
> - Every HTTP response carries a three-digit status code
> - The first digit determines the class: 1xx informational, 2xx success, 3xx redirection, 4xx client error, 5xx server error
> - The reason phrase (e.g., "OK", "Not Found") is human-readable and informational only
> - Status codes are standardized in RFC 9110
