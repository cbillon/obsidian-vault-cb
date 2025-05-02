---
link: https://thedevelopercafe.com/articles/variables-environment-variables-and-placeholders-in-caddy-363ab4cf724a
excerpt: In this article you are going to learn about placeholders, varibles and
  environment variables in Caddy.
slurped: 2024-05-13T14:31:41.884Z
title: Placeholders, Variables and Environment Variables in Caddy
---

In this article you are going to learn about placeholders, varibles and environment variables in Caddy.

## Placeholders 

You can access placeholders in Caddy using curly braces `{ }` to access dynamic values inside caddy's configuration. Some of these dynamic values are built in and some are defined by you.

For example, to get current time use `time.now`.

```
:80 {
	respond {time.now}
}
```

This will return the current time.

[Here is a complete list of some built in placeholders](https://caddyserver.com/docs/conventions#placeholders).

## Variables 

You can you define variables using the `vars` keyword.

The defined variable is to use with `vars.*`. For example, to access a variable named `root` use `vars.root`.

```
:80 {
	vars root https://example.com

	handle {
		respond "{vars.root}"
	}
}
```

To define multiple varibles use the block form of `vars`.

```
:80 {
	vars {
		root https://example.com
		message "Hello, World!"
	}

	handle {
		respond "{vars.root} {vars.message}"
	}
}
```

You can define variable values based on route as well. Below is how you can have different value in a variable depending on the path prefix.

```
:80 {
	vars base "/app"
	vars /admin* base "/admin"
	vars /manage* base "/manage"

	handle {
		respond "{vars.base}"
	}
}
```

Some examples of evaluation:

- Path: `/123` -> base: `/app`
- Path: `/admin/settings` -> base: `/admin`
- Path `/manage/users/1` -> base: `/manage`

## Environment Variables
Envrionment variables can be accessed from using the `{env.*}` placeholder.

For example, if you have an envrionment varible `BASE_URL: https://example.com`.

```
BASE_URL=https://example.com caddy run --config Caddyfile
```

You can access it using `env.BASE_URL`.

```
:80 {
	respond {env.BASE_URL}
}
```

```
$ curl localhost
https://example.com
```

You can also use the `{$ENV_NAME}` format to access envrionment variables in Caddy.

```
:80 {
	respond {$BASE_URL}
}
```

I personally prefer `{$}` over `{env.}`, keeps it separate from all other placeholders.