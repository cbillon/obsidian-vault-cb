---
link: https://thedevelopercafe.com/articles/authenticate-requests-in-caddy-using-forward-auth-85d8daa1c6a8
excerpt: In this article you are going to learn how to authorize requests in
  Caddy using forward_auth directive.
slurped: 2024-05-13T14:34:28.563Z
title: Authenticate Requests in Caddy using forward auth
---

In this article you are going to learn how to authorize requests in [Caddy](https://caddyserver.com/) using [`forward_auth`](https://caddyserver.com/docs/caddyfile/directives/forward_auth) directive.

Below is what you are trying to achieve.

There is an **auth server** that authenticates requests, and there is a **resource server** that returns the requested resource.

You want each request to get authenticated by the **auth server** and get to the **resource server** only if successfully authenticated.

![Caddy Forward Auth](https://tdc-public.nyc3.digitaloceanspaces.com/photos/0cf72bb8-3a5a-48d7-92d0-9d7186fdf176-caddy-forward-auth.png)

## Mock Auth Server 

I am going to provide you with a very simple auth server written in **Go**, you are free to rewrite this in any language you want. Its very simple.

```
package main

import (
	"fmt"
	"net/http"
)

func main() {
	fmt.Println("Starting server on port 8880")

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		if r.Header.Get("Authorization") == "" {
			w.WriteHeader(http.StatusForbidden)
			return
		}

		w.WriteHeader(http.StatusOK)
		w.Write([]byte("Authorized!"))
	})

	http.ListenAndServe(":8880", nil)
}
```

Here is what the server does (implement this in any other language you want).

1. Starts on port `8880`.
2. If the request has any value in `Authorization` header, returns status `200`.
3. If the request does not have any value in `Authorization` header, returns status `403`.

(Reason for these status codes explained later)

You can run this program by doing

```
go run main.go
```

## Caddyfile

Write a simple `Caddyfile` that returns a value on the route `/resource`.

```
:8080 {
    handle /resource {
        respond "Resource X"
    }
}
```

The above `Caddyfile` starts listening on port `8080` and returns **Resource X** on pathg `/resource`.

Run the caddy server by doing.

```
caddy run --config Caddyfile
```

([You can also run Caddy using Docker](https://thedevelopercafe.com/articles/hello-world-in-caddy-using-docker-c8b18ee67f2f))

Send a request to `localhost:8080` you should get a response.

```
curl -i "localhost:8080/resource"
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Server: Caddy
Date: Sun, 09 Jul 2023 21:59:47 GMT
Content-Length: 10

Resource X%
```

## Protecting with forward_auth 

You want to protect `/resource` endpoint and only allow authenticated requests to access it.

Use the `forward_auth` directive to send a request to the **auth server** running on port `8880` in order to authenticate the current request.

```
:8080 {
    handle /resource {
        forward_auth localhost:8880 {
            uri /
        }

        respond "Resource X"
    }
}
```

This is how the above works.

1. `forward_auth` sends a copy of each request to **auth server** running on `localhost:8880` at path `/`.
2. If the auth server returns a response with status `2XX` caddy continues with the request.
3. If the auth sever returns a non-`2XX` status code, the non-`2XX` response is sent back the client.

Make a request with no `Authorization` header.

```
curl -i "localhost:8080/resource"
HTTP/1.1 403 Forbidden
Content-Length: 0
Date: Sun, 09 Jul 2023 22:07:40 GMT
Server: Caddy
```

Make a request with a `Authorization` header.

```
curl -i "localhost:8080/resource" --header 'Authorization: any-value'
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Server: Caddy
Date: Sun, 09 Jul 2023 22:07:56 GMT
Content-Length: 10

Resource X%
```

Under the hood, `forward_auth` uses `reverse_proxy` directive, read the explanation in Caddy docs [here](https://caddyserver.com/docs/caddyfile/directives/forward_auth#expanded-form).

Below is a more detailed, complicated looking diagram showing the flow of `forward_auth`.

![Caddy Forward Auth Explained](https://tdc-public.nyc3.digitaloceanspaces.com/photos/78068b02-3b5a-4fdd-837a-aecf715b7fde-caddy-forward-auth-explained.png)

[Forward Auth Documentation](https://caddyserver.com/docs/caddyfile/directives/forward_auth)