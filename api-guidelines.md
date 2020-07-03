# API Guidelines <!-- omit in toc -->

- [HTTP Header](#http-header)
- [Naming](#naming)
- [HTTP Status Codes](#http-status-codes)
- [Response format](#response-format)
  - [Success](#success)
  - [Error](#error)

# HTTP Header
Set `Accept` to `application/json` in order to return data as a JSON format.

# Naming
| Type | Bad | Good |
|------|-----|------|
| Noun instead of verb | /getUser/55 | /users/55 |
| Resource hierarchy | /orders?user=55 | /users/55/orders |
| Proper grammar (singular, plural) | /users/55/addresses | /users/55/address |

# HTTP Status Codes 
HTTP defines a bunch of meaningful status codes that can be returned from your API. These can be leveraged to help the API consumers route their responses accordingly. I've curated a short list of the ones that you definitely should be using:

| Status Code | Meaning | Message |
|-------------|---------|---------|
| **200 OK** | The request has succeeded.|
| **201 Created** | The request has been fulfilled and has resulted in one or more new resources being created. |
| **204 No Content** | The server has successfully fulfilled the request and that there is no additional content to send in the response payload body. |
| **304 Not Modified** | A conditional GET or HEAD request has been received and would have resulted in a 200 OK response if it were not for the fact that the condition evaluated to false. |
| **400 Bad Request** | The server cannot or will not process the request due to something that is perceived to be a client error (e.g., malformed request syntax, invalid request message framing, or deceptive request routing). |
| **401 Unauthorized** | The request has not been applied because it lacks valid authentication credentials for the target resource. | "Authentication credentials were missing or incorrect" |
| **403 Forbidden** | The server understood the request but refuses to authorize it. | "The request is understood, but it has been refused or access is not allowed" |
| **404 Not Found** | The origin server did not find a current representation for the target resource or is not willing to disclose that one exists. | "Data not found" |
| **405 Method Not Allowed** | The method received in the request-line is known by the origin server but not supported by the target resource. |
| **410 Gone** | The target resource is no longer available at the origin server and that this condition is likely to be permanent. |
| **415 Unsupported Media Type** | The origin server is refusing to service the request because the payload is in a format not supported by this method on the target resource. |
| **422 Unprocessable Entity** | The server understands the content type of the request entity (hence a 415 Unsupported Media Type status code is inappropriate), and the syntax of the request entity is correct (thus a 400 Bad Request status code is inappropriate) but was unable to process the contained instructions. |
| **429 Too Many Requests** | The user has sent too many requests in a given amount of time ("rate limiting"). | "The request cannot be served due to the rate limit having been exhausted for the resource" |
| **500 Internal Server Error** | The server encountered an unexpected condition that prevented it from fulfilling the request. | "Something is broken" |
| **503 Service Unavailable** | The server is currently unable to handle the request due to a temporary overload or scheduled maintenance, which will likely be alleviated after some delay. | "The server is up, but overloaded with requests. Try again later!" |

# Response Format

## Success 

1. **GET** - **Resource** - a single record
```
{
    "data": {
        "id": 1,
        "code": "TH",
        "name": "Thailand",
        "status": "enabled"
    }
}
```

2. **GET** - **Collection** - a group of records
```
{
    "data": [
        {
            "id": 1,
            "code": "TH",
            "name": "Thailand",
            "status": "enabled"
        },
        {
            "id": 2,
            "code": "PH",
            "name": "Philippines",
            "status": "enabled"
        }
    ]
}
```

3. **POST/UPDATE** - create/update a record
```
{
    "message": "Successfully created/updated"
    "data": {
        "id": 1,
        "code": "TH",
        "name": "Thailand",
        "status": "enabled"
    }
}
```

4. **DELETE** - deleted a record
```
{
    "message": "Successfully deleted",
    "data": {}
}
```

## Error

```
{
    "message": "The given data was invalid.",
    "errors": {
        "name": [
            "The name field is required."
        ],
        "email": [
            "The email field is required."
        ],
        "message": [
            "The message field is required."
        ]
    }
}
```

