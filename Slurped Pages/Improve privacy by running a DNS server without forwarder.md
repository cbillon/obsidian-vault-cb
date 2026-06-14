---
link: https://louwrentius.com/improve-privacy-by-running-a-dns-server-without-forwarder.html
excerpt: "Disclaimer: no AI was used to write this article."
slurped: 2026-06-04T08:57
title: Improve privacy by running a DNS server without forwarder
tags:
  - dns
  - pi-hole
---

If you run a DNS server at home or for an organisation, it is likely that you configured a public DNS server as a forwarder[1](https://louwrentius.com/improve-privacy-by-running-a-dns-server-without-forwarder.html#fn:public).

_This is not required!_

Imagine we want to resolve the address 'wikipedia.org'.

If no forwarder is configured, a DNS server will behave like this:

1. Query the [root server](https://en.wikipedia.org/wiki/Root_name_server)[3](https://louwrentius.com/improve-privacy-by-running-a-dns-server-without-forwarder.html#fn:cluster) for an IP address of the .org [top-level domain](https://en.wikipedia.org/wiki/Top-level_domain) authoritative DNS server
2. Query the .org DNS server for an IP address of the wikipedia.org [authoritative](https://superuser.com/questions/370105/what-does-authoritative-dns-server-mean) DNS server
3. Query the wikipedia.org DNS server for the IP address of 'wikipedia.org'

This is what the forwarder does when you query a domain that is not cached by said forwarder.

### A forwarder responds faster?!

Yes, that's probably true, but I think it doesn't matter.

As a DNS forwarder caches DNS lookups, it will respond directly with the appropriate IP address. Only a single DNS query is required.

Without a forwarder, a domain lookup that is not cached, will require at least two or three DNS queries, the first time this domain is resolved. After this first set of queries, the domain is cached for the duration of the [TTL](https://en.wikipedia.org/wiki/Time_to_live#DNS_records).

I've performed a few manual lookups with 'dig' for domains that aren't cached, and most responses are <50ms. Depending on where you live and the quality of the network connection (in terms of latency) it may or may not be noticable, that's up to you to explore.

### So what is the privacy angle?

You are exposing privacy-sensitive information to Google or other large companies.

If a DNS forwarder is configured, the forwarder will 'see' all your DNS queries and will be able to build a nice profile about you as a person or organisation.

It's up to you if you care about this 'risk'.

## Running Pi-hole and BIND9 together

### Pi-hole only supports forwarders?

Because Pi-hole requires a DNS forwarder to be configured, I'm also running BIND as the actual DNS server that performs the outbound DNS queries.

Both Pi-hole and BIND want to listen on UDP port 53, which won't work[2](https://louwrentius.com/improve-privacy-by-running-a-dns-server-without-forwarder.html#fn:alternative). In my case, I've configured BIND9 to listen on UDP port 5353 and configured Pi-hole to use '_BIND listen ip address_#5353' as a forwarder.

This setup means that I benefit from the ad blocking by Pi-hole and won't send all my DNS requests to Google.

### Help, my local DNS authoritative '.internal' zone isn't working anymore!

I'm running a local DNS domain 'xyz.internal' for servers and services in my network. I've also configured my DHCP server to tell clients the search domain is 'xyz.internal', so when I issue `ssh server01` this will automatically resolve to `server01.xyz.internal`.

So to summarise, my BIND DNS server is both the DNS server that respons to client DNS lookups via Pi-hole for public domains, and it's also authoritative for the 'xyz.internal' domain.

However, there's a problem with this setup.

Pi-hole does not forward queries for the '.internal' domain because it considers upstream forwarding DNS servers as potentially hostile. Forwarding queries for '.internal' domains would expose potentially sensitive information.

This behaviour can be circumvented with the 'Conditional forwarding' option within Pi-hole. This allows DNS clients contacting the Pi-hole server to still query '.internal' domains. This is an example configuration line:

true,10.10.10.0/24,10.10.10.1#5353,xyz.internal

In this example, Pi-hole will forward DNS queries for the xyz.internal domain towards 10.10.10.1#5353 and everything will work as expected.

## Q & A

### How does the DNS server know how to contact the .org root server?

The DNS server is accompanied by a 'root zone file' containing the IP addresses for the root servers. As these IP addresses are not static, they need to be updated periodically. In Debian Linux (for example), this file is automatically updated through APT with the 'dns-root-data' package.