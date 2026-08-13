---
link: https://peteris.rocks/blog/modifying-xml-json-ini-configuration-files-without-sed/
byline: Pēteris Ņikiforovs
site: peteris.rocks
excerpt: How to modify configuration files in XML, JSON and INI formats without using sed
slurped: 2026-07-05T10:04
title: Modifying XML, JSON, INI configuration files without sed
tags:
  - jq
  - json
---

Let's say you have a configuration file in XML format called `zeppelin-site.xml` that you want to modify.

```
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="https://peteris.rocks/blog/modifying-xml-json-ini-configuration-files-without-sed/configuration.xsl"?>
<configuration>
  <property>
    <name>zeppelin.anonymous.allowed</name>
    <value>true</value>
    <description>Anonymous user allowed by default</description>
  </property>
  <!-- ... --->
</configuration>
```

In this case, let's pretend you want to change `true` to `false`.

You could try doing that with a regular expression and `sed` but that's not trivial because you need to find `zeppelin.anonymous.allowed` first and replace the value on the next line.

You can use [xmlstarlet](https://en.wikipedia.org/wiki/XMLStarlet) instead. It's installable with `sudo apt-get install xmlstarlet`.

You can use XPath to find the elements that you are interested in:

```
$ xmlstarlet sel -t -v "/configuration/property[name='zeppelin.anonymous.allowed']/value" zeppelin-site.xml
true
```

This expression looks for `<property>` elements under `<configuration>` that have a child element `<name>` with the value `zeppelin.download.prebuilt`. When those elements are found, it will select the value of the `<value>` child element. Right now it's `true` which we want to change.

To change it, use the `ed` action instead of `sel`. `-u` is the search pattern and `-v` is the replacement value.

```
$ xmlstarlet ed -u "/configuration/property[name='zeppelin.anonymous.allowed']/value" -v "false" zeppelin-site.xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="https://peteris.rocks/blog/modifying-xml-json-ini-configuration-files-without-sed/configuration.xsl"?>
<configuration>
  <!-- common params -->
  <property>
    <name>zeppelin.anonymous.allowed</name>
    <value>false</value>
    <description>Anonymous user allowed by default</description>
  </property>
...
```

Use `--inplace` to also save the changes to the file.

```
$ xmlstarlet ed --inplace -u "/configuration/property[name='zeppelin.anonymous.allowed']/value" -v "false" zeppelin-site.xml
```

[XMLStarlet Wikipedia page](https://en.wikipedia.org/wiki/XMLStarlet) has good usage examples.

## 

JSON

Let's imagine that you now have a configuration file in JSON.

```
{
  "name": "something",
  "version": "0.0.0",
  "array": [1,2],
  "params": {
    "name1": "value1",
    "name2": "value2"
  }
}
```

You want to add a new field `dependencies` that is an empty array `[]`.

You can use [jq](https://stedolan.github.io/jq/) which is like `sed` but for JSON. It's installable with `sudo apt-get install jq`.

`jq` will read and parse your JSON file. If it happens to be minified or not properly formatted, you can use the `.` operator to select everything and display it nicely with syntax highlighting.

```
$ jq "." config.json
{
  "params": {
    "name2": "value2",
    "name1": "value1"
  },
  "array": [
    1,
    2
  ],
  "version": "0.0.0",
  "name": "something"
}
```

Here is how to select the `params` object:

```
$ jq ".params" config.json
{
  "name2": "value2",
  "name1": "value1"
}
```

How to add a new field?

```
$ jq ".params.dependencies = []" config.json
{
  "params": {
    "dependencies": [],
    "name2": "value2",
    "name1": "value1"
  },
  "array": [
    1,
    2
  ],
  "version": "0.0.0",
  "name": "something"
}
```

How to change a value?

```
$ jq '.name = "changed"' config.json
{
  "params": {
    "name2": "value2",
    "name1": "value1"
  },
  "array": [
    1,
    2
  ],
  "version": "0.0.0",
  "name": "changed"
}
```

How to change a value based on other values?

```
$ jq '.name = .params.name1 + " " + .params.name2' config.json
{
  "params": {
    "name2": "value2",
    "name1": "value1"
  },
  "array": [
    1,
    2
  ],
  "version": "0.0.0",
  "name": "value1 value2"
}
```

How to add an element to an array?

```
$ jq ".array |= .+ [3]" config.json
{
  "params": {
    "name2": "value2",
    "name1": "value1"
  },
  "array": [
    1,
    2,
    3
  ],
  "version": "0.0.0",
  "name": "something"
}
```

`|=` will reassign the value and `.+ [...]` adds a new item.

How to delete a key?

```
$ jq 'del(.array)' config.json
{
  "params": {
    "name2": "value2",
    "name1": "value1"
  },
  "version": "0.0.0",
  "name": "something"
}
```

[jq website](https://stedolan.github.io/jq/) has more documentation.

## 

INI

Let's suppose you want to work with this INI configuration file.

```
[section]
host = "localhost"
port = 8888
```

We can use `augtool` which is installable with `sudo apt-get install augeas-tools -y`.

Its syntax is a bit weird.

List all parsed options.

```
$ augtool -At "Puppet.lns incl `pwd`/config.ini" print /files/`pwd`/config.ini
/files/home/ubuntu/config.ini
/files/home/ubuntu/config.ini/section
/files/home/ubuntu/config.ini/section/host = "localhost"
/files/home/ubuntu/config.ini/section/port = "8888"
```

Let's change `port` from `8888` to `9000`.

```
$ augtool -At "Puppet.lns incl `pwd`/config.ini" set /files/`pwd`/config.ini/section/port 9000
Saved 1 file(s)
$ cat config.ini
[section]
host = "localhost"
port = 9000
```

Why is there `Puppet.lns`? Appearantly there is no one standard INI file format and so there are several variations. Other options you can choose from are `MySQL.lns` (allows slashes in section names, supports `!include`) and `PHP.lns` (allows settings outside sections).

You can also use `augtool` to edit `/etc/hosts` and Java properties files!

```
$ augtool print /files/etc/hosts
/files/etc/hosts
/files/etc/hosts/1
/files/etc/hosts/1/ipaddr = "127.0.0.1"
/files/etc/hosts/1/canonical = "localhost"
```