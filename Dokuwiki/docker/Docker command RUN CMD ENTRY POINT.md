---
tags:
  - docker
  - dockerfile
---
## 1. Overview

In a Dockerfile, we often encounter instructions like _run_, _cmd,_ or _entrypoint_. At first glance, they are all used for specifying and running commands. But what’s the difference between them? And how do they interact with one another?

In this tutorial, we’ll answer these questions. We’ll present what each of these instructions does and how they work. We’ll also look at what role they play in building an image and running a [Docker container](https://www.baeldung.com/docker-images-vs-containers).

## 2. Setup

To start, let’s create a script, _log-event.sh._ It simply adds one line to a file and then prints it:

```bash
#!/bin/sh

echo `date` $@ >> log.txt;
cat log.txt;
```

And now, let’s create a simple Dockerfile:

```bash
FROM alpine
ADD log-event.sh /
```

It’ll make use of our script by appending lines to _log.txt_ in different scenarios.

## 3. The _run_ Command

The _run_ instruction executes when we build the image. That means the command passed to _run_ executes on top of the current image in a new layer. Then the result is committed to the image. Let’s see how this looks in action.

Firstly, we’ll add a _run_ instruction to our Dockerfile:

```bash
FROM alpine
ADD log-event.sh /
RUN ["/log-event.sh", "image created"]
```

Secondly, let’s build our image with:

```bash
docker build -t myimage .
```

Now we expect to have a Docker image containing a _log.txt_ file with one _image created_ line inside. Let’s check this by running a container based on the image:

```bash
docker run myimage cat log.txt
```

When listing the contents of the file, we’ll see an output like this:

```bash
Fri Sep 18 20:31:12 UTC 2020 image created
```

If we run the container several times, we’ll see that the date in our log file doesn’t change. This makes sense because the **_run_ step executes at image build time**, not at the container runtime.

Let’s now build our image again. We notice the creation time in our log didn’t change. This happens because **[Docker caches the result](https://www.baeldung.com/linux/docker-build-cache) for the run instruction** if the Dockerfile didn’t change. If we want to invalidate the cache, we need to pass the _–no-cache_ option to the build command.

## 4. The _cmd_ Command

With the **_cmd_ instruction, we can specify a default command that executes when the container is starting.** Let’s add a _cmd_ entry to our Dockerfile and see how it works:

```bash
...
RUN ["/log-event.sh", "image created"]
CMD ["/log-event.sh", "container started"]
```

After building the image, let’s now run it and check the output:

```bash
$ docker run myimage
Fri Sep 18 18:27:49 UTC 2020 image created
Fri Sep 18 18:34:06 UTC 2020 container started
```

If we run this multiple times, we’ll see that the _image created_ entry stays the same. But the _container started_ entry updates with every run. This shows how _cmd_ indeed executes every time the container starts.

Notice we’ve used a slightly different _docker run_ command to start our container this time. Let’s see what happens if we run the same command as before:

```bash
$ docker run myimage cat log.txt
Fri Sep 18 18:27:49 UTC 2020 image created
```

This time the _cmd_ specified in the Dockerfile is ignored. That’s because we have specified arguments to the _docker run_ command.

Let’s move on now and see what happens if we have more than one _cmd_ entry in the Dockerfile. Let’s add a new entry that will display another message:

```bash
...
RUN ["/log-event.sh", "image created"]
CMD ["/log-event.sh", "container started"]
CMD ["/log-event.sh", "container running"]
```

After building the image and running the container again, we’ll find the following output:

```bash
$ docker run myimage
Fri Sep 18 18:49:44 UTC 2020 image created
Fri Sep 18 18:49:58 UTC 2020 container running
```

As we can see, the _container started_ entry is not present, only the _container running_ is_._ That’s because **only the last cmd is invoked if more than one is specified.**

## 5. The _entrypoint_ Command

As we saw above, _cmd_ is ignored if passing any arguments when starting the container. What if we want more flexibility? Let’s say we want to customize the appended text and pass it as an argument to the _docker run_ command. For this purpose, let’s use _entrypoint._ We’ll specify the default command to run when the container starts. Moreover, we’re now able to provide extra arguments.

Let’s replace the _cmd_ entry in our Dockerfile with _entrypoint:_

```bash
...
RUN ["/log-event.sh", "image created"]
ENTRYPOINT ["/log-event.sh"]
```

Now let’s run the container by providing a custom text entry:

```bash
$ docker run myimage container running now
Fri Sep 18 20:57:20 UTC 2020 image created
Fri Sep 18 20:59:51 UTC 2020 container running now
```

We can see how **_entrypoint_ behaves similarly to** _**cmd**._ **And in addition, it allows us to customize the command executed at startup.**

Like with _cmd_, in case of multiple _entrypoint_ entries, only the last one is considered.

## 6. Interactions Between _cmd_ and _entrypoint

We have used both _cmd_ and _entrypoint_ to define the command executed when running the container. Let’s now move on and see how to use _cmd_ and _entrypoint_ in combination.

One such use-case is to define default arguments for _entrypoint._ Let’s add a _cmd_ entry after _entrypoint_ in our Dockerfile:

```bash
...
RUN ["/log-event.sh", "image created"]
ENTRYPOINT ["/log-event.sh"]
CMD ["container started"]
```

Now, let’s run our container without providing any arguments, and with the defaults specified in _cmd_:

```bash
$ docker run myimage
Fri Sep 18 21:26:12 UTC 2020 image created
Fri Sep 18 21:26:18 UTC 2020 container started
```

We can also override them if we choose so:

```bash
$ docker run myimage custom event
Fri Sep 18 21:26:12 UTC 2020 image created
Fri Sep 18 21:27:25 UTC 2020 custom event
```

Something to note is the different behavior of _entrypoint_ when used in its shell form. Let’s update the _entrypoint_ in our Dockerfile:

```bash
...
RUN ["/log-event.sh", "image created"]
ENTRYPOINT /log-event.sh
CMD ["container started"]
```

In this situation, when running the container, we’ll see how Docker ignores any arguments passed to either _docker run_ or _cmd_.

## 7. Conclusion

In this article, we’ve seen the differences and similarities between the Docker instructions: _run_, _cmd,_ and _entrypoint_. We’ve observed at what point they get invoked. Also, we’ve taken a look at their uses and how they work together.