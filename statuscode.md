# HTTP Response Status Codes

HTTP response status codes indicate whether a specific HTTP request has been successfully completed. The responses are grouped into five classes, each identified by the first digit of the status code:

## 1xx: Informational Responses (100–199)
These codes indicate that the request was received and understood, and that the process is continuing.

- **100 Continue**: The server has received the request headers, and the client should proceed to send the request body.
- **101 Switching Protocols**: The requester has asked the server to switch protocols, and the server has agreed to do so.
- **102 Processing**: The server has received and is processing the request, but no response is available yet.

## 2xx: Successful Responses (200–299)
These codes indicate that the request was successfully received, understood, and accepted.

- **200 OK**: The request has succeeded. The meaning depends on the HTTP method (GET, POST, etc.).
- **201 Created**: The request has been fulfilled and resulted in a new resource being created.
- **202 Accepted**: The request has been accepted for processing, but the processing has not been completed.
- **204 No Content**: The server successfully processed the request, but is not returning any content.

## 3xx: Redirection Messages (300–399)
These codes indicate that further action needs to be taken by the user agent in order to fulfill the request.

- **301 Moved Permanently**: The resource has been moved to a new URL permanently.
- **302 Found**: The resource resides temporarily under a different URL.
- **304 Not Modified**: Indicates that the resource has not been modified since the last request.

## 4xx: Client Error Responses (400–499)
These codes indicate that the client seems to have made an error.

- **400 Bad Request**: The server could not understand the request due to invalid syntax.
- **401 Unauthorized**: The client must authenticate itself to get the requested response.
- **403 Forbidden**: The client does not have access rights to the content.
- **404 Not Found**: The server can not find the requested resource.
- **405 Method Not Allowed**: The request method is known by the server but is not supported by the target resource.

## 5xx: Server Error Responses (500–599)
These codes indicate that the server failed to fulfill a valid request.

- **500 Internal Server Error**: The server has encountered a situation it doesn't know how to handle.
- **501 Not Implemented**: The request method is not supported by the server and cannot be handled.
- **502 Bad Gateway**: The server, while acting as a gateway or proxy, received an invalid response from the upstream server.
- **503 Service Unavailable**: The server is not ready to handle the request, often due to maintenance or overload.
- **504 Gateway Timeout**: The server is acting as a gateway and cannot get a response in time.

---

## Example Table of Common Status Codes

| Code | Name                     | Description                                      |
|------|--------------------------|--------------------------------------------------|
| 100  | Continue                 | Initial part of request received, continue.      |
| 200  | OK                       | Request succeeded.                               |
| 201  | Created                  | Resource created.                                |
| 204  | No Content               | No content to send.                              |
| 301  | Moved Permanently        | Resource moved permanently.                      |
| 302  | Found                    | Resource found at another URI.                   |
| 304  | Not Modified             | Resource not modified since last request.        |
| 400  | Bad Request              | Malformed request.                               |
| 401  | Unauthorized             | Authentication required.                         |
| 403  | Forbidden                | Access denied.                                   |
| 404  | Not Found                | Resource not found.                              |
| 405  | Method Not Allowed       | Method not allowed for resource.                 |
| 500  | Internal Server Error    | Server error.                                    |
| 501  | Not Implemented          | Method not implemented by server.                |
| 502  | Bad Gateway              | Invalid response from upstream server.           |
| 503  | Service Unavailable      | Server unavailable.                              |
| 504  | Gateway Timeout          | Upstream server timeout.                         |

---

## References
- [MDN Web Docs: HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
- [RFC 7231: Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231)

## Detailed Examples of Common Testing Status Codes

Below are more detailed explanations and practical examples for the most commonly used HTTP status codes in API testing and development:

### 200 OK
- **Description:** The request has succeeded. The meaning depends on the HTTP method used.
- **Example:**
  - **Request:**
    ```http
    GET /api/users/123 HTTP/1.1
    Host: example.com
    ```
  - **Response:**
    ```http
    HTTP/1.1 200 OK
    Content-Type: application/json

    {
      "id": 123,
      "name": "John Doe"
    }
    ```

### 201 Created
- **Description:** The request has been fulfilled and resulted in a new resource being created.
- **Example:**
  - **Request:**
    ```http
    POST /api/users HTTP/1.1
    Host: example.com
    Content-Type: application/json

    {
      "name": "Jane Doe"
    }
    ```
  - **Response:**
    ```http
    HTTP/1.1 201 Created
    Location: /api/users/124
    Content-Type: application/json

    {
      "id": 124,
      "name": "Jane Doe"
    }
    ```

### 204 No Content
- **Description:** The server successfully processed the request, but is not returning any content.
- **Example:**
  - **Request:**
    ```http
    DELETE /api/users/123 HTTP/1.1
    Host: example.com
    ```
  - **Response:**
    ```http
    HTTP/1.1 204 No Content
    ```

### 400 Bad Request
- **Description:** The server could not understand the request due to invalid syntax.
- **Example:**
  - **Request:**
    ```http
    POST /api/users HTTP/1.1
    Host: example.com
    Content-Type: application/json

    {
      "name": 12345
    }
    ```
  - **Response:**
    ```http
    HTTP/1.1 400 Bad Request
    Content-Type: application/json

    {
      "error": "Name must be a string."
    }
    ```

### 401 Unauthorized
- **Description:** The client must authenticate itself to get the requested response.
- **Example:**
  - **Request:**
    ```http
    GET /api/profile HTTP/1.1
    Host: example.com
    ```
  - **Response:**
    ```http
    HTTP/1.1 401 Unauthorized
    WWW-Authenticate: Bearer
    Content-Type: application/json

    {
      "error": "Authentication required."
    }
    ```

### 403 Forbidden
- **Description:** The client does not have access rights to the content.
- **Example:**
  - **Request:**
    ```http
    DELETE /api/admin HTTP/1.1
    Host: example.com
    Authorization: Bearer user-token
    ```
  - **Response:**
    ```http
    HTTP/1.1 403 Forbidden
    Content-Type: application/json

    {
      "error": "You do not have permission to perform this action."
    }
    ```

### 404 Not Found
- **Description:** The server can not find the requested resource.
- **Example:**
  - **Request:**
    ```http
    GET /api/users/999 HTTP/1.1
    Host: example.com
    ```
  - **Response:**
    ```http
    HTTP/1.1 404 Not Found
    Content-Type: application/json

    {
      "error": "User not found."
    }
    ```

### 500 Internal Server Error
- **Description:** The server has encountered a situation it doesn't know how to handle.
- **Example:**
  - **Request:**
    ```http
    GET /api/users HTTP/1.1
    Host: example.com
    ```
  - **Response:**
    ```http
    HTTP/1.1 500 Internal Server Error
    Content-Type: application/json

    {
      "error": "An unexpected error occurred."
    }
    ```
