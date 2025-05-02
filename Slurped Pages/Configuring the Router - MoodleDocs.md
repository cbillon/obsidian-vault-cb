---
link: https://docs.moodle.org/405/en/Configuring_the_Router
excerpt: This page is intended to help systems administrators to correctly configure the Routing Engine created as a part of Moodle 4.5.
slurped: 2025-04-11T06:36
title: Configuring the Router - MoodleDocs
tags:
  - moodle
  - configuration
  - router
---

_This page is intended to help systems administrators to correctly configure the Routing Engine created as a part of Moodle 4.5._

From Moodle 4.5 onwards, Moodle includes a Routing Engine which allows requests to be dynamically served without files in specific locations.

The use of these routes can help to provide nicer URLs, which are easier to remember and more user-friendly. This is also reported to assist with SEO ranking.

Configuration of the Moodle Router will be compulsory from Moodle 5.1 onwards.

The type of Router that Moodle uses is known as a Front Controller

## What the web server needs to do

In many cases a user will request a file which is available as requested on disk, for example if the user requests https://example.com/course/view.php?id=42 then this will relate to a file at /course/view.php.

However, in other cases, a user may be accessing a URL which does not directly relate to a file on disk, for example if the user requests https://example.com/course/42/manage then this must instead be handled by the Moodle Router.

The Moodle Router is located at `/r.php`.

## How to configure the web server

Moodle supports a wide range of web servers. The guidance here is general in nature and the configuration examples may not apply exactly to your circumstances.

### Configuring Apache2

Apache2 can be configured to support the Moodle Router by specifying a [`FallbackResource`](https://httpd.apache.org/docs/trunk/mod/mod_dir.html#fallbackresource) in the `Directory` configuration.

DocumentRoot /var/www/moodle/public
<Directory /var/www/moodle/public>
    AllowOverride None
    Require all granted
    FallbackResource /r.php
</Directory>

The `FallbackResource` is relative to the site root so if your Moodle is in a sub-directory, you will need to specify the entire URL:

DocumentRoot /var/www/moodle/public
<Directory /var/www/moodle/public/moodle>
    AllowOverride None
    Require all granted
    FallbackResource /moodle/r.php
</Directory>

### Configuring Nginx

The nginx server supports the use of a [`try_files`](https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files) directive, which checks for existence of files in the specified order, using the first found file for processing.

location / {
    try_files $uri /r.php
}

If your Moodle site is in a sub-directory, you will need to use a more specific location.

location /moodle/ {
  try_files $uri $uri/ /moodle/r.php;
}

If you host multiple Moodles then you may wish to use a case-sensitive regular expression match for the location:

location ~ ^/(?<sitepath>[^/]*)/ {
  try_files $uri $uri/ /$sitepath/r.php;
}

### IIS

Note: The following instructions are untested. If you have knoweldge of IIS and can provide more accurate or up-to-date instructions, please do so.

Using the [URL Rewriter](https://www.iis.net/downloads/microsoft/url-rewrite) module for IIS, you can create a Rewrite rule in your `web.config` file.

<rewrite>
    <rules>
        <rule name="Moodle" stopProcessing="true">
            <match url="^(.*)$" ignoreCase="false" />
            <conditions logicalGrouping="MatchAll">
                <add input="{REQUEST_FILENAME}" matchType="IsFile" ignoreCase="false" negate="true" />
            </conditions>
            <action type="Rewrite" url="r.php" appendQueryString="true" />
        </rule>
    </rules>
</rewrite>

## Configuring Moodle

After successfully configuring the Router, you should inform Moodle that it is correctly configured by setting the following in your `config.php`

$CFG->routerconfigured = true;