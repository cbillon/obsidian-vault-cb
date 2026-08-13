
Alternative Moodle Market Place


[discusion ](https://app.element.io/#/room/#openpluginsdirectory:matrix.org)
1

Alternatives to Moodle Marketplace

6

1. David Pesce
    
    Forkable, always. that's what keeps it honest. that said, it would be better to build on top of it
    
2. what we're proposing to HQ is that marketplace sits on top of it
    
3. this way the community owns what it created and hq can monetize where developers have authorized
    
4. the tool_camp plugin is a way to add trusted repos and prioritize where you get your plugins from
    
5. that's just prior to moodle 5.2 where composer will take over and utilize the camp-registry as a packigist of sorts
    
6. Adam Jenkins
    
    Looks like a solid proposal. Conceptually, it kind of reminds me of F-droid a bit.
    
7. David Pesce
    
    yeah, partially inspired by it. check out the RFC in the docs repo
    
8. Adam Jenkins
    
    Will do, sorry to be late to the party, will definitely be reading up on it more.
    
9. David Pesce
    
    this also takes into account a lot of the moodle partner concerns. they can have their own private repos that can be added to tool_camp and authorized with tokens. meaning their customers can pay them directly and are authorized to use their plugins. moodle stil gets their partner fee
    
10. > [
    > 
    > Adam Jenkins
    > 
    > Will do, sorry to be late to the party, will definitely be reading up on it more.
    > 
    > 
    > 
    > ](https://matrix.to/#/!cvBzQDMQGakZxaxBLU:matrix.org/$q3juBu_bHdnIHQ5ReNpe8x4jfIg0oO3-i3eFn3FPueI?via=matrix.org)
    
    No worries. The more input we get, the better!
    
11. Ask questions, poke holes!
    
12. Jordan Tomkinson
    
    I haven't fully read the docs, so forgive me if this is already answered, but looking at your current index, im only seeing plugins from github/gitlab/bitbucket. and the maintainers field needs a key of one of the same. how are you handling custom/self hosted git repos, where a maintainer tag cant be matched to one of the big clouds? comparing the current plugins API to the index, it seems those plugins didnt make the cut into tier 0
    
13. and also all releases lists are empty
    
14. David Pesce
    
    > [
    > 
    > Jordan Tomkinson
    > 
    > I haven't fully read the docs, so forgive me if this is already answered, but looking at your current index, im only seeing plugins from github/gitlab/bitbucket. and the maintainers field needs a key of one of the same. how are you handling custom/self hosted git repos, where a maintainer tag cant be matched to one of the big clouds? comparing the current plugins API to the index, it seems those plugins didnt make the cut into tier 0
    > 
    > 
    > 
    > ](https://matrix.to/#/!cvBzQDMQGakZxaxBLU:matrix.org/$saR8XDTM6uRorsN2dQ62jefPHhVpXURZvuxWVSQEnFs?via=matrix.org)
    
    yeah, this was a deliberate decision with speed. if you read the authors.md, there is a way to submit other version control systems as well
    
15. well, your plugin if it's on a diff vcs. i don't think we'll be scanning additional systems
    
16. At almost 6k plugins, we shouldn't need to scan any more and as the community/developers rally around it, it should expand as plugins are created
    
17. Jordan Tomkinson
    
    so you didnt use the plugins directory API to seed it?
    
18. David Pesce
    
    nope, that would be illegal
    
19. Jordan Tomkinson
    
    how so? its a public api
    
20. David Pesce
    
    and honestly, it's ancient and limited at that
    
21. robots.txt
    
22. Jordan Tomkinson
    
    there is no robots.txt on download.moodle.org that i can see
    
23. David Pesce
    
    maybe it was removed? Pretty sure we looked at first and abandoned it. I'll see if i can deploy the browsing interface tonight/tomorrow
    
24. Jordan Tomkinson
    
    6k plugins is impressive. official holds 2849
    
25. David Pesce
    
    [
    
    ![image.png](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/HWErFRgDQLodWhOoCJNHUuvv?allow_redirect=true)
    
    
    
    ](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/HWErFRgDQLodWhOoCJNHUuvv?allow_redirect=true)
    
26. Jordan Tomkinson
    
    well. its a good thing you arent using the API. they are bulk removing plugins from the results, presumably as they migrate the record to marketplace. the plugin count is dropping every few minutes by hundreds of plugins.
    
27. Adam Jenkins
    
    2412 now
    
28. Robots.txt at the base domain is this:
    
    User-agent: AhrefsBot  
    Crawl-Delay: 15
    
    User-agent: *  
    Disallow: /search/index.php  
    Disallow: /mod/forum/search.php  
    Disallow: /plugins/_/stats  
    Disallow: /plugins/_/translations  
    Disallow: /plugins/_/_  
    Content-Signal: ai-train=no, search=yes, ai-input=no  
    Crawl-Delay: 10
    
29. Jordan Tomkinson
    
    robots.txt is scoped per host. download.moodle.org is where the plugins API lives and has no robots.txt
    
30. Adam Jenkins
    
    > [
    > 
    > Jordan Tomkinson
    > 
    > robots.txt is scoped per host. download.moodle.org is where the plugins API lives and has no robots.txt
    > 
    > 
    > 
    > ](https://matrix.to/#/!cvBzQDMQGakZxaxBLU:matrix.org/$1uQ5dRMtUX1422z4JGxmczpwMnO_Si5CB5Cn_FUBujE?via=matrix.org)
    
    Good point
    
31. Jordan Tomkinson
    
    and technically, a surgical curl of an API endpoint is not a robot/crawler
    
32. millions (probably) of moodle sites hit that endpoint every day, a few curls go unnoticed and are also unblockable
    
33. if they blocked the tls fingerprint of libcurl, moodles php fetch also goes down with it :)
    
34. David Herney
    
    I submitted a pull request to claim one of my plugins to better understand the process. Is that okay, or should I wait for further discussion and conclusions?
    
35. David Pesce
    
    Excellent, that’s perfect! I’ll take a look at it in the morning.
    
36. David Pesce
    
    If anyone else wants to that’s fine as well.
    
37. I should be able to deploy the site and replication tomorrow
    
38. Then we can start customizing the interface
    
39. Alexander Bias
    
    I am particularly impressed who quickly you are creating facts and solutions here :)
    
40. David Pesce
    
    I’m dedicating quite a bit of time to this, but also others.
    
41. The thought is that if we lose plugins, we lose our say in Moodle. They are our works and efforts
    
42. We should be able to choose how they are published, secured, and monetized.
    
43. Alexander Bias
    
    I am deeply grateful for your approach
    
44. David Pesce
    
    > [
    > 
    > Alexander Bias
    > 
    > I am deeply grateful for your approach
    > 
    > 
    > 
    > ](https://matrix.to/#/!cvBzQDMQGakZxaxBLU:matrix.org/$NZQRb_4ZCYfYPv6LtwEtsLI2WbOSBUlCRGkbAp_tfpc?via=matrix.org)
    
    We’ll need your help very soon!
    
45. And the community in general.
    
46. Alexander Bias
    
    HQ just posted this in the Dev Chat as you might have seen:  
    [https://moodle.atlassian.net/wiki/external/OTE3YmUxNTI3YzYzNGJmMzgwZWVjNTEzYTk2ZGE2ZDk](https://moodle.atlassian.net/wiki/external/OTE3YmUxNTI3YzYzNGJmMzgwZWVjNTEzYTk2ZGE2ZDk)
    
    There I read:
    
    > Installing plugins directly from Moodle LMS through the Plugins Directory is no longer supported.
    
    David, I think your solution has just got another unique selling point ;)
    
    [Confluence](https://moodle.atlassian.net/wiki/external/OTE3YmUxNTI3YzYzNGJmMzgwZWVjNTEzYTk2ZGE2ZDk)
    
    {"serverDuration": 36, "requestCorrelationId": "582325cee8874b5cb845744ad700690f"}
    
    moodle.atlassian.net
    
47. David Pesce
    
    > [
    > 
    > Alexander Bias
    > 
    > 
    > 
    > ](https://matrix.to/#/!cvBzQDMQGakZxaxBLU:matrix.org/$70E-xtwdSImsJ528_l52-0nywgwCmdrcSPWPU5NrgRg?via=matrix.org)
    > 
    > [](https://matrix.to/#/!cvBzQDMQGakZxaxBLU:matrix.org/$70E-xtwdSImsJ528_l52-0nywgwCmdrcSPWPU5NrgRg?via=matrix.org)
    > 
    > [](https://matrix.to/#/!cvBzQDMQGakZxaxBLU:matrix.org/$70E-xtwdSImsJ528_l52-0nywgwCmdrcSPWPU5NrgRg?via=matrix.org)
    > 
    > [HQ just posted this in the Dev Chat as you might have seen:  
    > ](https://matrix.to/#/!cvBzQDMQGakZxaxBLU:matrix.org/$70E-xtwdSImsJ528_l52-0nywgwCmdrcSPWPU5NrgRg?via=matrix.org)[https://moodle.atlassian.net/wiki/external/OTE3YmUxNTI3YzYzNGJmMzgwZWVjNTEzYTk2ZGE2ZDk](https://moodle.atlassian.net/wiki/external/OTE3YmUxNTI3YzYzNGJmMzgwZWVjNTEzYTk2ZGE2ZDk)
    > 
    > There I read:
    > 
    > > Installing plugins directly from Moodle LMS through the Plugins Directory is no longer supported.
    > 
    > David, I think your solution has just got another unique selling point ;)
    
    Yeah, this was part of my discussion with Scott/Marie. It’s insane that this was overlooked. Remember the last time they removed that functionality? A ton of developers stopped publishing
    
48. 1. Gemma Lesterhuis
        
        Message supprimé
        
    2. Gemma Lesterhuis a quitté le salon
        
49. Jordan Tomkinson
    
    Eh?
    
50. Alexander Bias
    
    Maybe Gemma wanted to protect herself from all these sad marketplace news
    
51. Jordan Tomkinson
    
    It really is sad. I don't even want to pull from market place api anymore
    
52. Alexander Bias
    
    I see some similarities to this:
    
53. [
    
    ![image.png](https://matrix-client.matrix.org/_matrix/media/v3/thumbnail/matrix.org/HCqFHixCbThzXAjWHkkEyQxJ?width=1300&height=975&method=scale&allow_redirect=true)
    
    
    
    ](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/HCqFHixCbThzXAjWHkkEyQxJ?allow_redirect=true)
    
54. Alexander Bias
    
    [
    
    ![image.png](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/iQJDfTdfxSoFVCBNtZuITvKf?allow_redirect=true)
    
    
    
    ](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/iQJDfTdfxSoFVCBNtZuITvKf?allow_redirect=true)
    
55. I think I will get some popcorn now as Tim chimed in
    
56. Petr Skoda
    
    fun times ahead
    
57. Jordan Tomkinson
    
    i always thought dev chat was boring so i never joined, i've been missing out all this time 🍿
    
58. Alexander Bias
    
    better being late to the party than missing it completely, right?
    
59. David Pesce
    
    I'm still just baffled that they thought this was a good idea
    
60. Petr Skoda
    
    they would be much smarter to not enforce it initially, all these problems at the same time makes them look very incompetent
    
61. the non-competition
    
62. David Pesce
    
    [![](https://matrix-client.matrix.org/_matrix/media/v3/thumbnail/matrix.org/kuDaPvMFRXxifPfBDusFSBkg?width=26&height=26&method=crop&allow_redirect=true)David Herney](https://matrix.to/#/@cirano:matrix.org): you're the first one with a claimed plugin in CAMP!
    
63. There should be a badge for that. ;)
    
64. Alexander Bias
    
    David is a real ChAMP
    
65. David Pesce
    
    here's what CAMP looks like inside Moodle:
    
66. [
    
    ![image.png](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/cgBJGbajGbwCBqaPJNfuNSmg?allow_redirect=true)
    
    
    
    ](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/cgBJGbajGbwCBqaPJNfuNSmg?allow_redirect=true)
    
67. Jordan Tomkinson
    
    David, if you want that plugin security reviewed, we can donate some credits :)
    
68. David Pesce
    
    Still testing things out, so don't share this outside yet, but: [https://camp-registry.github.io/camp-index/site/](https://camp-registry.github.io/camp-index/site/)
    
    [CAMP — plugin archive](https://camp-registry.github.io/camp-index/site/)
    
    CAMP plugin archive All types Activity modules (772) Admin tools (264) aiprovider (1) antivirus (9) archivingmod (1) archivingstore (1) assignfeedback (34) assignment (1) assignsubmission (76)
    
    camp-registry.github.io
    
69. it's going to have broken URLs for things like claiming plugins, etc. but we'll get there
    
70. Jordan Tomkinson
    
    the design is very anthropic-like, have you tried v0 ?
    
71. David Pesce
    
    [https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExcTBsNndxbmxhMGhmc2FlYzYzaXV0bzNlb2hnMmVmdDc1dXpncWIybCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/3NgcLVc9B2tEPUUCMz/giphy.gif](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExcTBsNndxbmxhMGhmc2FlYzYzaXV0bzNlb2hnMmVmdDc1dXpncWIybCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/3NgcLVc9B2tEPUUCMz/giphy.gif)
    
    [media1.giphy.com](https://media1.giphy.com/media/v1.Y2lkPTc5MGI3NjExcTBsNndxbmxhMGhmc2FlYzYzaXV0bzNlb2hnMmVmdDc1dXpncWIybCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/3NgcLVc9B2tEPUUCMz/giphy.gif)
    
    media1.giphy.com
    
72. Spent almost no time on UI, but it's easy enough to swap as we have more time
    
73. Jordan Tomkinson
    
    another 200 plugins just dropped off the official api :/
    
74. David Pesce
    
    i thought it was supposed to be left as read-only?
    
75. Jordan Tomkinson
    
    thats what it still says on the website, but its very much not
    
76. 1. Luca Bösch a rejoint le salon
        
77. David Pesce
    
    Kinda cool that even the DACH projects are in here:
    
78. [
    
    ![image.png](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/RXXwhwPoNNSHAJdwvlMRVupv?allow_redirect=true)
    
    
    
    ](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/RXXwhwPoNNSHAJdwvlMRVupv?allow_redirect=true)
    
79. I'm taking a moment to breath. Any questions, comments, objections?
    
80. Alexander Bias
    
    > [
    > 
    > David Pesce
    > 
    > ![image.png](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/RXXwhwPoNNSHAJdwvlMRVupv?allow_redirect=true)
    > 
    > 
    > 
    > 
    > 
    > ](https://matrix.to/#/!cvBzQDMQGakZxaxBLU:matrix.org/$6E616M637fwjqNtlOr0GiVuXgpBpRPasNdC2YJB3j_0?via=matrix.org)
    
    Just shows that your index is up to date :)
    
81. David Pesce
    
    OK, looking for volunteers to help verify plugins as they are claimed!
    
82. Jordan Tomkinson
    
    isnt that Sonnet's job?
    
83. David Pesce
    
    I misspoke, who wants to volunteer to "review" plugins?
    
84. tier 2 - source verified, is able to be installed, but tier 3 needs two individuals to agree on it's promotion
    
85. David Pesce
    
    any complaints with making the style-checker failures a non-blocker?
    
86. Alexander Bias
    
    I am torn in this case. I think several style sniffs in Moodle are stupid / overpedantic, but not following them is a sign of unclean code and ignorance of rules. I would at least like to see that the codesniffer results are presented on the plugin page just like the old repo did so that admins can decide themselves.
    
87. David Pesce
    
    Yeah, that makes sense to me too
    
88. David Pesce
    
    Thoughts?
    
89. [
    
    ![image.png](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/xMWcvFtNkihowANvTpFAqffO?allow_redirect=true)
    
    
    
    ](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/xMWcvFtNkihowANvTpFAqffO?allow_redirect=true)
    
90. I let Claude /design come up with this
    
91. Jordan Tomkinson
    
    give v0.app a try
    
92. but i mean ultimately your going to be limited by the language your using
    
93. David Pesce
    
    Yeah, less worried about language for now.
    
94. Trying v0 now, doesn't look like i need to sign up
    
95. David Pesce
    
    ohhh, that's pretty!
    
96. [
    
    ![image.png](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/kRuExjohVNsnFUzxAbiKEcFa?allow_redirect=true)
    
    
    
    ](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/kRuExjohVNsnFUzxAbiKEcFa?allow_redirect=true)
    
97. [
    
    ![image.png](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/RPvmuDqSNSbjUxWcJHEkwQlj?allow_redirect=true)
    
    
    
    ](https://matrix-client.matrix.org/_matrix/media/v3/download/matrix.org/RPvmuDqSNSbjUxWcJHEkwQlj?allow_redirect=true)
    
98. David Herney
    
    I believe a preview image is always useful. Perhaps it could be part of the information for each approved plugin.
    
99. David Pesce
    
    Yeah, once the .camp directory is implemented in each plugin, you'll have the ability to add images/screenshots
    
100. David Herney
    
    A filter by Moodle version is also needed, right?
    
101. David Pesce
    
    > [
    > 
    > David Herney
    > 
    > A filter by Moodle version is also needed, right?
    > 
    > 
    > 
    > ](https://matrix.to/#/!cvBzQDMQGakZxaxBLU:matrix.org/$BIUmxiH40shpG1i7zUq9S6ECMITWmtYTMx0CPzNiU_k?via=matrix.org)
    
    yeah, good call
    
102. David Herney
    
    A color scheme for the cards, according to labels, also seems important to me, so we can quickly distinguish whether they are free or not. Otherwise, they all look very similar and it can easily go unnoticed.
    
103. I think giving programmers more recognition can be very helpful. I have complete faith in plugins from various developers and immediately dismiss those from certain "companies." Currently, this is something that goes largely unnoticed.
    
104. David Pesce
    
    All great feedback, keep it coming!
    
105. Yeah, I thought about having a "developer" landing page
    
106. Should even be a way when viewing a particular plugin page to "see plugins by this developer"
    
107. Thoughts on the left filtering vs top? I'm partial to the left because there is so much to filter on.
    
108. David Herney
    
    Left, of course.
    
109. Petr Skoda a changé le sujet du salon en « Discussions related to potential alternatives of Moodle Marketplace * https://camp-registry.org/ ».
    
110. Petr Skoda a changé le sujet du salon en « Discussions related to potential alternatives of Moodle Marketplace 1. https://camp-registry.org/ ».
    
111. Petr Skoda a changé le sujet du salon en « Discussions related to Moodle Marketplace alternatives: 1. https://camp-registry.org/ ».
    
112. David Pesce
    
    OK, new version posted. MDLShield badging is also now implemented
    
113. There's also the ability to have a CAMP badge to share
    
114. [https://camp-registry.org/](https://camp-registry.org/)
    
    [CAMP — Community Archive of Moodle Plugins](https://camp-registry.org/)
    
    An independent, mirrorable archive of Moodle plugins, source-verified byte for byte.
    
    camp-registry.org
    
115. Jordan Tomkinson
    
    where do we see the badge?
    
116. David Pesce
    
    You'll need to update the entry in the camp-index.
    
117. Jordan Tomkinson
    
    ah ok. the main listing needs server side pagination, takes a good ~5 seconds to finish rendering before anything becomes clickable - filters, categories etc
    
118. David Pesce
    
    yeah, i think it's also because i'm hosting it on github.
    
119. still need to move it to an actual server
    
120. Jordan Tomkinson
    
    its because the <script> for filtering is at the end of the 5.6mb list of every plugin
    

  

![](https://app.element.io/icons/warning.80e5cc2.svg)![](https://app.element.io/icons/bold.b7f0698.svg)![](https://app.element.io/icons/inline-code.f51200e.svg)![](https://app.element.io/icons/italic.be1e35d.svg)![](https://app.element.io/icons/quote.60f93d6.svg)![](https://app.element.io/icons/strikethrough.17fd61f.svg)

Fils de discussion

Paramètres rapides

Options du salon

Nouvelle conversation

Ce salon est public

Plus d’options

Paramètres de notifications

Plus d’options

Paramètres de notifications

Salon public

Appel vidéo

Fils de discussion

Information du salon

Personnes

Les messages dans ce salon ne sont pas chiffrés de bout en bout

Pièce jointe