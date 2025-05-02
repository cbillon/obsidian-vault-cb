---
tags:
  - moodle
  - plugin
  - ci
  - behat
---

[ici](https://moodlehq.github.io/moodle-plugin-ci/#github-actions)

[Plugin Moodle ci](https://moodlehq.github.io/moodle-plugin-ci/#github-actions)
[Github Action quick start](https://moodlehq.github.io/moodle-plugin-ci/#github-actions)

Marcus Green
#### [General developer forum](https://moodle.org/mod/forum/view.php?id=55) -> [Coding best practices for creating open source Moodle plugins](https://moodle.org/mod/forum/discuss.php?d=462107) -> [Re: Coding best practices for creating open source Moodle plugins](https://moodle.org/mod/forum/discuss.php?d=462107#p1855297)

par [Marcus Green](https://moodle.org/user/view.php?id=2246&course=5), mardi 17 septembre 2024, 04:35

![Avatar Core developers](https://moodle.org/pluginfile.php/53/group/icon/172/f1?rev=1446084 "Avatar Core developers") ![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers") ![Avatar Plugin developers](https://moodle.org/pluginfile.php/53/group/icon/306/f1?rev=1446090 "Avatar Plugin developers") ![Avatar Testers](https://moodle.org/pluginfile.php/53/group/icon/200/f1?rev=1446102 "Avatar Testers")

I am sure others will come in with advice but one of my top suggestions is that you run the moodle-ci (continuous integration) plugin, which typically runs against Github actions.  
[https://moodledev.io/general/development/tools/gha](https://moodledev.io/general/development/tools/gha)  
  
This checks a whole bunch of stuff. It will run behat and phpunit tests, will lint php, [javascript](https://moodle.org/mod/glossary/showentry.php?eid=25&displayformat=dictionary "Glossary of common terms : Javascript"), mustache etc etc and some moodle specific stuff such as upgrade scripts. This is an example I was using yesterday.  
[https://github.com/marcusgreen/moodle-qbank_bulktags/blob/main/.github/workflows/moodle-ci.yml](https://github.com/marcusgreen/moodle-qbank_bulktags/blob/main/.github/workflows/moodle-ci.yml)