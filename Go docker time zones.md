---
tags:
  - go
  - docker
  - time_zone
---
Yesterday I received a requirement for one of my Go services to convert some data from a specific timezone to UTC. As the process goes, I made the changes, tested it out locally on my machine, and pushed the code to the Github repo. After the changes got deployed to the test environment, I checked if the service is working correctly and found this error:

Copy

```
2020/11/06 08:29:33 error occurred during import and sync execution:
 unknown time zone Europe/Sarajevo
```

### [](https://lalatron.hashnode.dev/a-story-about-go-docker-and-time-zones#everything-is-better-with-an-example "Permalink")Everything is better with an example

Consider the code below.

Copy

```
package main

import (
    "fmt"
    "os"
    "time"
)

func main() {

    fmt.Println("Local time: ", time.Now())
    fmt.Println("UTC time: ", time.Now().UTC())

    //load timezone
    tz, err := time.LoadLocation("Europe/Sarajevo")
    if err != nil {
        fmt.Println(err)
        os.Exit(-1)
    }

    fmt.Println("Sarajevo time: ", time.Now().In(tz))
}
```

If you run this locally, it should work. If you run this in some [Go playground](https://play.golang.org/p/Xe7Kfc8kg9M), it will show the results. It might not be the results you expected, but that's because the instance on which the code is run might have different time settings. In any case, you should get some results.

Now, let's build and run the app inside a Docker container, with a nice multi-stage build. Below is a Dockerfile I used.

Copy

```
FROM golang:1.14-alpine
WORKDIR /app
ADD . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp

FROM scratch as final
COPY --from=0 /app/myapp .
CMD ["/myapp"]
```

The output:

Copy

```
Local time:  2020-11-07 17:55:09.1324588 +0000 UTC m=+0.000137401
UTC time:  2020-11-07 17:55:09.1326111 +0000 UTC
unknown time zone Europe/Sarajevo
```

**Note**

_As you might have seen, I've used Go 1.14 in my examples. You might get different errors based on the image you use for building the app. For example, I have also tried this with the `golang:alpine` image, and this was the output:_

Copy

```
Local time:  2020-11-07 08:29:59.037473424 +0000 UTC m=+0.000692888
UTC time:  2020-11-07 08:29:59.037545751 +0000 UTC
open /usr/local/go/lib/time/zoneinfo.zip: no such file or directory
```

### [](https://lalatron.hashnode.dev/a-story-about-go-docker-and-time-zones#wait-whaaat "Permalink")Wait, whaaat??

It seems that either the file cannot be found or that the used time zone is not in the available `zoneinfo.zip` file. This problem stems from using the `scratch` image in my Docker multi-stage build. We all love the `scratch` image because of its size, but because it is an image for a bare-bone container, it doesn't have all dependencies you expect to have when running an app within it. In my case, there is no time zone information that I needed.

### [](https://lalatron.hashnode.dev/a-story-about-go-docker-and-time-zones#how-to-fix-it "Permalink")How to fix it?

First, let's see where our Go app retrieves the time zone information from. Time to peek into the `time` package. You can find [here](https://golang.org/search?q=zoneSources#Global) info on all environments, but below you can check where the time zone info is in Unix systems.

_/src/time/zoneinfo_unix.go:21_

Copy

```
// Many systems use /usr/share/zoneinfo, Solaris 2 has
// /usr/share/lib/zoneinfo, IRIX 6 has /usr/lib/locale/TZ.
var zoneSources = []string{
    "/usr/share/zoneinfo/",
    "/usr/share/lib/zoneinfo/",
    "/usr/lib/locale/TZ/",
    runtime.GOROOT() + "/lib/time/zoneinfo.zip",
}
```

For this approach, there are two things to do to get the time zone info data available at runtime.

- Copy the time zone info from the build image
- Update the `ZONEINFO` environment variable with the correct path to the `zoneinfo.zip` file

Copy

```
FROM golang:1.14-alpine
WORKDIR /app
ADD . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp

FROM scratch
COPY --from=0 /app/myapp .
COPY --from=0 /usr/local/go/lib/time/zoneinfo.zip /
ENV ZONEINFO=/zoneinfo.zip
CMD ["/myapp"]
```

The output after building and running the Docker container:

Copy

```
Local time:  2020-11-07 18:51:30.03363 +0000 UTC m=+0.000164101
UTC time:  2020-11-07 18:51:30.0338934 +0000 UTC
Sarajevo time:  2020-11-07 19:51:30.0340067 +0100 CET
```

### [](https://lalatron.hashnode.dev/a-story-about-go-docker-and-time-zones#other-solutions "Permalink")Other solutions

The solution above works well, especially if you have some firewall restrictions that could prevent the solutions below to work properly.  

**Solution 1**

Basically, you can download the time zone info data with the `apk` tool in the build step and copy the data in the final step.

Copy

```
FROM golang:1.14-alpine
RUN apk --no-cache add tzdata
WORKDIR /app
ADD . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp

FROM scratch
COPY --from=0 /app/myapp .
COPY --from=0 /usr/share/zoneinfo /usr/share/zoneinfo
CMD ["/myapp"]
```

Or...

**Solution 2**

...you can download it directly in the final step, add read privileges and set the `ZONEINFO` environment variable. This solution is my least favorite, but it works.

Copy

```
FROM golang:1.14-alpine
WORKDIR /app
ADD . .
RUN CGO_ENABLED=0 GOOS=linux go build -o myapp

FROM scratch
COPY --from=0 /app/myapp .
ADD https://github.com/golang/go/raw/master/lib/time/zoneinfo.zip /zoneinfo.zip
RUN chmod +r /zoneinfo.zip
ENV ZONEINFO /zoneinfo.zip
CMD ["/myapp"]
```

### [](https://lalatron.hashnode.dev/a-story-about-go-docker-and-time-zones#conclusion "Permalink")Conclusion

As you've seen, the curse of _it works on my machine_ strikes again. It is reasonable to think that even the presented code should work in every environment. As it turns out, it doesn't. The lesson to learn here is to test your code in various runtime environments, to make sure it is working properly. Also, you learn a lot from failures. During the investigation of the presented errors, I learned more about how Go and Docker work together when a specific situation is presented. I'm sure there are a lot more different cases like these, and I can't wait to run into them.

Cheers!