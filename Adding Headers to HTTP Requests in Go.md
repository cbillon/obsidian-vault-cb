---
tags:
  - go
link: https://nelson.cloud/adding-headers-to-http-requests-in-go/
---
Adding Headers to HTTP Requests in Go
2026-01-30 · Updated 2026-03-28 · 2 min · 267 words
Go 
Go has several ways of sending requests, including some convenient methods such as:

http.Get()
http.Head()
http.Post()
http.PostForm()
However, these don’t let you add headers to requests! If you need customization of the HTTP method or headers, you need to use http.NewRequest().

There are three parts to this:

Create a request using http.NewRequest() where you specify the HTTP method and URL
Add headers to the request with Header.Set()
Send the request using http.Client{}
Here’s a full example:

package main

import (
	"log"
	"net/http"
)

func main() {
	// create the client that will send the request later
	client := &http.Client{}

	// create a request with NewRequest, specifying HTTP method and URL
	req, err := http.NewRequest("GET", "https://example.com/", nil)
	if err != nil {
		log.Fatal(err)
	}

	// add headers to the request
	req.Header.Set("User-Agent", "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:147.0) Gecko/20100101 Firefox/147.0")
	req.Header.Set("Accept-Language", "en-US,en;q=0.9")
	req.Header.Set("Accept-Encoding", "gzip, deflate")
	req.Header.Set("Sec-GPC", "1")
	req.Header.Set("Connection", "keep-alive")

	// use the client to actually send the request
	resp, err := client.Do(req)
	if err != nil {
		log.Fatal(err)
	}
	defer resp.Body.Close()
}
Note
req.Header.Set() can be used to set a header but will overwrite any existing value for that header.

req.Header.Add() can be used to add a value to a header and will append to any existing value for that header.

For the purposes of this blog post we only need to worry about setting the header once, hence the usage of req.Header.Set().

Similar to my “Using time.Sleep() in Go” post, I wrote this up because the Go docs are too dense and I just needed one full example to understand it and get going.

References
https://pkg.go.dev/net/http
« Prev
I Hate Workday
Next »
Using time.Sleep() in Go
Random Post
Return Home

Related Posts
Using time.Sleep() in Go
Examples of using time.Sleep() in Go because the official documentation is lacking.

Validate HTTP Status Codes in Go Using Built-in Constants
Use Go net/http constants like StatusOK and StatusNotFound for more readable code.

Iterate Through Strings in Go with a for-range Loop
You can use for-range loops to iterate through strings in Go without splitting because Go handles strings as byte slices.