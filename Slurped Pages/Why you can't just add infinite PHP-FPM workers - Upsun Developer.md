---
link: https://developer.upsun.com/posts/insights/why-you-cant-just-add-infinite-php-fpm-workers
site: Upsun Developer
excerpt: Workers need memory. Learn how PHP-FPM sizing hints work, why you can't
  just add infinite workers, and how to tune your worker count based on actual
  memory usage.
slurped: 2026-08-25T09:22
title: Why you can't just add infinite PHP-FPM workers - Upsun Developer
---

After reading about [PHP-FPM worker exhaustion and 502 errors](https://developer.upsun.com/posts/how-it-works/when-php-fpm-runs-out-of-workers-a-502-error-field-guide), the obvious question is: why not add more workers? If workers are like highway lanes, why don’t we build highways with 20 lanes? Because lanes need space. They have to sit somewhere. For PHP-FPM workers, that space is measured in memory. Here’s the math: if each request needs 50MB of memory to process, and your container has 512MB of memory total, you can theoretically fit 10 workers (10 × 50MB = 500MB, with 12MB left over). That’s an oversimplification - nginx needs memory, SSH needs memory, the container itself needs overhead. But that’s the basic constraint: workers consume memory, and you only have so much memory available.

## What Upsun does automatically

On Upsun, we calculate your worker count automatically using this formula:

For example, with a 256MB container using default settings (45MB request memory, 70MB reserved memory):

With a 512MB container:

You can see the full details in the [PHP-FPM documentation on Upsun docs](https://docs.upsun.com/languages/php/fpm.html). The defaults (45MB request memory, 70MB reserved memory) work fine for most Drupal and WordPress sites. But they’re not always right for your application.

## When the defaults don’t fit

If you’re running Magento, 45MB per request might not be enough - you’re going to run out of container memory before you run out of workers. Your metrics will show 100% memory utilization and Out Of Memory kills. If you’re running a lightweight API, you might only need 4MB per request. The system is assuming 45MB, so you’re underutilizing your memory and running fewer workers than you could support. This is when you want to adjust sizing hints.

### Understanding sizing hints

Sizing hints are exactly that: hints. They’re not limits or restrictions. They tell Upsun how to calculate the number of workers to allocate. You configure them in your `.upsun/config.yaml`:

Here’s the counterintuitive part: **you lower request memory to increase worker count**, because it’s a division. If you set `request_memory: 35` instead of `45`, you get more workers for the same container size. Some people get this backwards and think “more is better.” It’s not. Lower request memory tells the system “my requests are lighter, I can support more workers.”

### Finding your actual memory usage

You can check how much memory your requests actually use by analyzing your PHP access logs:

This shows memory usage for each request. You’ll see output like:

Those numbers are in kilobytes. Most requests hover around the same range. Take a representative average and use that as your `request_memory` value (converted to MB). This isn’t an exact science. Not all requests use the same memory. Some take 40MB, some take 20MB, some take 100MB. You’re not utilizing all workers at the same time. The goal is to find a reasonable average that keeps your memory utilization under 100% while maximizing worker availability.

### Tuning in small increments

If you decide to adjust sizing hints, do it gradually:

1. Start with your current value (default is 45MB)
2. Lower it by 10MB (to 35MB)
3. Check your metrics - are you still under 100% memory utilization?
4. Wait a day and verify the system is stable
5. If you need more workers, lower it another 10MB
6. Repeat until you find the right balance

If you go too low, you’ll see high memory utilization in your metrics dashboard and potentially Out Of Memory kills in your logs. Back it off by 10MB and stabilize there. Also watch your CPU usage. More workers mean more CPU load. Upsun’s container profiles balance CPU and memory, so this usually isn’t an issue if you’re using the right profile. But if you’re manually adjusting container resources, keep an eye on it.

### CPU limits and worker caps

To ensure that Upsun doesn’t add more workers than the CPU can handle, a CPU limit applies as soon as the number of set workers equals or exceeds 25. This limit is calculated as: **number of vCPU cores × 5**. For example, with 8 vCPUs, you can have a maximum of 40 workers (8 × 5 = 40) - even if you have enough memory to support more. Why? Because the CPU has to switch between all these processes. If you give it too many workers, context switching overhead becomes a problem. When 8 vCPUs are juggling 40 different tasks (5 per vCPU), they’re already working hard. Going beyond that makes things slower, not faster.

## Sizing hints vs memory limit

Let’s clarify the difference between sizing hints and PHP memory limit, since they’re often confused: **Sizing hints** (`request_memory`, `reserved_memory`):

- Used to calculate worker count
- Not enforced as limits
- Hints for capacity planning
- Lower values = more workers allocated

**PHP memory limit**:

- Enforced per-request by PHP itself
- Kills requests that exceed the limit
- Protects the container from runaway processes
- Logs violations for debugging

If you set `request_memory: 45` but one request uses 50MB, nothing bad happens. The sizing hints are averages, not hard limits. If you set `memory_limit: 128M` and a request tries to use 200MB, that request gets killed. See our article on [why you shouldn’t set your PHP memory limit to 60GB](https://developer.upsun.com/posts/insights/why-setting-your-php-memory-limit-to-60gb-wont-help) for more details.

## Finding the right balance

When you understand what these settings actually do, you can tune them appropriately for your application. The goal is to find the sweet spot where:

- Your workers have enough memory to handle typical requests
- You have enough workers to handle concurrent traffic
- Your memory utilization stays under 100%
- Your CPU isn’t overwhelmed by context switching

Start with the defaults, measure your actual usage, and adjust gradually. Don’t try to optimize prematurely - let real-world traffic patterns guide your tuning.

## Ready to get started?

If you’re dealing with 502 errors and worker exhaustion issues, check out our companion article: [When PHP-FPM Runs Out of Workers: A 502 Error Field Guide](https://developer.upsun.com/posts/how-it-works/when-php-fpm-runs-out-of-workers-a-502-error-field-guide). Want to see how Upsun handles PHP-FPM configuration automatically? [Sign up for a free trial](https://upsun.com/) and deploy your first project.