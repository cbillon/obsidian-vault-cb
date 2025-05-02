---
link: https://thedevelopercafe.com/articles/reverse-proxy-in-caddy-5c40d2fe21fe
excerpt: In this article I am going to show you how to use Caddy as a reverse
  proxy and also covering some additional features such as multiple upstreams,
  health checks and load balancing.
slurped: 2024-05-13T14:00:44.104Z
title: Reverse Proxy in Caddy with load balancing and health checks
---

[Code for this articles is available on GitHub](https://github.com/gurleensethi/caddy-reverse-proxy-example)


In this article I am going to show you how to use Caddy as a reverse proxy and also covering some additional features such as multiple upstreams, health checks and load balancing.

## Mock Server 
Since you are doing reverse proxy, you need something to reverse proxy to. You can set up a **mock server** as you want, it has to be nothing special, just a basic server with an endpoint.

I am going to use a simple server written in [Go](https://go.dev/) that logs each request's **URL**, **Method** and **Headers**.

Don't worry if you are unable to understand the following piece of code.

```
package main

import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	if len(os.Args) < 3 {
		fmt.Println("Example: go run main.go <name> <port>")
		os.Exit(1)
	}

	serverName := os.Args[1]
	serverPort := os.Args[2]

	fmt.Printf("starting server %s on port %s\n", serverName, serverPort)

	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Printf("----> Received on server %s\n", serverName)

		fmt.Printf("URL: %s %s\n", r.Method, r.URL.String())

		fmt.Println("Headers:")
		for key, value := range r.Header {
			fmt.Printf("\t%s: %v\n", key, value)
		}

		w.Write([]byte("Hello From Server " + serverName))

		fmt.Println("<---- Response sent")
	})

	http.ListenAndServe(fmt.Sprintf(":%s", serverPort), nil)
}
```

The above takes in a **server name** and a **port**, if you have `go` installed you can run it using `go run main.go server-1 8881`.

## Reverse Proxy [#](https://thedevelopercafe.com/articles/reverse-proxy-in-caddy-5c40d2fe21fe#reverse-proxy)

Let's write a basic reverse proxy that listens on port **8000** and forwards all traffic to a server running on port **8881**.

![Caddy Reverse Proxy Simple](https://tdc-public.nyc3.digitaloceanspaces.com/photos/7ff560e8-09ae-44fc-a282-b14bd6c614a4-caddy-reverse-proxy-1.png)

Before doing this, start the mock server on port **8881**.

```
go run main.go server-1 8881
```

Now comes the Caddyfile.

```
:8000

reverse_proxy localhost:8881
```

Send a request on port **8000** and you get back response from the server.

```
$ curl -X GET localhost:8000

Hello From Server server-1
```

Meanwhile in the server console, you can see the complete request log.

```
$ go run main.go server-1 8881

----> Received on server server-1
URL: GET /
Headers:
	User-Agent: [curl/7.86.0]
	Accept: [*/*]
	X-Forwarded-For: [127.0.0.1]
	X-Forwarded-Host: [localhost:8000]
	X-Forwarded-Proto: [http]
	Accept-Encoding: [gzip]
<---- Response sent
```

## Multiple Upstreams [#](https://thedevelopercafe.com/articles/reverse-proxy-in-caddy-5c40d2fe21fe#multiple-upstreams)

You can proxy to **multiple upstream** servers by just providing additional network addresses.

![Caddy Reverse Proxy Multiple Upstreams](https://tdc-public.nyc3.digitaloceanspaces.com/photos/4422cce8-f2ef-4275-b762-0a1405c3c727-caddy-reverse-proxy-multiple-upstreams.png)

Before doing this, let's start another mock server on port **8882**.

```
go run main.go server-2 8882
```

```
:8000

reverse_proxy localhost:8881 localhost:8882
```

You can add as many backends as you want.

Sending multiple request to `localhost:8000` you will get back different responses.

```
$ curl -X GET localhost:8000

Hello From Server server-1

$ curl -X GET localhost:8000

Hello From Server server-1

$ curl -X GET localhost:8000

Hello From Server server-2
```

You can re-write the above Caddyfile using the [to](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy#to) directive as well.

```
:8000

reverse_proxy {
	to localhost:8881 localhost:8882
}
```

## Load Balancing [#](https://thedevelopercafe.com/articles/reverse-proxy-in-caddy-5c40d2fe21fe#load-balancing)

You must have noticed that when sending requests to port `:8000` the response we get back is random i.e. in no particular order from the two servers, sometimes you get response from a server more often than the other.

This is due to the **default** load balancer policy being `random`.

Let's see how you can configure a round robin load balancer policy for the reverse proxy.

![Caddy Round Robin](https://tdc-public.nyc3.digitaloceanspaces.com/photos/db8a94ed-4ddc-4181-8bd9-2d545a21925e-caddy-round-robin.png)

Start a new server called `server-3` on port `8883`.

```
go run main.go server-3 8883
```

And add it to the `Caddyfile`.

```
:8000

reverse_proxy {
	to localhost:8881 localhost:8882 localhost:8883
}
```

To configure the round robin policy use `lb_policy`.

```
:8000

reverse_proxy {
	to localhost:8881 localhost:8882 localhost:8883

	lb_policy round_robin
}
```

And thats is it, now if you make multiple requests, caddy will start sending those requests in a round robin manner to all the provided servers.

```
$ curl -X GET localhost:8000

Hello From Server server-1

$ curl -X GET localhost:8000

Hello From Server server-2

$ curl -X GET localhost:8000

Hello From Server server-3
```

There are many other load balancing policies available, refer to section on load balancer [here](https://caddyserver.com/docs/caddyfile/directives/reverse_proxy#load-balancing).

## Load Balancing Retries [#](https://thedevelopercafe.com/articles/reverse-proxy-in-caddy-5c40d2fe21fe#load-balancing-retries)

Let's say one our of your 3 servers is down, with the current setup (round robin), every 3rd request to the server will fail.

Stop the 3rd server and send 3 requests to port `8000`.

Below is the result of sending 3 requests, as you can see the last request returns with a `502` status code.  
![Caddy Failed Server](https://tdc-public.nyc3.digitaloceanspaces.com/photos/6b9501da-8ade-488c-ba68-9c6a04c676bd-caddy-failed-server.png)

With `lb_retries` you can have the request be retried multiple times before returning a `502` status.

```
:8000

reverse_proxy {
	to localhost:8881 localhost:8882 localhost:8883

	lb_policy round_robin
    lb_retries 2
}
```

Now if you send 3 requests, you can observe that the 3rd request gets routed to **server 2** instead of returning `502` status due to failed server 3.

![Caddy Load Balancer Retries](https://tdc-public.nyc3.digitaloceanspaces.com/photos/0b593008-68d1-47ce-b977-5081f9abdb54-caddy-lb-retries.png)

You can improve on this setup by using health checks.

## Health Checks 

Let's say you are load balancing between 10 servers and suddenly 4 of them went down, your `lb_retires` policy won't be enough to cover the case. Yes you can set it to `10` but then it would add latency to the request.

Using health checks Caddy can know which upstreams are unavailable and can completely avoid them.

```
:8000

reverse_proxy {
	to localhost:8881 localhost:8882 localhost:8883

	lb_policy round_robin
	lb_retries 2

	health_uri /
	health_interval 5s
	health_timeout 2s
	health_status 2xx
}
```

The above will setup a health check that will call the `/` endpoint for each _upstream_ every 5 seconds.

If the endpoint returns a non-`2xx` status code the health checks fails and caddy **will not use that upstream** to proxy the requests until the health checks starts passing again i.e. the upstream is back up and running.

If you are using the server i provided you with, you will start seeing health checks being logged in the terminal.

```
----> Received on server server-1
URL: GET /
Headers:
	User-Agent: [Go-http-client/1.1]
	Accept-Encoding: [gzip]
<---- Response sent
----> Received on server server-1
URL: GET /
Headers:
	Accept-Encoding: [gzip]
	User-Agent: [Go-http-client/1.1]
<---- Response sent
```

Now if **server 3** fails health check, caddy will stop routing requests to it.

![Caddy Health Check](https://tdc-public.nyc3.digitaloceanspaces.com/photos/dd9c1e35-ee5e-4605-ace3-f300580957e9-caddy-health-check.png)

## Handing Path Matches [#](https://thedevelopercafe.com/articles/reverse-proxy-in-caddy-5c40d2fe21fe#handing-path-matches)

Let's say you want to reverse proxy only the endpoints that start with `/api` and send them to the upstreams.

You can use `handle` for this.

Wrap the `reverse_proxy` directive with the `handle` directive.

```
:8000

handle /api* {
	reverse_proxy {
		to localhost:8881 localhost:8882 localhost:8883

		lb_policy round_robin
		lb_retries 2

		health_uri /
		health_interval 5s
		health_timeout 2s
		health_status 2xx
	}
}
```

For example, a request sent to `localhost:8000/api/testing` will be routed as `/api/testing` to one of the upstreams.

If you want to match `/api` path but not send it use `handle_path` instead, this will route `/api/testing` as `/testing` to upstreams.

![Caddy Handle Path](https://tdc-public.nyc3.digitaloceanspaces.com/photos/2fa4e5a7-718b-454e-a773-93fc54bcee7c-caddy-handle-path.png)