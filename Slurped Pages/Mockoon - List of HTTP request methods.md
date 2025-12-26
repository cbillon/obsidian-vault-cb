---
link: https://mockoon.com/articles/list-http-request-methods/
site: Mockoon
excerpt: This article provides a comprehensive list of HTTP request methods, their definitions, and use cases.
slurped: 2025-12-22T17:32
title: Mockoon - List of HTTP request methods
tags:
  - api
---

You may have heard about the different HTTP request methods, but what are they, and how do they differ? In this article you will find a list of all HTTP request methods, their definitions, and their use cases.

##  [](https://mockoon.com/articles/list-http-request-methods/#what-is-an-http-request-method)What is an HTTP request method?

An **HTTP request method** (or verb) is a part of the HTTP protocol that indicates the desired action to be performed on a specific resource identified by a URL. Each method has a specific meaning and is used for different purposes in web communication. HTTP request methods are used in RESTful APIs to define the type of operation to be performed on the resources. They are part of the HTTP protocol, which is the foundation of data communication on the World Wide Web.

To learn more about HTTP methods and REST APIs, you can read our [API guide part 2: REST(ful) APIs](https://mockoon.com/articles/api-guide-rest-api-components/).

##  [](https://mockoon.com/articles/list-http-request-methods/#list-of-http-request-methods)List of HTTP request methods

Here is a nearly exhaustive list of HTTP request methods, their definitions, and their use cases. Note that some methods are less commonly used or specific to certain protocols or applications.

The most commonly used HTTP request methods are: [**GET**](https://mockoon.com/articles/list-http-request-methods/#get), [**POST**](https://mockoon.com/articles/list-http-request-methods/#post), [**PUT**](https://mockoon.com/articles/list-http-request-methods/#put), [**DELETE**](https://mockoon.com/articles/list-http-request-methods/#delete), [**PATCH**](https://mockoon.com/articles/list-http-request-methods/#patch), [**OPTIONS**](https://mockoon.com/articles/list-http-request-methods/#options), and [**HEAD**](https://mockoon.com/articles/list-http-request-methods/#head). These methods are widely supported and used in RESTful APIs.

[ACL](https://mockoon.com/articles/list-http-request-methods/#acl)  
[BASELINE-CONTROL](https://mockoon.com/articles/list-http-request-methods/#baseline-control)  
[BIND](https://mockoon.com/articles/list-http-request-methods/#bind)  
[CHECKIN](https://mockoon.com/articles/list-http-request-methods/#checkin)  
[CHECKOUT](https://mockoon.com/articles/list-http-request-methods/#checkout)  
[CONNECT](https://mockoon.com/articles/list-http-request-methods/#connect)  
[COPY](https://mockoon.com/articles/list-http-request-methods/#copy)  
[DELETE](https://mockoon.com/articles/list-http-request-methods/#delete)  
[GET](https://mockoon.com/articles/list-http-request-methods/#get)  
[HEAD](https://mockoon.com/articles/list-http-request-methods/#head)  
[LABEL](https://mockoon.com/articles/list-http-request-methods/#label)  
[LINK](https://mockoon.com/articles/list-http-request-methods/#link)  
[LOCK](https://mockoon.com/articles/list-http-request-methods/#lock)  
[M-SEARCH](https://mockoon.com/articles/list-http-request-methods/#m-search)  
[MERGE](https://mockoon.com/articles/list-http-request-methods/#merge)  
[MKACTIVITY](https://mockoon.com/articles/list-http-request-methods/#mkactivity)  
[MKCALENDAR](https://mockoon.com/articles/list-http-request-methods/#mkcalendar)  
[MKCOL](https://mockoon.com/articles/list-http-request-methods/#mkcol)  
[MKREDIRECTREF](https://mockoon.com/articles/list-http-request-methods/#mkredirectref)  
[MKWORKSPACE](https://mockoon.com/articles/list-http-request-methods/#mkworkspace)  
[MOVE](https://mockoon.com/articles/list-http-request-methods/#move)  
[NOTIFY](https://mockoon.com/articles/list-http-request-methods/#notify)  
[OPTIONS](https://mockoon.com/articles/list-http-request-methods/#options)  
[ORDERPATCH](https://mockoon.com/articles/list-http-request-methods/#orderpatch)  
[PATCH](https://mockoon.com/articles/list-http-request-methods/#patch)  
[POST](https://mockoon.com/articles/list-http-request-methods/#post)  
[PROPFIND](https://mockoon.com/articles/list-http-request-methods/#propfind)  
[PROPPATCH](https://mockoon.com/articles/list-http-request-methods/#proppatch)  
[PURGE](https://mockoon.com/articles/list-http-request-methods/#purge)  
[PUT](https://mockoon.com/articles/list-http-request-methods/#put)  
[QUERY](https://mockoon.com/articles/list-http-request-methods/#query)  
[REBIND](https://mockoon.com/articles/list-http-request-methods/#rebind)  
[REPORT](https://mockoon.com/articles/list-http-request-methods/#report)  
[SEARCH](https://mockoon.com/articles/list-http-request-methods/#search)  
[SOURCE](https://mockoon.com/articles/list-http-request-methods/#source)  
[SUBSCRIBE](https://mockoon.com/articles/list-http-request-methods/#subscribe)  
[TRACE](https://mockoon.com/articles/list-http-request-methods/#trace)  
[UNBIND](https://mockoon.com/articles/list-http-request-methods/#unbind)  
[UNCHECKOUT](https://mockoon.com/articles/list-http-request-methods/#uncheckout) [UNLINK](https://mockoon.com/articles/list-http-request-methods/#unlink)  
[UNLOCK](https://mockoon.com/articles/list-http-request-methods/#unlock)  
[UNSUBSCRIBE](https://mockoon.com/articles/list-http-request-methods/#unsubscribe)  
[UPDATE](https://mockoon.com/articles/list-http-request-methods/#update)  
[UPDATEREDIRECTREF](https://mockoon.com/articles/list-http-request-methods/#updateredirectref)  
[VERSION-CONTROL](https://mockoon.com/articles/list-http-request-methods/#version-control)

###  [](https://mockoon.com/articles/list-http-request-methods/#acl)ACL

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3744)**

The **ACL** (Access Control List) method is used to retrieve or modify the access control list of a resource. It allows clients to manage permissions for users or groups on a specific resource.

###  [](https://mockoon.com/articles/list-http-request-methods/#baseline-control)BASELINE-CONTROL

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **BASELINE-CONTROL** method is used to manage baselines in a version-controlled resource. It allows clients to create, modify, or delete baselines, which are snapshots of a resource at a specific point in time.

###  [](https://mockoon.com/articles/list-http-request-methods/#bind)BIND

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc5842)**

The **BIND** method is used to bind a resource to a new URI. It allows clients to create a new binding for a resource, making it accessible under a different URI.

###  [](https://mockoon.com/articles/list-http-request-methods/#checkin)CHECKIN

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **CHECKIN** method is used to submit changes to a version-controlled resource. It allows clients to create a new version of a resource by checking in the modified content.

###  [](https://mockoon.com/articles/list-http-request-methods/#checkout)CHECKOUT

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **CHECKOUT** method is used to allow modifications to a version-controlled resource.

###  [](https://mockoon.com/articles/list-http-request-methods/#connect)CONNECT

The **CONNECT** method is used to establish a tunnel to the server identified by a given URI. It is primarily used for proxy servers to create a connection to a remote server, often for HTTPS communication.

###  [](https://mockoon.com/articles/list-http-request-methods/#copy)COPY

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4918)**

The **COPY** method is used to create a copy of a resource at a different URI.

###  [](https://mockoon.com/articles/list-http-request-methods/#delete)DELETE

The **DELETE** method is used to remove a resource identified by a given URI. Example: `DELETE /api/v1/users/123` would delete the user with ID 123.

###  [](https://mockoon.com/articles/list-http-request-methods/#get)GET

The **GET** method is used to retrieve a representation of a resource identified by a given URI. It is the most common method used in [RESTful APIs](https://mockoon.com/articles/api-guide-what-are-rest-api/). Example: `GET /api/v1/users/123` would retrieve the user with ID 123.

###  [](https://mockoon.com/articles/list-http-request-methods/#head)HEAD

The **HEAD** method is similar to the GET method, but it only retrieves the [headers](https://mockoon.com/articles/api-glossary/#header) of a resource without the [body](https://mockoon.com/articles/api-glossary/#body). It is often used to check if a resource exists or to get metadata about a resource. Example: `HEAD /api/v1/users/123` would return the headers for the user with ID 123 without the user data.

###  [](https://mockoon.com/articles/list-http-request-methods/#label)LABEL

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **LABEL** method is used to assign a label to a version-controlled resource. It allows clients to categorize or tag resources with specific labels for easier identification and management.

###  [](https://mockoon.com/articles/list-http-request-methods/#link)LINK

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc2068)**

The **LINK** method is used to create a link between two resources. It allows clients to establish relationships between resources by creating a link from one resource to another.

###  [](https://mockoon.com/articles/list-http-request-methods/#lock)LOCK

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4918)**

The **LOCK** method is used to lock a resource to prevent other clients from modifying it. It allows clients to obtain a lock on a resource, ensuring that no other client can make changes until the lock is released.

###  [](https://mockoon.com/articles/list-http-request-methods/#m-search)M-SEARCH

Specific to SSDP (Simple Service Discovery Protocol)

The **M-SEARCH** method is used in the context of SSDP to discover devices and services on a network. It allows clients to send a multicast request to find available resources.

###  [](https://mockoon.com/articles/list-http-request-methods/#merge)MERGE

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **MERGE** method is used to merge changes from one resource into another. It allows clients to combine the state of two resources, typically in a version-controlled environment.

###  [](https://mockoon.com/articles/list-http-request-methods/#mkactivity)MKACTIVITY

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **MKACTIVITY** method is used to create a new activity resource in a version-controlled environment. It allows clients to initiate a new set of changes to a resource.

###  [](https://mockoon.com/articles/list-http-request-methods/#mkcalendar)MKCALENDAR

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4791)**

The **MKCALENDAR** method is used to create a new calendar resource. It allows clients to create a calendar collection for managing events and schedules.

###  [](https://mockoon.com/articles/list-http-request-methods/#mkcol)MKCOL

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4918)**

The **MKCOL** method is used to create a new collection (directory) at a specified URI. It allows clients to create a new folder or directory in a WebDAV server.

###  [](https://mockoon.com/articles/list-http-request-methods/#mkredirectref)MKREDIRECTREF

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4437)**

The **MKREDIRECTREF** method is used to create a redirect reference to a resource. It allows clients to create a reference that points to another resource, enabling redirection.

###  [](https://mockoon.com/articles/list-http-request-methods/#mkworkspace)MKWORKSPACE

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **MKWORKSPACE** method is used to create a new workspace resource. It allows clients to create a workspace for organizing related resources in a WebDAV server.

###  [](https://mockoon.com/articles/list-http-request-methods/#move)MOVE

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4918)**

The **MOVE** method is used to move a resource from one URI to another. It allows clients to change the location of a resource by moving it to a new URI.

###  [](https://mockoon.com/articles/list-http-request-methods/#notify)NOTIFY

**Specific to WebDAV**

The **NOTIFY** method is used to send notifications about changes to a resource. It allows clients to receive updates or alerts when a resource is modified or updated.

###  [](https://mockoon.com/articles/list-http-request-methods/#options)OPTIONS

The **OPTIONS** method is used to retrieve the supported [HTTP methods](https://mockoon.com/articles/api-glossary/#methods-http-methods) and other options for a specific [resource](https://mockoon.com/articles/api-glossary/#resource). It allows clients to discover the capabilities of a server or resource. Example: `OPTIONS /api/v1/users/123` would return the allowed methods for the user with ID 123.

###  [](https://mockoon.com/articles/list-http-request-methods/#orderpatch)ORDERPATCH

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3648)**

The **ORDERPATCH** method is used to modify the order of resources in a collection. It allows clients to change the sequence or arrangement of resources within a collection.

###  [](https://mockoon.com/articles/list-http-request-methods/#patch)PATCH

The **PATCH** method is used to apply partial modifications to a resource. It allows clients to update specific fields or properties of a resource without sending the entire representation. Example: `PATCH /api/v1/users/123` with a body containing the fields to update.

###  [](https://mockoon.com/articles/list-http-request-methods/#post)POST

The **POST** method is used to submit data to a server to create a new resource or perform an action. It is often used to send data in the request body, such as creating a new user or submitting a form. Example: `POST /api/v1/users` with a body containing the user data.

###  [](https://mockoon.com/articles/list-http-request-methods/#propfind)PROPFIND

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4918)**

The **PROPFIND** method is used to retrieve properties of a resource. It allows clients to request metadata or attributes associated with a resource, such as its creation date, last modified date, or custom properties.

###  [](https://mockoon.com/articles/list-http-request-methods/#proppatch)PROPPATCH

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4918)**

The **PROPPATCH** method is used to modify properties of a resource. It allows clients to update or set specific properties associated with a resource, such as changing its title or description.

###  [](https://mockoon.com/articles/list-http-request-methods/#purge)PURGE

The **PURGE** method is used to remove cached content from a cache server. It is often used in content delivery networks (CDNs) to invalidate cached resources and ensure that clients receive the most up-to-date content.

###  [](https://mockoon.com/articles/list-http-request-methods/#put)PUT

The **PUT** method is used to create or update a resource at a specified URI. It allows clients to send the entire representation of a resource in the request body. If the resource already exists, it will be replaced; if it does not exist, it will be created. Example: `PUT /api/v1/users/123` with a body containing the user data.

###  [](https://mockoon.com/articles/list-http-request-methods/#query)QUERY

**[New proposal, not yet standardized](https://datatracker.ietf.org/doc/draft-ietf-httpbis-safe-method-w-body/)**

The **QUERY** method is used to retrieve data from a resource based on specific criteria or parameters. It is similar to the GET method but will allow for a request body to be included, enabling more complex queries. This method is not yet widely supported or standardized.

###  [](https://mockoon.com/articles/list-http-request-methods/#rebind)REBIND

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc5842)**

The **REBIND** method is used to change the binding of a resource to a new URI. It allows clients to update the URI associated with a resource without changing its content.

###  [](https://mockoon.com/articles/list-http-request-methods/#report)REPORT

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **REPORT** method is used to request a report or summary of information about a resource. It allows clients to retrieve specific data or statistics related to a resource, such as version history or access control information.

###  [](https://mockoon.com/articles/list-http-request-methods/#search)SEARCH

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc5323)**

The **SEARCH** method is used to perform a search operation on a resource or collection. It allows clients to query resources based on specific criteria, such as keywords or properties.

###  [](https://mockoon.com/articles/list-http-request-methods/#source)SOURCE

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4918)**

The **SOURCE** method is used to retrieve the source code or content of a resource. It allows clients to access the underlying data or representation of a resource, typically in a format suitable for further processing or analysis.

###  [](https://mockoon.com/articles/list-http-request-methods/#subscribe)SUBSCRIBE

**Specific to WebDAV**

The **SUBSCRIBE** method is used to subscribe to notifications or events related to a resource. It allows clients to receive updates or alerts when changes occur to a resource, such as modifications or new versions.

###  [](https://mockoon.com/articles/list-http-request-methods/#trace)TRACE

The **TRACE** method is used to perform a diagnostic trace of the request path. It allows clients to see the exact request and response headers as they are sent and received by the server. It is primarily used for debugging purposes.

###  [](https://mockoon.com/articles/list-http-request-methods/#unbind)UNBIND

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc5842)**

The **UNBIND** method is used to remove a binding of a resource from a specific URI. It allows clients to delete the association between a resource and its URI without affecting the resource itself.

###  [](https://mockoon.com/articles/list-http-request-methods/#uncheckout)UNCHECKOUT

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **UNCHECKOUT** method is used to cancel a checkout operation on a version-controlled resource. It allows clients to discard changes made during a checkout and revert the resource to its previous state.

###  [](https://mockoon.com/articles/list-http-request-methods/#unlink)UNLINK

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc2068)**

The **UNLINK** method is used to remove a link between two resources. It allows clients to delete a previously established link without affecting the resources themselves.

###  [](https://mockoon.com/articles/list-http-request-methods/#unlock)UNLOCK

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4918)**

The **UNLOCK** method is used to release a lock on a resource. It allows clients to remove a lock previously obtained with the LOCK method, enabling other clients to modify the resource.

###  [](https://mockoon.com/articles/list-http-request-methods/#unsubscribe)UNSUBSCRIBE

**Specific to WebDAV**

The **UNSUBSCRIBE** method is used to cancel a subscription to notifications or events related to a resource. It allows clients to stop receiving updates or alerts when changes occur to a resource.

###  [](https://mockoon.com/articles/list-http-request-methods/#update)UPDATE

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **UPDATE** method is used to modify a version-controlled resource. It allows clients to apply changes to a resource without creating a new version, typically used in conjunction with version control systems.

###  [](https://mockoon.com/articles/list-http-request-methods/#updateredirectref)UPDATEREDIRECTREF

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc4437)**

The **UPDATEREDIRECTREF** method is used to update a redirect reference to a resource. It allows clients to change the target URI of a redirect reference, enabling redirection to a different resource.

###  [](https://mockoon.com/articles/list-http-request-methods/#version-control)VERSION-CONTROL

**[Specific to WebDAV](https://datatracker.ietf.org/doc/html/rfc3253)**

The **VERSION-CONTROL** method is used to create a new version-controlled resource. It allows clients to initiate version control for a resource, enabling tracking of changes and versions over time.

###  [](https://mockoon.com/articles/list-http-request-methods/#custom-http-request-methods)Custom HTTP request methods?

While the methods listed above are standardized and widely used, it is possible to define custom HTTP request methods for specific applications or protocols. However, custom methods should be used with caution, as they may not be supported by all clients or servers and can lead to compatibility issues.