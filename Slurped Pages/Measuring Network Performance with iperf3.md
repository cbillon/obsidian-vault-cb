---
link: https://cylab.be/blog/378/measuring-network-performance-with-iperf3
excerpt: Measuring the speed of your internet connection is a crucial step in identifying and troubleshooting network issues. While tools like speedtest.net allow to measure the speed of your Internet connection they can only measure the end-to-end speed. If there is a bottleneck, these tools cannot pinpoint where. In this blog post, we’ll explore a more advanced network benchmarking utility called iperf3, which allows you to measure the maximum achievable bandwidth on a network segment and identify slow segments and bottlenecks.
slurped: 2026-05-19T08:39
title: Measuring Network Performance with iperf3
tags:
  - network
  - performance
---

Measuring the speed of your internet connection is a crucial step in identifying and troubleshooting network issues. While tools like [speedtest.net](https://www.speedtest.net/) allow to measure the speed of your Internet connection they can only measure the end-to-end speed. If there is a bottleneck, these tools cannot pinpoint where. In this blog post, we’ll explore a more advanced network benchmarking utility called `iperf3`, which allows you to measure the maximum achievable bandwidth on a network segment and identify slow segments and bottlenecks.

[![speedtest.png](https://cylab.be/storage/blog/378/files/rM5gVBRwSsRsJAzH/speedtest.png)](https://cylab.be/storage/blog/378/files/rM5gVBRwSsRsJAzH/speedtest.png)

[iperf3](https://github.com/esnet/iperf) consists of a server and a client that strive to transfer data at the fastest possible rate. By running the client and server on critical points in your network, you can identify slow segments and bottlenecks, allowing you to optimize your network’s performance and improve overall efficiency.

## Installation[](https://cylab.be/blog/378/measuring-network-performance-with-iperf3#installation)

`iperf3` is available in the standard repository of most distributions, so you can install it with something like

```
sudo apt install iperf3
```

## Server[](https://cylab.be/blog/378/measuring-network-performance-with-iperf3#server)

You can start an iperf3 server with

```
iperf3 --server

# or
iperf3 -s
```

By default, the server listens on port `5201`, so make sure it’s open on your firewall. You can also specify another port with

```
iperf3 -s -p <port>
```

## Client[](https://cylab.be/blog/378/measuring-network-performance-with-iperf3#client)

The client however proposes a lot more options. In the most simple call, we only have to set `iperf3` in client mode and specify the address of the server:

```
iperf3 -c <server>
```

For example:

```
iperf3 -c 192.168.178.2
```

[![iperf3.png](https://cylab.be/storage/blog/378/files/CBHn66ozErNsgo2d/iperf3.png)](https://cylab.be/storage/blog/378/files/CBHn66ozErNsgo2d/iperf3.png)

In this example, the top terminal runs the client, and the bottom terminal shows ther server. With the default parameters:

- `iperf3` uses **TCP** to transfer data;
- data is sent **from the client to the server** (so we are testing upload speed);
- the test runs for 10 seconds, and each second the current speed is displayed;
- the **Cwnd** column shows the current size of TCP window.

As mentioned above, `iperf3` client has a lot of options. Here are the most interesting:

- `-p` or `--port` : connect to another port
- `-R` or `--reverse` : test download speed (from server to client)

- `-t` or `--time` : sets the duration of the est (in seconds)
- `-i` or `--interval` : interval for displaying periodic throughput reports (in seconds)

## UDP[](https://cylab.be/blog/378/measuring-network-performance-with-iperf3#udp)

`iperf3` can also be used to test UDP streams. However, unlike TCP, UDP has no acknowledgement mechanism. This means the sender cannot adjust the sending speed depending on the available bandwidth between the sender and receiver.

When testing with UDP, we must specify a target bitrate with `-b` or `--bitrate`. To measure the speed of the link, `iperf` will count the number of lost datagrams and measure the jitter (the the variation in datagrams delay).

[![iperf-udp.png](https://cylab.be/storage/blog/378/files/Tt0krU6dfGGuJZY9/iperf-udp.png)](https://cylab.be/storage/blog/378/files/Tt0krU6dfGGuJZY9/iperf-udp.png)

## Conclusion[](https://cylab.be/blog/378/measuring-network-performance-with-iperf3#conclusion)

By using `iperf3` to run a server and client on critical points in your network, you can gain a deeper understanding of your network’s behavior and make data-driven decisions to improve its performance. Whether you’re troubleshooting network issues, optimizing network configuration, or simply wanting to improve your network’s overall efficiency, `iperf3` is a valuable tool that can help you achieve your goals.
