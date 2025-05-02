---
tags:
  - curl
  - tips
---

====== Curl ======

====== Two handy CURL tips for working with RESTful APIs ======


I was recently working on a REST API for an iPhone app. I used curl alongside automated tests to manually test as I built. curl is a great tool but if you need to specify lots of arguments, as I did, it can soon become unwieldy.

These two tips helped me out.

  - Read arguments from a file

I had many arguments to pass when testing the API including the Accept and Authorisation headers. With those alone, the argument list is already pretty big.

Luckily, the authors of curl have thought of this and have built-in an option for them to be read from a file.

Create a file for your arguments and list them inside. Mine looked something like this.

-H "Accept:application/vnd.vendor-v1+json"
-H "Authorization:Basic cZHV0Z29vdVhBR1..."
Then tell curl where to find them.

  curl <url> -K path/to/file
  
  
- Liste numérotéeReading POST data from a file

I needed to POST a lot of JSON data at various API endpoints. As with arguments, you can also tell curl to read the data from a file. This is especially useful as formatting JSON data on the command line is no fun!

To make curl read the data from a file, prefix the filename with an @ symbol.

  curl <url> –data @filename
  
These tips should hopefully make using curl easier.