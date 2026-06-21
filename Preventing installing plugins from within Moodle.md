---
tags:
  - moodle
  - install
  - plugin
---
If required, installing and updating from within Moodle can be prevented by copying the following lines of code from config-dist.php and pasting them in config.php.

// Use the following flag to completely disable the installation of plugins
// (new plugins, available updates and missing dependencies) and related
// features (such as cancelling the plugin installation or upgrade) via the
// server administration web interface.

$CFG->disableupdateautodeploy = true; 