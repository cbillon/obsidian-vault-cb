---
link: https://mockoon.com/articles/api-glossary/
byline: Authorization
site: Mockoon
excerpt: This article provides a glossary for many terms and acronyms you will come accross when working with APIs
slurped: 2025-12-22T17:27
title: Mockoon - Mockoon's APIs glossary
tags:
  - api
---

##  [](https://mockoon.com/articles/api-glossary/#table-of-content)Table of content

[API (Application Programming Interface)](https://mockoon.com/articles/api-glossary/#api-application-programming-interface)  
[Authentication](https://mockoon.com/articles/api-glossary/#authentication)  
[Authorization](https://mockoon.com/articles/api-glossary/#authorization)  
[Body](https://mockoon.com/articles/api-glossary/#body)  
[Cache](https://mockoon.com/articles/api-glossary/#cache)  
[Client](https://mockoon.com/articles/api-glossary/#client)  
[CORS (Cross-Origin Resouce Sharing)](https://mockoon.com/articles/api-glossary/#cors-cross-origin-resouce-sharing)  
[CRUD (Create Read Update Delete)](https://mockoon.com/articles/api-glossary/#crud-create-read-update-delete)  
[Endpoint](https://mockoon.com/articles/api-glossary/#endpoint)  
[External API](https://mockoon.com/articles/api-glossary/#external-api)  
[API Gateway](https://mockoon.com/articles/api-glossary/#api-gateway) [Header](https://mockoon.com/articles/api-glossary/#header)  
[HTTP/HTTPS](https://mockoon.com/articles/api-glossary/#http-https)  
[Internal API](https://mockoon.com/articles/api-glossary/#internal-api)  
[JSON](https://mockoon.com/articles/api-glossary/#json)  
[API Key](https://mockoon.com/articles/api-glossary/#api-key)  
[Latency](https://mockoon.com/articles/api-glossary/#latency)  
[Load balancing](https://mockoon.com/articles/api-glossary/#load-balancing) [Methods (HTTP methods)](https://mockoon.com/articles/api-glossary/#methods-http-methods)  
[Middleware](https://mockoon.com/articles/api-glossary/#middleware)  
[Mime type](https://mockoon.com/articles/api-glossary/#mime-type)  
[API Mocking](https://mockoon.com/articles/api-glossary/#api-mocking)  
[OpenAPI/Swagger](https://mockoon.com/articles/api-glossary/#openapi-swagger) [Pagination](https://mockoon.com/articles/api-glossary/#pagination)  
[Path Parameters](https://mockoon.com/articles/api-glossary/#path-parameters)  
[Payload](https://mockoon.com/articles/api-glossary/#payload)  
[Polling](https://mockoon.com/articles/api-glossary/#polling)  
[Query Parameters](https://mockoon.com/articles/api-glossary/#query-parameters)  
[Rate limiting](https://mockoon.com/articles/api-glossary/#rate-limiting)  
[Request](https://mockoon.com/articles/api-glossary/#request)  
[Response](https://mockoon.com/articles/api-glossary/#response)  
[Resource](https://mockoon.com/articles/api-glossary/#resource)  
[REST API](https://mockoon.com/articles/api-glossary/#rest-api)  
[Route](https://mockoon.com/articles/api-glossary/#route)  
[Server](https://mockoon.com/articles/api-glossary/#server)  
[Status code (HTTP)](https://mockoon.com/articles/api-glossary/#status-code-http)  
[URL (Uniform Resource Locator)](https://mockoon.com/articles/api-glossary/#url-uniform-resource-locator)  
[Versioning](https://mockoon.com/articles/api-glossary/#versioning)  
[Web API](https://mockoon.com/articles/api-glossary/#web-api)  
[Webhooks](https://mockoon.com/articles/api-glossary/#webhooks)  
[WebSocket](https://mockoon.com/articles/api-glossary/#websocket)

##  [](https://mockoon.com/articles/api-glossary/#a)A

###  [](https://mockoon.com/articles/api-glossary/#api-application-programming-interface)API (Application Programming Interface)

API is the acronym for Application Programming Interface. In contrast to a User Interface (UI) that connects a person to a computer, it's a software-to-software interface, or intermediary, enabling two applications to talk to each other.

See also: [Web API](https://mockoon.com/articles/api-glossary/#web-api), [REST API](https://mockoon.com/articles/api-glossary/#rest-api)

###  [](https://mockoon.com/articles/api-glossary/#authentication)Authentication

Authentication is the process of **verifying the identity of a user or system**. In the context of APIs, it often involves the use of [API keys](https://mockoon.com/articles/api-glossary/#api-key), tokens, or other credentials to ensure that only authorized clients can access the API. An API may require authentication to protect sensitive data or resources and to ensure that only legitimate users can perform actions on the API. It is usually done by sending credentials in the [request](https://mockoon.com/articles/api-glossary/#request) headers, like the `Authorization` header, or in the [body](https://mockoon.com/articles/api-glossary/#body) of the request. An invalid or missing authentication will usually result in a `401 Unauthorized` [status code](https://mockoon.com/articles/api-glossary/#status-code-http).

Authorization is the process of **determining whether a user or system has permission to perform a specific action or access certain resources**. In APIs, this often follows authentication and involves checking the user's permissions against the requested resource or action. For example, an API may allow a user to read data but not modify it, or it may restrict access to certain endpoints based on the user's role or permissions. Authorization is typically enforced by the API server and can be implemented using various methods, such as role-based access control (RBAC) or attribute-based access control (ABAC). Missing access rights on a [resource](https://mockoon.com/articles/api-glossary/#resource) will usually result in a `403 Forbidden` [status code](https://mockoon.com/articles/api-glossary/#status-code-http).

##  [](https://mockoon.com/articles/api-glossary/#b)B

###  [](https://mockoon.com/articles/api-glossary/#body)Body

The body refers to the data transmitted in an API transaction in the [request](https://mockoon.com/articles/api-glossary/#request) or the [response](https://mockoon.com/articles/api-glossary/#response). Requests and responses do not always contain a body. [JSON](https://mockoon.com/articles/api-glossary/#json) is one of the most popular data formats to transfer data in the body

See also: [Payload](https://mockoon.com/articles/api-glossary/#payload)

##  [](https://mockoon.com/articles/api-glossary/#c)C

###  [](https://mockoon.com/articles/api-glossary/#cache)Cache

In an API, a cache is a system for storing and retrieving [responses](https://mockoon.com/articles/api-glossary/#response) to avoid reprocessing [requests](https://mockoon.com/articles/api-glossary/#request) that are frequent and identical. Multiple cache systems may coexist at different levels: clients (browsers), API gateways or proxies, servers, etc. [Servers](https://mockoon.com/articles/api-glossary/#server) usually indicate to the [client](https://mockoon.com/articles/api-glossary/#client) the caching policy of a request using [headers](https://mockoon.com/articles/api-glossary/#header).

###  [](https://mockoon.com/articles/api-glossary/#client)Client

A client is a piece of hardware or software that access services or [resources](https://mockoon.com/articles/api-glossary/#resource) made available by [servers](https://mockoon.com/articles/api-glossary/#server) in a client-server model. It usually sends a [request](https://mockoon.com/articles/api-glossary/#request) to the server, which processes it and returns a [response](https://mockoon.com/articles/api-glossary/#response). The client may access the server using a network, especially when the server is not on the same computer system.  
For example, a web browser is a client that connects to web servers to display web pages.

See also: [Server](https://mockoon.com/articles/api-glossary/#server)

###  [](https://mockoon.com/articles/api-glossary/#cors-cross-origin-resouce-sharing)CORS (Cross-Origin Resouce Sharing)

Cross-Origin Resource Sharing is an HTTP mechanism that allows a [server](https://mockoon.com/articles/api-glossary/#server) to indicate the origins from which a browser is allowed to load resources.  
By default, cross-origin requests (originating from a different host than the one serving the API) are restricted, and only same-origin requests are allowed. Practically, for all non-simple requests (based on multiple criteria, like the HTTP method used, the presence of a JSON body, etc.), browsers send a pre-flight request using the `OPTIONS` HTTP method and read the [response's](https://mockoon.com/articles/api-glossary/#response) headers (`Access-Control-Allow-Origin`, `Access-Control-Allow-Methods`, `Access-Control-Allow-Headers`, etc.) to check if the server allows requests emanating from this specific host.

###  [](https://mockoon.com/articles/api-glossary/#crud-create-read-update-delete)CRUD (Create Read Update Delete)

CRUD is an acronym for **C**reate, **R**ead, **U**pdate, and **D**elete, four basic operations of persistent storages. It is usually used in the [REST API](https://mockoon.com/articles/api-glossary/#rest-api) world to describe a group of [resource](https://mockoon.com/articles/api-glossary/#resource) [endpoints](https://mockoon.com/articles/api-glossary/#endpoint) and [HTTP methods](https://mockoon.com/articles/api-glossary/#methods-http-methods) matching each of the operations:

- **POST /resource** for an operation **C**reating a resource.
- **GET /resource** for an operation **R**eading a resource.
- **PUT /resource** for an operation **U**pdating a resource.
- **DELETE /resource** for an operation **D**eleting a resource.

##  [](https://mockoon.com/articles/api-glossary/#e)E

###  [](https://mockoon.com/articles/api-glossary/#endpoint)Endpoint

An endpoint is a communication channel or a location where an API will receive [requests](https://mockoon.com/articles/api-glossary/#request) for a specific [resource](https://mockoon.com/articles/api-glossary/#resource). For example, in a [REST API](https://mockoon.com/articles/api-glossary/#rest-api), accessing or modifying information related to users or invoices would be available on multiple `/users` or `/invoices` [routes](https://mockoon.com/articles/api-glossary/#route).

###  [](https://mockoon.com/articles/api-glossary/#external-api)External API

An external API usually exposes a company's internal resources outside of the organization letting third-party companies and developers use the data, for example, to create new applications. They are usually subject to restrictions and may require a paid subscription.

See also: [Internal API](https://mockoon.com/articles/api-glossary/#internal-api)

###  [](https://mockoon.com/articles/api-glossary/#api-gateway)API Gateway

An API Gateway is a server that acts as an **intermediary** between clients and backend services. It is responsible for routing requests from clients to the appropriate services, handling tasks such as [authentication](https://mockoon.com/articles/api-glossary/#authentication), [rate limiting](https://mockoon.com/articles/api-glossary/#rate-limiting), and [caching](https://mockoon.com/articles/api-glossary/#cache). API Gateways can simplify the client-side experience by providing a single entry point for multiple services and can also enhance security by hiding the internal architecture of the API.

##  [](https://mockoon.com/articles/api-glossary/#h)H

HTTP headers are used to pass additional information with HTTP [requests](https://mockoon.com/articles/api-glossary/#request) and [responses](https://mockoon.com/articles/api-glossary/#response). They take the form of a list of key-value pairs.  
Among the most used request headers:

- `Authorization: Bearer xxxxxxx`: contains the [API key or token](https://mockoon.com/articles/api-glossary/#api-key) used to authenticate and identify the client.
- `Content-Type: application/json`: indicates the mime type of the data sent in the request's body (`application/json`, `text/html`, etc.).
- `Accept-Encoding: gzip, deflate, br`: indicates the types of data encoding supported by the client.

Some widely used response headers:

- `Content-Type: application/json`: indicates the mime type of the data sent in the response's body (`application/json`, `text/html`, etc.).
- `Cache-Control: max-age=604800`: to indicate the duration after which the response should be refreshed.
- `Last-Modified: Fri, 24 June 2022 08:00:00 GMT`: indicate the data when the resource was last modified.

###  [](https://mockoon.com/articles/api-glossary/#httphttps)HTTP/HTTPS

HTTP (Hypertext Transfer Protocol) is the foundation of data communication on the web. It is an **application layer protocol** used for transmitting hypermedia documents, such as HTML. HTTPS (HTTP Secure) is the secure version of HTTP, which uses encryption (SSL/TLS) to protect the data exchanged between clients and servers.

##  [](https://mockoon.com/articles/api-glossary/#i)I

###  [](https://mockoon.com/articles/api-glossary/#internal-api)Internal API

An internal API provides resources within an organization's software system. They are usually consumed by internal applications and back-ends and are often used in micro-services architectures. Internal APIs target in-house services and developers and are an efficient way to share departments' data within the organization.

See also: [External API](https://mockoon.com/articles/api-glossary/#external-api)

##  [](https://mockoon.com/articles/api-glossary/#j)J

###  [](https://mockoon.com/articles/api-glossary/#json)JSON

JSON is a data format using human-readable text to transmit data objects consisting of key-value pairs. It is a popular data format for [web APIs](https://mockoon.com/articles/api-glossary/#web-api) used in the bodies of [requests](https://mockoon.com/articles/api-glossary/#request) and [responses](https://mockoon.com/articles/api-glossary/#response) of API transactions.  
A JSON example:

Copy

`{   "response": "success",  "status": 200 }`

##  [](https://mockoon.com/articles/api-glossary/#k)K

###  [](https://mockoon.com/articles/api-glossary/#api-key)API Key

An API key is a unique identifier used to authenticate and identify a user or an application accessing an [API](https://mockoon.com/articles/api-glossary/#api-application-programming-interface). Most APIs require their consumers (companies, developers, etc.) to register and request an API key as they are often paid products subjected to restrictions: consumer identification, volume billing, etc. API keys are frequently sent by the client along with the [request](https://mockoon.com/articles/api-glossary/#request) in an `Authorization` [header](https://mockoon.com/articles/api-glossary/#header).

##  [](https://mockoon.com/articles/api-glossary/#l)L

###  [](https://mockoon.com/articles/api-glossary/#latency)Latency

Latency is the **time** it takes for a [request](https://mockoon.com/articles/api-glossary/#request) to **travel from the client to the server and back**, including the time taken by the server to process the request and generate a [response](https://mockoon.com/articles/api-glossary/#response). It is usually measured in milliseconds (ms) and can be affected by various factors, such as network speed, server load, and the complexity of the request.

###  [](https://mockoon.com/articles/api-glossary/#load-balancing)Load balancing

Load balancing is the process of **distributing incoming API requests across multiple servers or instances** to ensure that no single server becomes overwhelmed with traffic. It helps improve the performance, reliability, and scalability of an API by preventing bottlenecks and ensuring that resources are used efficiently. Load balancers can be implemented at various levels, including hardware, software, or cloud-based solutions.

##  [](https://mockoon.com/articles/api-glossary/#m)M

###  [](https://mockoon.com/articles/api-glossary/#methods-http-methods)Methods (HTTP methods)

A [request](https://mockoon.com/articles/api-glossary/#request) is always targeting an API [route](https://mockoon.com/articles/api-glossary/#route) which comprises an HTTP verb or method, and a path. It indicates to the server what action the client intends to perform on a specific resource. There are multiple methods available: `GET`, `HEAD`, `POST`, `PUT`, `DELETE`, `CONNECT`, `OPTIONS`, `TRACE`, `PATCH`.

The most used ones are the following and embody specific meanings in [REST APIs](https://mockoon.com/articles/api-glossary/#rest-api):

- `POST`: create a new resource
- `GET`: retrieve a resource
- `PUT`: update an existing resource
- `DELETE`: remove an existing resource

See also: [CRUD](https://mockoon.com/articles/api-glossary/#crud-create-read-update-delete)

###  [](https://mockoon.com/articles/api-glossary/#middleware)Middleware

Middleware refers to software components that sit between the client and server in an API architecture. They can **intercept and process requests and responses**, allowing for tasks such as logging, authentication, and data transformation. Middleware can be used to add additional functionality to an API without modifying the core logic of the application.

###  [](https://mockoon.com/articles/api-glossary/#mime-type)Mime type

A MIME type (Multipurpose Internet Mail Extensions type) is a standard way to **indicate the nature and format of a document**, file, or byte stream. In the context of APIs, MIME types are used to specify the format of the data being sent in requests and responses, and is usually included in the `Content-Type` [header](https://mockoon.com/articles/api-glossary/#header) of the [request](https://mockoon.com/articles/api-glossary/#request) or [response](https://mockoon.com/articles/api-glossary/#response). It helps the client and server understand how to interpret the data being exchanged. Common MIME types include:

- `application/json`: JSON format
- `application/xml`: XML format
- `text/html`: HTML format
- `text/plain`: Plain text format

###  [](https://mockoon.com/articles/api-glossary/#api-mocking)API Mocking

API mocking is the action of simulating or imitating actual [APIs](https://mockoon.com/articles/api-glossary/#api-application-programming-interface) by answering fake realistic [responses](https://mockoon.com/articles/api-glossary/#response) to [requests](https://mockoon.com/articles/api-glossary/#request). It replaces APIs you cannot currently use because they are unavailable, down, or still under development. APIs could also be unavailable due to the context: like a restricted testing environment. It is a fast and easy way to test your applications with the APIs you are integrating, without the hassles.

##  [](https://mockoon.com/articles/api-glossary/#o)O

###  [](https://mockoon.com/articles/api-glossary/#openapiswagger)OpenAPI/Swagger

OpenAPI (formerly known as Swagger) is a **specification for building APIs**. It provides a standard way to describe the structure and behavior of an API using a JSON or YAML document. This document serves as a contract between the API provider and the consumers, allowing for better collaboration, documentation, and automation.

##  [](https://mockoon.com/articles/api-glossary/#p)P

Pagination is a technique used in APIs to **divide large sets of data into smaller, manageable chunks** or pages. It allows clients to retrieve data in smaller portions rather than fetching all the data at once, which can improve performance and reduce the load on the server. Pagination is often implemented using [query parameters](https://mockoon.com/articles/api-glossary/#query-parameters), such as `page` and `limit`, to specify which page of data to retrieve and how many items per page.

###  [](https://mockoon.com/articles/api-glossary/#path-parameters)Path Parameters

A path parameter is a non-optional section of the [route's](https://mockoon.com/articles/api-glossary/#route) path used as a placeholder populated with a value during a [request](https://mockoon.com/articles/api-glossary/#request). It allows the [client](https://mockoon.com/articles/api-glossary/#client) to indicate the target of the request to the [server](https://mockoon.com/articles/api-glossary/#server). They are usually represented in API documentations between curly braces or preceded by a colon.  
For example, in `/users/{id}` or `/users/:id`, `id` is a path parameter indicating that the action targets a user with a specific id: `/users/123`. It is up to the API [server](https://mockoon.com/articles/api-glossary/#server) to define which query parameters are available and needed.

See also: [Query parameters](https://mockoon.com/articles/api-glossary/#query-parameters)

###  [](https://mockoon.com/articles/api-glossary/#payload)Payload

The payload is the **data sent by the client in a [request](https://mockoon.com/articles/api-glossary/#request) or returned by the server in a [response](https://mockoon.com/articles/api-glossary/#response)**. In the context of APIs, the payload typically contains the information needed to create, update, or retrieve a resource. The format of the payload can vary depending on the API and the specific endpoint being used. Common formats include JSON, XML, and form data.

See also: [Body](https://mockoon.com/articles/api-glossary/#body)

###  [](https://mockoon.com/articles/api-glossary/#polling)Polling

Polling is a technique used in APIs to **retrieve data from the server at regular intervals**. Instead of waiting for the server to push updates to the client (as with [webhooks](https://mockoon.com/articles/api-glossary/#webhooks)), the client repeatedly sends requests to the server to check for new data. This can be useful in scenarios where real-time updates are not critical, but it can also lead to increased server load and [latency](https://mockoon.com/articles/api-glossary/#latency).

##  [](https://mockoon.com/articles/api-glossary/#q)Q

###  [](https://mockoon.com/articles/api-glossary/#query-parameters)Query Parameters

A query parameter is an optional parameter added by a [client](https://mockoon.com/articles/api-glossary/#client), placed after the [route's](https://mockoon.com/articles/api-glossary/#route) path, and sent with the [request](https://mockoon.com/articles/api-glossary/#request). It allows the client to add more parameters to its request. They are separated from the path by an interrogation mark and represented as key-value pairs separated by ampersands. For example, in `/users?filter=active&sort=asc`, two query parameters are sent: a `filter` parameter set to `active`, and a `sort` parameter set to `asc`. It is up to the API [server](https://mockoon.com/articles/api-glossary/#server) to define which query parameters are available and needed.

See also: [Path parameters](https://mockoon.com/articles/api-glossary/#path-parameters)

##  [](https://mockoon.com/articles/api-glossary/#r)R

###  [](https://mockoon.com/articles/api-glossary/#rate-limiting)Rate limiting

Rate limiting is a technique used in APIs to **control the amount of incoming requests from clients within a specific time frame**, for example, allowing a maximum of 100 requests per minute per client. It helps prevent abuse, ensures fair usage, and protects the server from being overwhelmed by too many requests. When a client exceeds the allowed request limit, the server typically responds with a `429 Too Many Requests` [status code](https://mockoon.com/articles/api-glossary/#status-code-http).

###  [](https://mockoon.com/articles/api-glossary/#request)Request

A request is usually sent by a [client](https://mockoon.com/articles/api-glossary/#client) connecting to an API [server](https://mockoon.com/articles/api-glossary/#server) which will process it and send a [response](https://mockoon.com/articles/api-glossary/#response) back to the client.

See also: [Response](https://mockoon.com/articles/api-glossary/#response)

###  [](https://mockoon.com/articles/api-glossary/#response)Response

A response is built by a [server](https://mockoon.com/articles/api-glossary/#server) after processing a [request](https://mockoon.com/articles/api-glossary/#request) sent by the [client](https://mockoon.com/articles/api-glossary/#client). It usually contains the data requested by the client and information related to the execution of the request, like the [status code](https://mockoon.com/articles/api-glossary/#status-code-http).

See also: [Request](https://mockoon.com/articles/api-glossary/#request)

###  [](https://mockoon.com/articles/api-glossary/#resource)Resource

In [REST APIs](https://mockoon.com/articles/api-glossary/#rest-api), a resource is an object with a type, associated data, and optional sub-resources. They are usually interacted with individually or in collections through [endpoints](https://mockoon.com/articles/api-glossary/#endpoint). For example, an object of type `User`, which can be read individually on the `GET /users/{id}` endpoint.

See also: [CRUD](https://mockoon.com/articles/api-glossary/#crud-create-read-update-delete)

###  [](https://mockoon.com/articles/api-glossary/#rest-api)REST API

REST stands for **RE**presentational **S**tate **T**ransfer. It's a software architectural style that defines a set of constraints used to create standardized [APIs](https://mockoon.com/articles/api-glossary/#api-application-programming-interface). [Web APIs](https://mockoon.com/articles/api-glossary/#web-api) adhering to the REST architectural constraints are called RESTful APIs. RESTful APIs must follow six constraints: client-server architecture, statelessness, cacheability, layered system, code on demand, and uniform interface.

###  [](https://mockoon.com/articles/api-glossary/#route)Route

In [REST APIs](https://mockoon.com/articles/api-glossary/#rest-api), routes are couples of [HTTP methods](https://mockoon.com/articles/api-glossary/#methods-http-methods) and paths of an API, usually representing a action to be performed on a specific resource. For example, accessing information about the users or invoices would be done on routes named after the [resources](https://mockoon.com/articles/api-glossary/#resource) using the `GET` method: `GET company.com/api/users` or `GET company.com/api/invoices`.

See also: [CRUD](https://mockoon.com/articles/api-glossary/#crud-create-read-update-delete)

##  [](https://mockoon.com/articles/api-glossary/#s)S

###  [](https://mockoon.com/articles/api-glossary/#server)Server

A server is a piece of hardware or software providing functionalities to other programs or devices called [clients](https://mockoon.com/articles/api-glossary/#client). In a client-server architecture, servers can provide different functionalities or services, such as providing [resources](https://mockoon.com/articles/api-glossary/#resource) or content.  
Client-server systems usually implement a request-response model where the client sends a [request](https://mockoon.com/articles/api-glossary/#request) to the server, and the server returns a [response](https://mockoon.com/articles/api-glossary/#response) to the client after performing a server-side action.

See also: [Client](https://mockoon.com/articles/api-glossary/#client)

###  [](https://mockoon.com/articles/api-glossary/#status-code-http)Status code (HTTP)

An HTTP status code is added to the [response](https://mockoon.com/articles/api-glossary/#response) by the [server](https://mockoon.com/articles/api-glossary/#server) to indicate to the [client](https://mockoon.com/articles/api-glossary/#client) the status of its request without having to further analyze the other response's components (headers, body, etc.). The status code varies depending on the success of the action but also on its nature. More concretely, it's a number with three digits (between 100 and 599) associated with a name: `200 Success`, `404 Not Found`, etc. There are many status codes grouped into five main categories: informational responses (1xx), successes (2xx), redirections (3xx), client errors (4xx), and server errors (5xx).

##  [](https://mockoon.com/articles/api-glossary/#u)U

###  [](https://mockoon.com/articles/api-glossary/#url-uniform-resource-locator)URL (Uniform Resource Locator)

A URL is a reference to a web [resource](https://mockoon.com/articles/api-glossary/#resource) specifying its location on a network and a mechanism to retrieve this resource. A typical URL, like `https://company.com/api/users`, contains multiple information:

- The protocol used to reach the resource: `HTTPS`.
- The hostname: `company.com`.
- A path to the resource: `/api/users`.

##  [](https://mockoon.com/articles/api-glossary/#v)V

###  [](https://mockoon.com/articles/api-glossary/#versioning)Versioning

Versioning is the process of assigning unique version numbers to different iterations of an API. It allows developers to make changes, add features, or fix bugs without disrupting existing clients that rely on a specific version of the API. Common versioning strategies include:

- **URI Versioning**: Including the version number in the API endpoint URL (e.g., `/api/v1/users`).
- **Header Versioning**: Specifying the version in the request headers (e.g., `Accept: application/vnd.company.v1+json`).
- **Query Parameter Versioning**: Including the version as a query parameter (e.g., `/api/users?version=1`).

##  [](https://mockoon.com/articles/api-glossary/#w)W

###  [](https://mockoon.com/articles/api-glossary/#web-api)Web API

Web APIs are a specific type of [APIs](https://mockoon.com/articles/api-glossary/#api-application-programming-interface) that can be accessed over the web, frequently using the HTTP protocol. They usually involve a [client](https://mockoon.com/articles/api-glossary/#client) (your browser) and a server exposing [resources](https://mockoon.com/articles/api-glossary/#resource) publicly.

See also: [REST API](https://mockoon.com/articles/api-glossary/#rest-api)

###  [](https://mockoon.com/articles/api-glossary/#webhooks)Webhooks

Webhooks are a way for an API to **send real-time notifications or data to a client** when certain events occur. Instead of the client polling the API for updates, the API sends a [request](https://mockoon.com/articles/api-glossary/#request) to a predefined [URL](https://mockoon.com/articles/api-glossary/#url-uniform-resource-locator) (the webhook endpoint) when an event happens. This allows for more efficient communication and reduces the need for constant polling.

###  [](https://mockoon.com/articles/api-glossary/#websocket)WebSocket

WebSocket is a protocol that enables **full-duplex communication channels** over a single TCP connection. It allows for **real-time data exchange between a client and a server**, making it suitable for applications that require low latency and high interactivity, such as chat applications, online gaming, and live data feeds. WebSockets are often used in conjunction with APIs to provide real-time updates and notifications.