---
link: https://thedevelopercafe.com/articles/serving-static-files-using-caddy-8513e8f36e46
excerpt: In this article I am going to show you how to server static files using Caddy.
slurped: 2024-05-13T14:29:17.842Z
title: Serving static files using Caddy
---

In this article I am going to show you how to server static files using [Caddy](https://caddyserver.com/).

I am going to use Docker to run Caddy, take a quick look at [this](https://thedevelopercafe.com/articles/hello-world-in-caddy-using-docker-c8b18ee67f2f) article on how to run Caddy using Docker.

## Prepare index.html file
Prepare the `index.html` file which you will be serving using Caddy.

```
<html>
	<head>
		<title>Hello Caddy</title>
	</head>
	<body>
		<h1>Hello Caddy</h1>
	</body>
</html>
```

## Create the Caddyfile [#](https://thedevelopercafe.com/articles/serving-static-files-using-caddy-8513e8f36e46#create-the-caddyfile)

Let's start with a simple `Caddyfile` that runs the server on port `80` of the docker container.

```
:80
```

Run this simple file using the following command.

```
docker run --rm --name caddy -p 8080:80 -v ${PWD}/Caddyfile:/etc/caddy/Caddyfile caddy:latest
```

You will get the output.

```
$ docker run --rm --name caddy -p 8080:80 -v ${PWD}/Caddyfile:/etc/caddy/Caddyfile caddy:latest
{"level":"info","ts":1687325417.6084845,"msg":"using provided configuration","config_file":"/etc/caddy/Caddyfile","config_adapter":"caddyfile"}
{"level":"warn","ts":1687325417.6088786,"msg":"Caddyfile input is not formatted; run the 'caddy fmt' command to fix inconsistencies","adapter":"caddyfile","file":"/etc/caddy/Caddyfile","line":2}
{"level":"info","ts":1687325417.6095884,"logger":"admin","msg":"admin endpoint started","address":"localhost:2019","enforce_origin":false,"origins":["//localhost:2019","//[::1]:2019","//127.0.0.1:2019"]}
{"level":"info","ts":1687325417.6099453,"logger":"tls.cache.maintenance","msg":"started background certificate maintenance","cache":"0xc0000cff10"}
{"level":"warn","ts":1687325417.6103315,"logger":"http","msg":"server is listening only on the HTTP port, so no automatic HTTPS will be applied to this server","server_name":"srv0","http_port":80}
{"level":"info","ts":1687325417.6104715,"logger":"http.log","msg":"server running","name":"srv0","protocols":["h1","h2","h3"]}
{"level":"info","ts":1687325417.610657,"msg":"autosaved config (load with --resume flag)","file":"/config/caddy/autosave.json"}
{"level":"info","ts":1687325417.6106935,"msg":"serving initial configuration"}
{"level":"info","ts":1687325417.6107538,"logger":"tls","msg":"cleaning storage unit","description":"FileStorage:/data/caddy"}
{"level":"info","ts":1687325417.6108081,"logger":"tls","msg":"finished cleaning storage units"}
```

## file_server directive 

Use the [`file_server`](https://caddyserver.com/docs/caddyfile/directives/file_server) directive to setup a basic fileserver in Caddy.

```
:80

file_server
```

If you run the above `Caddyfile` nothing will show up if you visit `http://localhost:8080`, we still have to tell Caddy where to find the `index.html` file.

Let's say all the files we want to serve are under `/www/html` folder in the container. Use the [`root`](https://caddyserver.com/docs/caddyfile/directives/root) directive to tell Caddy which is the root folder to serve files from.

```
:80

root * /www/html
file_server
```

### Mounting file in docker container 

Since we are running **Caddy** in a **Docker** container, we need to mount our `index.html` inside the contianer.

You can do that by doing `-v ${PWD}/index.html:/www/html/index.html`, this will place the `index.html` file in the `/www/html` folder in the container.

```
docker run --rm --name caddy -p 8080:80 \
-v ${PWD}/Caddyfile:/etc/caddy/Caddyfile \
-v ${PWD}/index.html:/www/html/index.html \
caddy:latest
```

Visit [http://localhost:8080](http://localhost:8080/) and you should see **Hello Caddy** being displayed.