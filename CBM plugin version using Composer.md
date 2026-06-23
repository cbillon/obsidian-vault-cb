---
tags:
  - cbm
  - composer
---
I have a plugin developed in PHP;  
the plugin version to use depends on the project version.  
The plugin is included in a project installed with Composer;  
which composer.json file should I use?

Thought for 7 seconds
Short answer

Use the project’s composer.json.
The plugin itself must have its own composer.json (the one you publish to Packagist or keep in its own repo), but the version you actually pull in is chosen by the version constraint you put in the project’s composer.json.

Why the project’s composer.json is the one you edit
WHEN YOU RUN
WHAT COMPOSER READS
WHAT IT DOES
composer install / composer update in your main project
composer.json in the project root
Installs the packages you listed, respecting their version constraints
The package you are installing (e.g. a plugin)
The composer.json that lives in that package’s repository
Provides the package name, autoloading, dependencies, etc. – it is read automatically, you never touch it from the project side


So the plugin’s composer.json is necessary (it tells Composer what the package is called, what PHP version it requires, how to autoload its classes, etc.), but you don’t edit it to decide which plugin version to pull in. That decision lives in the project’s composer.json.

How to tie plugin version to project version
1. Tag the plugin releases
Give your plugin release tags that match the project version it is compatible with:

PROJECT
COMPATIBLE PLUGIN TAG
1.x
1.0.0, 1.1.0, …
2.x
2.0.0, 2.1.0, …


When you publish the plugin to Packagist (or keep a private repo) each tag becomes a distinct version that Composer can download.

2. Add the plugin as a dependency in the project
Project 1.x (composer.json):

json

Collapse
Save
Copy
1
2
3
4
5
⌄
⌄
{
  "require": {
    "vendor/plugin": "^1.0"
  }
}
Project 2.x (composer.json):

json

Collapse
Save
Copy
1
2
3
4
5
⌄
⌄
{
  "require": {
    "vendor/plugin": "^2.0"
  }
}
The caret (^) means “any compatible 1.x (or 2.x) release”.
If you need stricter control you can use >=1.0,<2.0 or exact versions.

3. Run Composer
bash

Collapse
Save
Copy
1
composer update
Composer will resolve the constraints, pick the correct tag, and install the corresponding plugin code.

Optional tricks
NEED
SOLUTION
The plugin is optional (only for dev)
Put it in require-dev instead of require.
You want to override a dependency that the plugin pulls
Use replace or provide in the project’s composer.json.
The plugin contains two sub‑packages (e.g. core + UI)
Keep a single composer.json in the plugin repo; use the autoload section to expose both namespaces.
You want to ship the plugin inside the project (no external repo)
Place the plugin in a subdirectory and add "path": "./vendor/plugin" in the project’s composer.json under "repositories".


Bottom line
Keep one composer.json in the plugin repo – that’s what Composer uses when pulling the plugin in.
Edit the project’s composer.json – that’s where you decide which version of the plugin to install, using the appropriate version constraint that matches the project’s own version.
Once you follow that pattern, Composer will automatically fetch the right plugin release for every project version.








Follow up