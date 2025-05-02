---
tags:
  - moodle
  - debug
---

====== Debug Trace ======

===== Plugins =====

Version Moodle 4.1
installer depuis le depot de Valery Fremaux:

  * moodle-local_advancedperfs MOODLE_401_STABLE
  * moodle-local_vflibs MOODLE_39_STABLE  

===== Configuration config.php =====


  // Start Advancedperfs 
  $CFG->dirroot   = "/srv/cbi-test-41/current";
  $CFG->trace = '/srv/debug/trace';
  define('MOODLE_EARLY_INTERNAL',1);
  require_once(dirname(__FILE__) . '/local/advancedperfs/debugtools.php');
  // end Advandeperfs
  
Le fichier trace se trouve dans /srv/cbi-test-41/debug/trace

==== Debug ====

Pour obtenir une trace inserer le code

  debug_trace($var, 'comment);
  
Apres chaque modification

  sudo systemctl restart apache2&&sudo systemctl restart php7.4-fpm
  
