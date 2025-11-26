---
link: https://evertras.com/blog/2022/10/26/suffix-variables-with-units/
excerpt: |-
  Quiz time! You’re doing code reviews for a few different projects. Which of
  these is incorrect, and which would you accept?
slurped: 2025-11-26T11:36
title: Evertras - Suffix Variables with Units
tags:
  - variable
  - type
  - tips
---

## The problem

Quiz time! You’re doing code reviews for a few different projects. Which of these is incorrect, and which would you accept?

```
// C++
#include <ctime>
#include <iostream>

int main()
{
  std::time_t result = std::time(nullptr);
  std::cout << "A minute ago was " << result - 60 << std::endl;
}
```

```
// Javascript
const now = Date.now();

console.log("A minute ago:", now - 60);
```

```
# Python
import time

idk = time.time()
inThePast = idk - 60;

print("One minute ago:", inThePast);
```

```
<?php
echo 'A minute ago: '. (time() - 60);
?>
```

You may have spotted it if you work in the language often enough. Unless you’re a heavy polyglot, you probably had to check at least one. I certainly did, and I wrote it! If you haven’t checked yet, don’t worry and just read on to the next section.

## Fixed

Now let’s add a unit suffix to each of these functions and variables. Pretend the new names are how the standard libraries for each of these languages are actually defined. Now can you see which one is wrong?

```
// C++
#include <ctime>
#include <iostream>

int main()
{
  std::time_t result = std::timeSeconds(nullptr);
  std::cout << "A minute ago was " << result - 60 << std::endl;
}
```

```
# Python
import time

seconds = time.timeSeconds()
inThePastSeconds = seconds - 60;

print("One minute ago:", inThePastSeconds);
```

```
<?php
echo 'A minute ago: '. (timeSeconds() - 60);
?>
```

```
// Javascript
const nowMs = Date.nowMilliseconds();

console.log("A minute ago:", nowMs - 60);
```

Javascript! You rascal.

You don’t need intellisense. You just immediately see that this function returns milliseconds, so the variable is also named with milliseconds. Then when the variable is used, it’s crystal clear that this is a mismatch between expected values - we meant `60000 ms` as our minute, not `60 ms`.

## It’s not just time

It is a common misconception that a convention is “obvious”. Time units are a common culprit, but it could be anything. Weight, force, distance, size. Here’s a snippet of a [Nomad](https://nomadproject.io/) configuration that describes how many resources a job is given to run:

```
task "server" {
  resources {
    cpu    = 2
    memory = 1
  }

  /* ... */
}
```

_What does that even mean?_ I love Nomad, but this is not something that is friendly to someone reading/maintaining the code. In this case, you might guess that it means the server task gets 2 CPU cores and… a gig of memory, maybe? But no, that’s actually being allocated 0.2% of a CPU core and 1 MB of memory. You can only know this by reading the docs and getting used to it as a convention for that configuration section.

How about…

```
task "server" {
  resources {
    cpuMillicores = 2
    memoryMB      = 1
  }

  /* ... */
}
```

Now it’s clearly just wrong, and we can fix it.

_That’s a lot of extra typing,_ you say. _Very inefficient. We can have a convention that size is in megabytes, that’s a reasonable level of granularity for this kind of thing._

Ok cool, so let’s try to Terraform up an Elastic Block Storage of 40GB so we can store some data! Terraform is made by the same company as Nomad, so it should use the same convention, right?

```
resource "aws_ebs_volume" "example" {
  availability_zone = "us-west-2a"
  size              = 40000

  tags = {
    Name = "HelloWorld"
  }
}
```

…except we just created a 40 TB block. Or tried to, if AWS even let us. Turns out that this time a size value is in GB, and [the only way to know that is to look at the docs](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ebs_volume) .

Full disclosure, I love Hashicorp’s stuff! But this is not something I love about them, so I’m going to be mean again for a moment.

```
resource "aws_redshift_usage_limit" "example" {
  cluster_identifier = aws_redshift_cluster.example.id
  feature_type       = "concurrency-scaling"
  limit_type         = "time"
  amount             = 60
}
```

Amount is… what? Seconds? Milliseconds? [Nope, minutes](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/redshift_usage_limit) . Unless you do this:

```
resource "aws_redshift_usage_limit" "example" {
  cluster_identifier = aws_redshift_cluster.example.id
  feature_type       = "concurrency-scaling"
  limit_type         = "data-scanned"
  amount             = 60
}
```

Now what is it? Terabytes, because why not. We’ve now seen three different units of size in three different configuration sections of tools built by the same company, two of which were in the same tool.

## The cure

As you might have guessed from the title, the cure is simple: suffix your variables! This includes both in code and in configuration. Embrace verbosity. Even if you’re working in your own project, with no one else around, you’ll thank yourself later if you stick with this habit because it’s one less useless thing that you need to keep in your brain. Instead, you get to focus on getting stuff done.

```
// Which would you rather read?

const distance = 3;
const distanceMeters = 3;

const size = 50;
const sizeMb = 50;

const duration = 3;
const durationMinutes = 3;

// Just because the function doesn't follow this rule doesn't mean we can't!
const timeMs = Date.now();
```

Many typed languages make this foolproof out of the box with type checking.

```
// Golang

// There is no suffix to add here, because 'now' is a time.Time type
now := time.Now()

// Type safety and explicitness!
aMinuteAgo := now.Add(-time.Minute)
```

And you can add your own types to help.

```
// Create a type that's really just a number
type Meters float64
type Feet float64

// Now we can make it clear what kind of distance we're dealing with!
distance := Meters(3)

// This errors, which is exactly what we want
distance = distance + Feet(3)

// This isn't foolproof, though... 3 what?  This may be handled better in your
// typed language of choice, but Go "helps" us here.
distance = distance + 3

// Is this correct?
fmt.Println(distance, "feet away")
```

You can do better.

```
type Meters float64

// Overkill?  I'd argue not.  When you don't have access to intellisense,
// it can be easy to forget context.
distanceMeters := Meters(3)

// Still wish we had type safety with the 3, but at least this is much clearer
distanceMeters = distanceMeters + 3

// Now this is obviously a bug that can even be spotted in a PR review!
fmt.Println(distanceMeters, "feet away")
```

So please… suffix your variables. Anyone reading your code will thank you, you’ll thank yourself in a week when you didn’t have to look up whether that time function was nanoseconds or microseconds for the 50th time, and the taxpayers will thank you [when you don’t lose $327 million by unintentionally deorbiting a lander into Mars](https://en.wikipedia.org/wiki/Mars_Climate_Orbiter) .