---
link: https://www.elearningworld.org/moodle-behat-accessing-host-chromedriver-from-docker/
byline: ElearningWorld Admin
site: ElearningWorld.org
date: 2024-05-24T01:10
excerpt: From time to time, you may want Behat tests on your docker container to connect to chromedriver on your host
twitter: https://twitter.com/@_elearningworld
slurped: 2025-06-13T18:59
title: "Moodle Behat: Accessing host chromedriver from docker - ElearningWorld.org"
tags:
  - moodle
  - behat
---
staticcheck.dev

From time to time, you may want Behat tests on your docker container to connect to chromedriver on your host so that you can easily debug a failing Behat test.

To do this, it is important that chromedriver is started as follows:

chromedriver –whitelisted-ips= –allowed-origins=host.docker.internal

Your Moodle Behat config will then need to reference your host. This is done by setting ‘wd_host’ to ‘host.docker.internal:9515’

$CFG->behat_profiles = [  
‘default’ => [  
‘browser’ => ‘chrome’,  
‘tags’ => ‘@javascript’,  
‘wd_host’ => ‘host.docker.internal:9515’  
],  
‘headless’ => [  
‘browser’ => ‘chrome’,  
‘wd_host’ => ‘host.docker.internal:9515’,  
‘tags’ => ‘@javascript’,  
‘capabilities’ => [  
‘chrome’ => [  
‘switches’ => [  
‘–no-sandbox’,  
‘–headless’,  
‘–disable-gpu’  
]  
],  
‘extra_capabilities’ => [  
‘chromeOptions’ => [  
‘args’ => [  
‘headless’,  
‘no-gpu’  
]  
]  
]  
]  
]  
];

Running Behat on your docker container with the profile flag “headless” will allow you to see the tests running on your host. E.g:

vendor/bin/behat –config /yourmoodlepath/behat/behatrun/behat/behat.yml –profile=headless Source: https://brudinie.medium.com/feed