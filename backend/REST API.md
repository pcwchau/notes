# Index

- [Index](#index)
- [REST API](#rest-api)
- [Service Side](#service-side)
  - [API Response Format](#api-response-format)
    - [JSend Standard](#jsend-standard)
    - [Concluded Format](#concluded-format)
  - [API Document Example](#api-document-example)
    - [Send Message API](#send-message-api)
      - [HTTP Request](#http-request)
      - [Path Parameters](#path-parameters)
      - [Request Body](#request-body)
- [Client Side](#client-side)

# REST API

- [Architectural constraints - Wikipedia](https://en.wikipedia.org/wiki/REST#Architectural_constraints)

REST, Representational State Transfer.

REST is an architectural style (not a protocol like SOAP, not a technology itself or even not an implementation, it is basically a set of a rule), This architecture offers some constraints for using HTTP. If you stick by this architectural constraints while using HTTP, it is called RESTful, otherwise, it is not-RESTful. ([stackflow](https://stackoverflow.com/questions/32369856/non-restful-vs-restful))

REST is the way HTTP should be used. Today we only use a tiny bit of the HTTP protocol's methods – namely `GET` and `POST`. The REST way to do it is to use all of the protocol's methods. For example, REST dictates the usage of `DELETE` to erase a document (be it a file, state, etc.) behind a URI, whereas, with HTTP, you would misuse a `GET` or `POST` query like `...product/?delete_id=22`. ([stackflow](https://stackoverflow.com/questions/2190836/what-is-the-difference-between-http-and-rest))

Instead of having randomly named setter and getter URLs and using `GET` for all the getters and `POST` for all the setters, we try to have the URLs identify resources, and then use the HTTP actions `GET`, `POST`, `PUT` and `DELETE` to do stuff to them. So instead of

```
GET /get_article?id=1
POST /delete_article   id=1
```

You would do

```
GET /articles/1/
DELETE /articles/1/
```

([stackflow](https://stackoverflow.com/questions/2191049/what-is-the-advantage-of-using-rest-instead-of-non-rest-http))

# Service Side

## API Response Format

To develop an API server, you need to ensure that all response formats are standardized and consistent. There are many standards, and I have concluded a standard based on my experience.

### JSend Standard

- [JSend](https://github.com/omniti-labs/jsend)

Type | Description | Required Keys | Optional Keys
---- | ----------- | ------------- | -------------
success | All went well, and (usually) some data was returned. | status, data
fail    | There was a problem with the data submitted, or some pre-condition of the API call wasn't satisfied. | status, data
error   | An error occurred in processing the request, i.e. an exception was thrown. | status, message | code, data

```json
{
    "status" : "success",
    "data" : {
        "post" : { "id" : 1, "title" : "A blog post", "body" : "Some useful content" }
    }
}
```

```json
{
    "status" : "fail",
    "data" : {
        "title" : "A title is required"
    }
}
```

```json
{
    "status" : "error",
    "message" : "Unable to communicate with database."
}

{
    "status" : "error",
    "code": "NO_RECIPIENTS",
    "message" : "No devices matched the specified condition."
}
```

### Concluded Format

`GET` Success:

```json
{
    "code": 0,
    "message" : "Success",
    "data" : {
        "post" : { "id" : 1, "title" : "A blog post", "body" : "Some useful content" }
    }
}
```

`GET` Fail / Error:

```json
{
    "code": 1,  // Any other number
    "message" : "Missing parameter",  // Reason of failure / error
    "data" : null
}
```

## API Document Example

### Send Message API

#### HTTP Request

```
POST /notification/sendMessage?company_id={companyId}&service_id={serviceId}
```

#### Path Parameters

| Field       | Type     | Description            | Example |
|-------------|----------|------------------------|---------|
| `companyId` | `String` | The ID of the company. | `1`     |
| `serviceId` | `String` | The ID of the service. | `1`     |

#### Request Body

*Note: Please make sure to send the `Content-type: application/json` header with your request.*

```json
{
    "targets": ["a6df9b1f18fc22dd68e969", "4f7af3f8aa7c22845a43f6"],
    "targetType": "user",
    "message": "Hello"
}
```

Field | Type | Description | Example
----- | ---- | ----------- | -------
`targets` | `String[]` | The token of receivers, either individual users or channels. | `["a6df9b1f18fc22dd68e969"]` <br> `["a6df9b1f18fc22dd68e969", "4f7af3f8aa7c22845a43f6"]` <br> `["news"]` <br> `["news", "media"]`
`targetType` | `String` | Use `user` to send message to individual users. <br> Use `channel` to send message to subscription channels. | `"user"` <br> `"channel"`
`message` | `String` | The message content. | `"Hello"`

Sample Response Body:

```json
{
    "status": "Success",
    "data": "{\"success\":true,\"id\":\"6615081cefc4ef7c3a4f74a9\",\"info\":{\"devices\":1}}"
}
```

Sample Error Response:

```json
{
    "status": "Fail",
    "message": "Service not found"
}
```

- `Fail` means error in request data. `Error` means system error.

# Client Side