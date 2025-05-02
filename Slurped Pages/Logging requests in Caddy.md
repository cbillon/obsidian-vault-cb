---
link: https://thedevelopercafe.com/articles/logging-requests-in-caddy-373296425166
excerpt: In this article you going to learn how to log requests and various
  options available to configure logging using the log directive.
slurped: 2024-05-13T14:42:08.023Z
title: Logging requests in Caddy
---

In this article you going to learn how to log requests and various options available to configure logging using the [log](https://caddyserver.com/docs/caddyfile/directives/log) directive.

## Enable logging using log directive [#](https://thedevelopercafe.com/articles/logging-requests-in-caddy-373296425166#enable-logging-using-log-directive)

Use the `log` directive to enable request logs.

```
:80 {
	log

	handle {
		respond "Hello World"
	}
}
```

Visit `http://localhost:80` and you will see the request being logged.

Be default the log format is `json`.

```
{"level":"info","ts":1690074874.4119136,"logger":"http.log.access","msg":"handled request","request":{"remote_ip":"172.17.0.1","remote_port":"39286","proto":"HTTP/1.1","method":"GET","host":"localhost:8080","uri":"/","headers":{"Sec-Ch-Ua":["\"Not.A/Brand\";v=\"8\", \"Chromium\";v=\"114\", \"Brave\";v=\"114\""],"User-Agent":["Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36"],"Sec-Gpc":["1"],"Accept-Encoding":["gzip, deflate, br"],"Cache-Control":["max-age=0"],"Sec-Ch-Ua-Platform":["\"macOS\""],"Upgrade-Insecure-Requests":["1"],"Sec-Fetch-Mode":["navigate"],"Sec-Fetch-Dest":["document"],"Cookie":[],"Connection":["keep-alive"],"Sec-Fetch-User":["?1"],"Sec-Ch-Ua-Mobile":["?0"],"Accept":["text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8"],"Sec-Fetch-Site":["none"],"Accept-Language":["en-US,en;q=0.9"]}},"user_id":"","duration":0.000011793,"size":11,"status":200,"resp_headers":{"Server":["Caddy"],"Content-Type":["text/plain; charset=utf-8"]}}
```

## Change log format 

There two formats available `json` and `console`, the default is `json`.

Use the `format` directive and set it to `console` as follows.

```
:80 {
	log {
		format console
	}

	handle {
		respond "Hello World"
	}
}
```

Most of it is still json even after chaing it to `console`.

```
2023/07/23 02:49:05.900	INFO	http.log.access.log0	handled request	{"request": {"remote_ip": "172.17.0.1", "remote_port": "39862", "proto": "HTTP/1.1", "method": "GET", "host": "localhost:8080", "uri": "/", "headers": {"Upgrade-Insecure-Requests": ["1"], "Accept": ["text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8"], "Sec-Gpc": ["1"], "Sec-Fetch-Dest": ["document"], "Sec-Ch-Ua-Platform": ["\"macOS\""], "User-Agent": ["Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36"], "Accept-Language": ["en-US,en;q=0.9"], "Sec-Fetch-Site": ["none"], "Sec-Fetch-User": ["?1"], "Accept-Encoding": ["gzip, deflate, br"], "Cookie": [], "Connection": ["keep-alive"], "Cache-Control": ["max-age=0"], "Sec-Ch-Ua": ["\"Not.A/Brand\";v=\"8\", \"Chromium\";v=\"114\", \"Brave\";v=\"114\""], "Sec-Ch-Ua-Mobile": ["?0"], "Sec-Fetch-Mode": ["navigate"]}}, "user_id": "", "duration": 0.00001076, "size": 11, "status": 200, "resp_headers": {"Server": ["Caddy"], "Content-Type": ["text/plain; charset=utf-8"]}}
```

## Filter fields 

By default `log` spits out everything inside the log. But use can use the `filter` directive to perform operations on the fields as required.

For example, to hide the `Authorization` header from the log do as follows.

```
:80 {
	log {
		format filter {
			wrap console
			fields {
				request>headers>Authorization delete
			}
		}
	}

	handle {
		respond "Hello World"
	}
}
```

This will remove the `Authorization` header from the logs.

`filter` works slightly differently, it is set as a format value and the actual format (`json` or `console`) is set inside the `wrap` property.

You can perform many more operations using filter such as `replace` and `hash`. Read about all the filter options [here](https://caddyserver.com/docs/caddyfile/directives/log#filter).

Take another example, let's say you want to remove the `q` param from **query**, you would do it as follows.

```
:80 {
	log {
		format filter {
			wrap console
			fields {
				request>uri query {
					delete q
				}
			}
		}
	}

	handle {
		respond "Hello World"
	}
}
```

Read the [log](https://caddyserver.com/docs/caddyfile/directives/log) documentation to learn more about `query` and `cookie` filters.