---
tags:
  - moodle
  - install
---
Having nothing better to do this weekend decided to give a fresh 5.2 install a go ...

Fresh install of a 5.2 via [git](https://moodle.org/mod/glossary/showentry.php?eid=10110&displayformat=dictionary "Glossary of common terms : Git") - which is working.  
Code is in a subdirectory of a FQDN.

m52 is a symlink that points to another/real 52 code directory.

Have a 5.1 configured the same way running without errors.

However, in the 5.2 running cli checks.php with -v option, reports

    ERROR | Router configuration (core_router)                       
          |     The router is not configured.                             
          |     The router is not c
The shim file is _only_ served by the router. This is a check to ensure that you have the router configured correctly. It seems that you do not.

Typically this happens because you have your web server configured to always treat files requests for a URL ending in `.php` with the PHP handler, even if the file does not exist. As a result the Router is not used.

You need to correctly configured your web server to only use the the PHP handler if the target file exists.