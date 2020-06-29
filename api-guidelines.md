# API Guidelines <!-- omit in toc -->

- [Use JSON](#use-json)
- [Use Nouns instead of Verbs](#use-nouns-instead-of-verbs)
- [Name the collections using Plural Nouns](#name-the-collections-using-plural-nouns)
- [Use resource nesting to show relations or hierarchy](#use-resource-nesting-to-show-relations-or-hierarchy)
- [Error Handling](#error-handling)
- [Use API Documentation](#use-api-documentation)
- [HTTP Status Codes](#http-status-codes)
- [Response format](#response-format)
  - [Success Reponses](#success-reponses)
  - [Error Responses](#error-responses)

## Use JSON
You should choose JSON as the data format used in the communication, both the payload and the response. What is more application/JSON is a generic MIME type which makes it a practical approach to use.
- Set the `Content-Type` to `application/json`


## Use Nouns instead of Verbs
In API development REST approach can be called a resource based. So in your application, you work with resources and their collections (eg. a single book, and a list of all books). The actions on the resources are strictly defined by the HTTP methods such as GET, PUT, POST, PATCH, DELETE (and a few others, that are less relevant for this example) and only they should be used to change the state of the resource. This leads us to the endpoint URI construction. Taking the above into account properly constructed endpoint should look similar to this:
```dotnetcli
    GET /books/123
    DELETE /books/123
    POST /books
    PUT /books/123
    PATCH /books/123
```
> There are certain cases where it is ok to use actions in a similar way to manipulating resources, eg.:
PUT /accounts/123/activation.


## Name the collections using Plural Nouns
For the collections in REST API development use plural nouns. It is not just an established good practice. It actually makes it easier for humans to get an idea that something is actually a collection.
```dotnetcli
    GET  /cars/123
    POST /cars
    GET /cars
```


## Use resource nesting to show relations or hierarchy
Resource objects often have some kind of functional hierarchy or are related to each other. For example in the online store, we have ‘users’ and ‘orders’. Orders always belong to some user, therefore we may have the following endpoints structure laid out:
```dotnetcli
    /users <- user’s list
    /users/123 <- specific user
    /users/123/orders <- orders list that belongs to a specific user
    /users/123/orders/0001 <- specific order of a specific user
```


## Error Handling
- Return error details in the response body
- Pay attention to status codes
- Use status codes consistently


## Use API Documentation
- we are using laravel-swagger for api documentaion
  

## HTTP Status Codes 
HTTP defines a bunch of meaningful status codes that can be returned from your API. These can be leveraged to help the API consumers route their responses accordingly. I've curated a short list of the ones that you definitely should be using:

- 200 OK - Response to a successful GET, PUT, PATCH or DELETE. Can also be used for a POST that doesn't result in a creation.
- 201 Created - Response to a POST that results in a creation. Should be combined with a Location header pointing to the location of the new resource
- 204 No Content - Response to a successful request that won't be returning a body (like a DELETE request)
- 304 Not Modified - Used when HTTP caching headers are in play
- 400 Bad Request - The request is malformed, such as if the body does not parse
- 401 Unauthorized - When no or invalid authentication details are provided. Also useful to trigger an auth popup if the API is used from a browser
- 403 Forbidden - When authentication succeeded but authenticated user doesn't have access to the resource
- 404 Not Found - When a non-existent resource is requested
- 405 Method Not Allowed - When an HTTP method is being requested that isn't allowed for the authenticated user
- 410 Gone - Indicates that the resource at this end point is no longer available. Useful as a blanket - response for old API versions
- 415 Unsupported Media Type - If incorrect content type was provided as part of the request
- 422 Unprocessable Entity - Used for validation errors
- 429 Too Many Requests - When a request is rejected due to rate limiting


## Response format

### Success Reponses
1. **GET** - Get single item ( **HTTP Response Code: 200** )
   ```dotnetcli
   HTTP/1.1 200
   Content-Type: application/json

   {
       "data":{
           "id": 1,
           "name": "Jhon Doe",
           "active": true
        },
        "token": "grRgZxJmxMHA5v7rHaJhjnbkebgTaFGQPtVW9JFw7KN6K4bzFKw9ScJa7jtQ4YjQaJWCrdve6AFJbjXqYnc42eQkn5Kt2MHg47QYan86WquyxsNgpeMQvPeAWaxcK5zH"
   }
   ```

2. GET - Get collection - ( **HTTP Response Code: 200** )
   ```dotnetcli
    HTTP/1.1 200
    Pagination-Count: 100
    Pagination-Page: 5
    Pagination-Offset: 1
    Pagination-Limit: 20
    Content-Type: application/json

    {
        "message": null,
        "data":{[
            {
              "id": 1,
              "name": "John Doe",
              "active": true
            },
            {
              "id": 2,
              "name": "Jane Doe",
              "active": true
            }
        ]},
        "token": "grRgZxJmxMHA5v7rHaJhjnbkebgTaFGQPtVW9JFw7KN6K4bzFKw9ScJa7jtQ4YjQaJWCrdve6AFJbjXqYnc42eQkn5Kt2MHg47QYan86WquyxsNgpeMQvPeAWaxcK5zH"
    }
   ```
3. POST - Create new record - ( **HTTP Response Code: 201** )
   ```dotnetcli
   HTTP/1.1 201
   Content-Type: application/json

   {
       "message": "successfully created user"
       "data":{},
       "token": "grRgZxJmxMHA5v7rHaJhjnbkebgTaFGQPtVW9JFw7KN6K4bzFKw9ScJa7jtQ4YjQaJWCrdve6AFJbjXqYnc42eQkn5Kt2MHg47QYan86WquyxsNgpeMQvPeAWaxcK5zH"
   }
   ```

4. PATCH - Update a record - ( **HTTP Response Code: 200** )
   > If updated entity is sent back.
   ```dotnetcli
   HTTP/1.1 200
   Content-Type: application/json

   {
       "message": "Successfully updated"
       "data":{
           "id": 3,
           "name": "Johnny Craig",
           "active": true
       }
   }
   ```

   > If entiry is not sent back after update
   ```dotnetcli
   HTTP/1.1 204
   ```

5. DELETE - Delete a record ( **HTTP Response Code: 204** )
   ```dotnetcli
   HTTP/1.1 204
   ```
### Error Responses
1. GET - **HTTP Response Code: 404**
   ```dotnetcli
   HTTP/1.1 404
   Content-Type: application/json

   {
       "error": "Data not found"
   }
   ```

2. DELETE - **HTTP Response Code: 404**
   ```dotnetcli
   HTTP/1.1 404
   Content-Type: application/json

   {
       "error": "Data not found"
   }
   ```

3. POST - **HTTP Response Code: 400**
   ```dotnetcli
   HTTP/1.1 400
   Content-Type: application/json

   {
       "error": [
           {
               "message": "Invaid value",
               "field": "email"
           },
           {
               "message": "Invalid format",
               "field": "TelNo"
           }
       ]
   }
   ```

4. PATCH - **HTTP Response Code: 400/404**
   ```dotnetcli
   HTTP/1.1 400
   Content-Type: application/json

   {
       "error":[
           {
               "message": "Invalid format",
               "field": "TelNo"
           }
       ]
   }



   HTTP/1.1 404
   Content-Type: application/json

   {
       "error":[
           {
               "message": "Field does not exists"
           }
       ]
   }
   ```

5. VERB Unauthorized - **HTTP Response Code: 401**
   ```dotnetcli
    HTTP/1.1  401
    Content-Type: application/json
 
    {
      "error": "Authentication credentials were missing or incorrect"
    }
   ```

6. VERB Forbidden - **HTTP Response Code: 403**
   ```dotnetcli
    HTTP/1.1  403
    Content-Type: application/json
 
    {
      "error": "The request is understood, but it has been refused or access is not allowed"
    }
   ```

7. VERB Conflict - **HTTP Response Code: 409**
   ```dotnetcli
    HTTP/1.1  409
    Content-Type: application/json
 
    {
      "error": "Any message which should help the user to resolve the conflict"
    }
   ```

8. VERB Too Many Requests - **HTTP Response Code: 429**
   ```dotnetcli
    HTTP/1.1  429
    Content-Type: application/json
 
    {
      "error": "The request cannot be served due to the rate limit having been exhausted for the resource"
    }
   ```
9. VERB Internal Server Error - **HTTP Response Code: 500**
    ```dotnetcli
     HTTP/1.1  500
     Content-Type: application/json
 
     {
       "error": "Something is broken"
     }
    ```

10. VERB Service Unavailable - **HTTP Response Code: 503**
    ```dotnetcli
     HTTP/1.1  503
     Content-Type: application/json
 
     {
       "message": "The server is up, but overloaded with requests. Try again later!"
     }
    ```
