

1. Updates/Upgrades and Changes to Moodle Administration: Introducing the New Directory Structure in 5.1 and Composer for Moodle 5.2

![](https://moodle.org/theme/image.php/moodleorg/forum/1787212466/monologo?filtericon=1)

# Installing and upgrading help

### Updates/Upgrades and Changes to Moodle Administration: Introducing the New Directory Structure in 5.1 and Composer for Moodle 5.2

par [Visvanath Ratnaweera](https://moodle.org/user/view.php?id=41095&course=5), mercredi 19 août 2026, 21:42

Nombre de réponses : 23

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers") ![Avatar Translators](https://moodle.org/pluginfile.php/53/group/icon/173/f1?rev=1446096 "Avatar Translators")

Adams Jenkins aka WiseCat summarizes the recurring topics in this forum that were introduced by Moodle 5.1 and 5.2 in his presentation at the MoodleMoot Estonia, July 2026:

**Updates, Upgrades and Changes to Moodle Administration**  
[https://www.youtube.com/watch?foo=bar&v=izVUPboucNk](https://www.youtube.com/watch?foo=bar&v=izVUPboucNk)

It covers a couple of topics:

1. When to Update/Upgrade
2. Directory Structure Changes
3. Composer is here
4. Router is here, Router is coming
5. What about people on shared hosting?

Since they are widely different topics, I suggest opening a sub-thread for each topic -  with matching subject line (like [here](https://moodle.org/mod/forum/discuss.php?d=482322#p1909848)).

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909847&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Visvanath Ratnaweera](https://moodle.org/pluginfile.php/2987/user/icon/moodleorg/f1?rev=2296189 "Avatar Visvanath Ratnaweera")

### 1) When to Update/Upgrade

par [Visvanath Ratnaweera](https://moodle.org/user/view.php?id=41095&course=5), mercredi 19 août 2026, 22:10

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers") ![Avatar Translators](https://moodle.org/pluginfile.php/53/group/icon/173/f1?rev=1446096 "Avatar Translators")

Out of the two messages, "Follow point releases every two months" is convincing the second message, "Not to go LTS to LTS" is controversial. If that is the truth, why all this hullaboo about LTS?

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909848&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Ken Task](https://moodle.org/pluginfile.php/20282/user/icon/moodleorg/f1?rev=1 "Avatar Ken Task")

### 1) When to Update/Upgrade

par [Ken Task](https://moodle.org/user/view.php?id=141618&course=5), jeudi 20 août 2026, 01:38

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers")

Me thinks ... 

**From an LTS to the next LTS - hyperjump ... NO!**  
4.5 highest is now .13  
to the next LTS (which hasn't been released yet)  
would skip 5.0 - which had 9 point releases  
would skip 5.1 = which is now .6  
would skip 5.2 - which is now .2

That's a minimum of 17 updates ... some minor ... some not.

From moodle [https://moodledev.io/general/releases](https://moodledev.io/general/releases)

_Minor (Point) (eg. 3.x.y) 2 monthly February, April, June, August, October, December_  
(but I've seen point releases nearly every week - some minor and some not!).

_Minor releases dates differ slightly from release to release depending on the timing of public holidays in Western Australia._

4.5 introduced Router but it wasn't really active.  
5.0 no Router change ...  
**BIG changes  ...**   
5.1 introduced new structure and router now in play.   
5.2 router more in play.

MoodleMarket Place for plugins a factor + updates to plugins to be compat with core.

Presenter did mention the issue with shared hosting, but I can state for a fact that update/upgrades issues exist with VPS's - depending upon [hosting provider](https://moodle.org/mod/glossary/showentry.php?eid=22&displayformat=dictionary "Glossary of common terms : hosting provider").

Sooooooooo .... only thing that is certain ... change!

'SoS', Ken

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909851&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar James Steerpike](https://moodle.org/pluginfile.php/1879320/user/icon/moodleorg/f1?rev=2377205 "Avatar James Steerpike")

### 1) When to Update/Upgrade

par [James Steerpike](https://moodle.org/user/view.php?id=1736745&course=5), jeudi 20 août 2026, 01:42

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers")

Wisecat makes a clear distinction between updates and upgrades, one that is perhaps not well understood. Updates are essential as he points out in a another video and every point release ( ie 5.2.**2**) should be done within a week of that release before the security holes fixed are publicly released. Upgrades are  optional ( as long as security patches are supported) and not a decision a SysAdmin makes. What was not mentioned in the video was the process of checking plugins, teaching workflows and teacher training on required new features after each upgrade. The decision to do this 4 times as often as required for LTS may not suit everyone.

Moyenne des évaluations [Useful (1)](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909852&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Visvanath Ratnaweera](https://moodle.org/pluginfile.php/2987/user/icon/moodleorg/f1?rev=2296189 "Avatar Visvanath Ratnaweera")

### 1) When to Update/Upgrade

par [Visvanath Ratnaweera](https://moodle.org/user/view.php?id=41095&course=5), jeudi 20 août 2026, 03:50

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers") ![Avatar Translators](https://moodle.org/pluginfile.php/53/group/icon/173/f1?rev=1446096 "Avatar Translators")

Yeah, in an "update/upgrade" discussion one has to use certain terms consistently. Wisecat uses the same terms as in [https://moodledev.io/general/releases](https://moodledev.io/general/releases), which makes sense.  
  
For example, limiting ourselves to existing releases (not future releases),  
- 4.5, 5.0, 5.1,.. are major releases, what the community casually call Moodle versions  
- 4.5.0, 4.5.1,.. 4.5.13, 5.0.0, 5.0.1,.. 5.0.9 are minor or point releases or just releases.  
- Any transition, say from 4.5.x to 4.5.y are updates  
- Every transition, say 4.5.x to 5.0.y, or from 4.5.x to 5.1.z are upgrades  
  
Wisecat's message is,  
a) keep your Moodle updated to the most recent point release. Today, if you are on 4.5, it needs to be 4.5.13 from 10 August 2026 - which is not disputed.  
  
b) Also follow every major release. So, today you must be either

- on 5.2, then on 5.2.2 to be exact, next major being 5.3 before October 2027, or

- on 5.1, then on 5.1.6 to be exact, next major being 5.2 before April 2027, 

as opposed to being on 4.5, 4.5.13 to be exact, next major being 5.3 before October 2027 - the "LTS to LTS" strategy. That is where the controversy is.

Yes, if you follow Wisecat's advice, there will be a new major (upgrade) every 6 months, whereas if you follow the "LTS to LTS" strategy, you need a major (upgrade) only every 2 years.

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909853&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Visvanath Ratnaweera](https://moodle.org/pluginfile.php/2987/user/icon/moodleorg/f1?rev=2296189 "Avatar Visvanath Ratnaweera")

### 2) Directory Structure Changes

par [Visvanath Ratnaweera](https://moodle.org/user/view.php?id=41095&course=5), jeudi 20 août 2026, 04:02

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers") ![Avatar Translators](https://moodle.org/pluginfile.php/53/group/icon/173/f1?rev=1446096 "Avatar Translators")

**On 2) Directory Structure Changes**

This has been thoroughly discussed in this forum. Only a small change, point the DocumentRoot of the site to /PATH/TO/MOODLE/public - provided that your Moodle site is on the [domain](https://moodle.org/mod/glossary/showentry.php?eid=17&displayformat=dictionary "Glossary of common terms : domain"), [https://example.com/](https://example.com/) or [https://something.example.com/](https://something.example.com/). If your Moodle has a path, [https://example.com/path/](https://example.com/path/), then you need a symlink.

Interestingly, Wisecat advises the symlink even if the site is on the domain, i.e. the symlink is not a must, arguing that it improves security.

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909854&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar James Steerpike](https://moodle.org/pluginfile.php/1879320/user/icon/moodleorg/f1?rev=2377205 "Avatar James Steerpike")

### 1) When to Update/Upgrade

par [James Steerpike](https://moodle.org/user/view.php?id=1736745&course=5), jeudi 20 août 2026, 05:43

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers")

Ken,

Just for interest, why would a hosting provider cause update or upgrade issues on a VPS?

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909856&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Ken Task](https://moodle.org/pluginfile.php/20282/user/icon/moodleorg/f1?rev=1 "Avatar Ken Task")

### 1) When to Update/Upgrade

par [Ken Task](https://moodle.org/user/view.php?id=141618&course=5), jeudi 20 août 2026, 07:27

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers")

Yeah!  I don't get it either ... but ... have run across one ... SG!

SG strat - fewer customers on each [VM](https://moodle.org/mod/glossary/showentry.php?eid=10474&displayformat=dictionary "Glossary of common terms : VM") and call them VPS's ... but with higher limitations/caps.   Can't change document root.   Must contact tech support to request, but responses from support staff offer 2 alternatives ... .htaccess (on nginx) or symlinks for public_html.   There are about 6 PHP settings that cannot be changed.   SG says they are set optimal for 'most customers' - but 'most customers' don't choose SG for hosting multiple moodles.  About ALL hosting providers, they do WordPress and do it well.   Have and developing their own Panel ... not cPanel/[Plesk](https://moodle.org/mod/glossary/showentry.php?eid=10385&displayformat=dictionary "Glossary of common terms : Plesk") (the well-knowns) - no Terminal ... and there might not be [Git](https://moodle.org/mod/glossary/showentry.php?eid=10110&displayformat=dictionary "Glossary of common terms : Git") either.

Am hoping one of the entities am attempting to assist with SG will setup a 'collaborator' account so I can explore more.   So far that's been a very slow process.

Of course, we get to 'pointing the finger' ... customer has hosted with SG for years without issues ... until decision made to upgrade to 5.2 from 4.5.x.  'Never had to do that before!!!!' is commonly expressed.   And, don't really appreciate hearing "we're not in Kansas anymore, Toto!" ![triste](https://moodle.org/theme/image.php/moodleorg/core/1787212466/s/sad "triste")

'SoS', Ken

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909859&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Ken Task](https://moodle.org/pluginfile.php/20282/user/icon/moodleorg/f1?rev=1 "Avatar Ken Task")

### 1) When to Update/Upgrade

par [Ken Task](https://moodle.org/user/view.php?id=141618&course=5), jeudi 20 août 2026, 07:43

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers")

This might be better described using git.

If on 4.5.x, only a git pull (if using branches) is required to get the most recent 4.5.x core code.

If on 4.5.x, to 'upgrade' to 5.0, or 5.1, or 5.2, one must issue a couple more commands to tell git the branch to track and to acquire the higher version core code.  In git version numbers: 500, 501, 502, and soon to be released 503 (think of the 0's as a 'point').

Moodle has never had a way to update/upgrade in the Admin interface.   But it sure has been misleading for some when going to Notifications and one sees a [download](https://moodle.org/mod/glossary/showentry.php?eid=18&displayformat=dictionary "Glossary of common terms : download") button for every higher version shown there - suggesting they can easily 'hyperjump'!

'SoS', Ken

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909861&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Visvanath Ratnaweera](https://moodle.org/pluginfile.php/2987/user/icon/moodleorg/f1?rev=2296189 "Avatar Visvanath Ratnaweera")

### 5) What about people on shared hosting?

par [Visvanath Ratnaweera](https://moodle.org/user/view.php?id=41095&course=5), jeudi 20 août 2026, 07:49

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers") ![Avatar Translators](https://moodle.org/pluginfile.php/53/group/icon/173/f1?rev=1446096 "Avatar Translators")

On

**5) What about people on shared hosting?  
**[https://www.youtube.com/watch?foo=bar&v=izVUPboucNk](https://www.youtube.com/watch?foo=bar&v=izVUPboucNk)  starting 25:00

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909862&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Visvanath Ratnaweera](https://moodle.org/pluginfile.php/2987/user/icon/moodleorg/f1?rev=2296189 "Avatar Visvanath Ratnaweera")

### Upgrading to Moodle 5 on Siteground

par [Visvanath Ratnaweera](https://moodle.org/user/view.php?id=41095&course=5), jeudi 20 août 2026, 07:52

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers") ![Avatar Translators](https://moodle.org/pluginfile.php/53/group/icon/173/f1?rev=1446096 "Avatar Translators")

Ken  
  
Probably you mean [Upgrading from 4.5.8 to 5.2 broke on Siteground](https://moodle.org/mod/forum/discuss.php?d=482245#p1909717)? Isn't it a very specific case of the 1) When to Update/Upgrade topic in the presentation? As it  seems Siteground is specially nasty trying to sell shared hosting labeled as VPS. (From what you've reported, no first hand knowledge.)  
  

Aren't we are at the final topic in the presentation, [5) What about people on shared hosting](https://moodle.org/mod/forum/discuss.php?d=482322#p1909862)?

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909863&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar James Steerpike](https://moodle.org/pluginfile.php/1879320/user/icon/moodleorg/f1?rev=2377205 "Avatar James Steerpike")

### Upgrading to Moodle 5 on Siteground

par [James Steerpike](https://moodle.org/user/view.php?id=1736745&course=5), jeudi 20 août 2026, 08:14

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers")

According to WC, doomed. The two questions:  
1. Is it possible to set up the router and composer on most shared hosting sites?  
2. Can the average SH user do the necessary configurations?  
  
And siteground are definitely being creative in labeling plans as VPS hosting.

Moyenne des évaluations [Useful (1)](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909867&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Ken Task](https://moodle.org/pluginfile.php/20282/user/icon/moodleorg/f1?rev=1 "Avatar Ken Task")

### Upgrading to Moodle 5 on Siteground

par [Ken Task](https://moodle.org/user/view.php?id=141618&course=5), jeudi 20 août 2026, 08:42

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers")

So, you can 'jabberwalk', but no one else can!   So we changed the subject line cause it's not related, huh?  Hmmmmm....

SG ... 'first hand experience' at watching in a Zoom meeting and discovering [OP](https://moodle.org/mod/glossary/showentry.php?eid=10401&displayformat=dictionary "Glossary of common terms : OP") wasn't all that familiar with that interface. ![triste](https://moodle.org/theme/image.php/moodleorg/core/1787212466/s/sad "triste").  But that's also true of folks who have cPanel/Plesk/other - in my experience!

'Shared hosted' folks took that path based purely on cost ... not knowing future needs of a moodle that grew!   In their defense, no one knows future of anything!

But there are other 'catch 22's with shared hosting besides memory/space/frequency of running a process (cron) ... like inode limits.  No work-arounds there!

Outta here ... said my 2 cents worth! ![sourire](https://moodle.org/theme/image.php/moodleorg/core/1787212466/s/smiley "sourire")

'SoS', Ken

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909869&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Visvanath Ratnaweera](https://moodle.org/pluginfile.php/2987/user/icon/moodleorg/f1?rev=2296189 "Avatar Visvanath Ratnaweera")

### 5) What about people on shared hosting?

par [Visvanath Ratnaweera](https://moodle.org/user/view.php?id=41095&course=5), jeudi 20 août 2026, 08:43

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers") ![Avatar Translators](https://moodle.org/pluginfile.php/53/group/icon/173/f1?rev=1446096 "Avatar Translators")

For those who prefer not to watch videos, here is the transcript:  
  
25:02 And so talking about that, uh, shared hosting, you might find if you're if you're hosting your Moodle on  
25:09 a shared hosting platform, uh, where you're not actually running the actual OS and every aspect of the [server](https://moodle.org/mod/glossary/showentry.php?eid=30&displayformat=dictionary "Glossary of common terms: server"), you  
25:16 might find that hard because what configuration you can do to the web server configuration might be limited by your provider.  
25:24 Every provider will be different in this aspect. So my take on that is that um shared hosting environments can't really  
25:33 be claimed to be supported by Moodle because they're that's not shared hosting environments are not actually a  
25:40 thing. They're a realm and so there's no way of supporting a realm. So I often like to say that  
25:48 people on shared hosting, what about them? What you know, we don't want to leave them behind because we we still love you guys, but uh you're doomed. And  
25:56 I mean doomed in terms of the game Doom, which  
26:03 is a old uh 1993 shooter, first person shooter. I'm pretty sure everybody here would remember this game. But the cool  
26:12 thing about Doom is that it spawned a meme called can it run Doom? And you find nowadays uh people running um smart  
26:21 watches, they run Doom everywhere. And so instead of can you run Doom, it's I think in the future we have to be can it  
26:28 run Moodle. And this is just the challenge that we're accepting if we're choosing not to use a VPS or bare metal  
26:35 or um a platform where uh we have full control over the web server config. So  
26:42 can you run Moodle on it? Well, you can run Doom on a that's a little um Adafruit board. Uh that's on an ATM.  
26:49 Doom running on an ATM. That's it on a Super Mario Brothers game and on an Apple Watch uh on an old Kodak digital  
26:57 camera from like that would be like the early n early 90s I want to say uh on an old calculator  
27:05 on a printer and I don't know why you would want to run Doom on a printer but anyway it it has been done. So where are you going to run your Moodle?  
27:15 Um, so my thoughts are on the whole idea of shared hosting and things like that.  
27:20 There are complexities that are changing on Moodle. Moodle is changing. The way it's got to be admin is changing. We  
27:26 have to adapt. We can't tell Moodle not to move forward with the router with composer. We can't we can't ask Moodle  
27:34 to stop that. And I don't think Moodle should try to um make sure that uh if  
27:41 we're not ready that they're waiting for us all the time. They shouldn't wait forever. Just progress forward and it's  
27:48 our job to keep up with them. So, um I still though I do re I do respect though  
27:55 that Moodle HQ tries really really hard to make sure everybody is well supported  
28:01 and it's hard and respect to that. But I think it would be well my position is shared hosting environments should not really be supported.  
28:11 So um anyway uh that's uh pretty much it for today.

Summary:

_But I think it would be well my position is shared hosting environments should not really be supported._

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909870&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar Visvanath Ratnaweera](https://moodle.org/pluginfile.php/2987/user/icon/moodleorg/f1?rev=2296189 "Avatar Visvanath Ratnaweera")

### Upgrading to Moodle 5 on Siteground

par [Visvanath Ratnaweera](https://moodle.org/user/view.php?id=41095&course=5), jeudi 20 août 2026, 08:49

![Avatar Particularly helpful Moodlers](https://moodle.org/pluginfile.php/53/group/icon/1/f1?rev=1446108 "Avatar Particularly helpful Moodlers") ![Avatar Translators](https://moodle.org/pluginfile.php/53/group/icon/173/f1?rev=1446096 "Avatar Translators")

Shared hosting "doomed". And for the answer to the two questions, pl. continue in its own topic  [5) What about people on shared hosting?](https://moodle.org/mod/forum/discuss.php?d=482322#p1909862)  
  
The specific case Siteground is better suited in its original discussion: [Upgrading from 4.5.8 to 5.2 broke on Siteground?](https://moodle.org/mod/forum/discuss.php?d=482245#p1909717)

Moyenne des évaluations  [-](https://moodle.org/rating/index.php?contextid=133&component=mod_forum&ratingarea=post&itemid=1909871&scaleid=-88)Évaluer...Useful[](https://moodle.org/course/scales.php?id=5&list=1&scaleid=88)

![Avatar James Steerpike|36](https://moodle.org/pluginfile.php/1879320/user/icon/moodleorg/f1?rev=2377205 "Avatar James Steerpike")

par [James Steerpike](https://moodle.org/user/view.php?id=1736745&course=5), jeudi 20 août 2026, 10:37
'Shared hosted' folks took that path based purely on cost '  
Bluehost is offering shared hosting for $3.99 a month for a 36 month term, then $9.99 while a 1 vCPU Core. 2 GB DDR5 RAM with 50GB NVMe Storage VPS goes for $4.69 for the first 24 months, then $5.69.  
Less than a dollar a month more for a VPS and cheaper once you are hooked.
Shared hosting isn't being chosen on cost. It is chosen because a VPS was seen as being too difficult, not for the non techies but a shared hosting plan isn't so easy once Moodle needs more than just unzipping into public_html.

![Avatar Andrew Lyons|45](https://moodle.org/pluginfile.php/76798/user/icon/moodleorg/f1?rev=2319149 "Avatar Andrew Lyons")

### Shared Hosting problem

par [Andrew Lyons](https://moodle.org/user/view.php?id=268794&course=5), lundi 20 octobre 2025, 14:55

My issue with shared hosting is that you have little-to-no control over the software installed.  
  
For example, I recently installed a cPanel host on Namecheap, and they only offer version 10 of [Postgres](https://moodle.org/mod/glossary/showentry.php?eid=4363&displayformat=dictionary "Glossary of common terms : postgres"). That version was released in 2017, and went out of support in November 2022.
## Moodle :
 
 - application LAMP source PHP web server base de données  - source PHP - vcs git version X.Y.Z
   Moodle suit  a peu pres le modele semver
- X serie les depreciations deviennent effective lors du chagement de série
	- un plugin fonctionnant pour une version Moodle, foctionnera pour les versions suivantes de la série
	- X.Y version majeure introduction new features, corrections
	- .X.Y.Z corections de la version X.Y
	- one branch for each major version
	- each release coresponds  a tag
## Plugins
- apporte des fonctionnalites supplémentaires
- nom normalisé  <<type_name>
-  version.php : metadata du plugin

### role du catalogue

-  administation du plugin au cours de sa vie
	-  critéres d'acceptation
	-  unicité des noms 
	- changement de mainteneur,  
	

## CodeBase
A codebase is ==the complete collection of source code, configuration files, and assets maintained together to build a software system==. It goes beyond raw source code by including documentation, build scripts, and dependencies.
[1](https://www.sonarsource.com/resources/library/code-base-in-software-development/)
[2](https://en.wikipedia.org/wiki/Codebase)
[3](https://www.quora.com/What-is-the-difference-between-codebase-and-source-code)

Core Components

- **Source code**: human-readable instructions written in programming languages.

- **Configuration**: settings required to run or compile the program.

- **Tooling and tests**: scripts used to automate deployment and verify functionality.
	[1](https://en.wikipedia.org/wiki/Codebase)
	[2]](https://www.sonarsource.com/resources/library/code-base-in-software-development/)

### installation d'un plugin

- simplemnt recopie du sour du plugin dans un répertoire qui dépend du type de plugin


from ![Avatar James Steerpike|32](https://moodle.org/pluginfile.php/1879320/user/icon/moodleorg/f1?rev=2377205 "Avatar James Steerpike")

### Composer install of dependencies & superuser account

par [James Steerpike](https://moodle.org/user/view.php?id=1736745&course=5)
Installing composer as root could allow plugins to do naughty stuff. A solution is to create a new user specifically for composer and add the web server to the group.

AI suggested something like this (The web server may be apache and not www-data if you are on Redhat based )  You could also set your regular no sudo account as the one installing composer.

# Create a deploy user if you don't have one  
sudo useradd -m -s /bin/bash deploy

# Run composer as the deploy user  
sudo -u deploy composer install --no-dev  --classmap-authoritative

sudo chown -R deploy:www-data vendor
sudo chmod -R 644 vendor

see also https://getcomposer.org/doc/faqs/how-to-install-untrusted-packages-safely.md

Please see:

[https://docs.moodle.org/502/en/Installing_plugins](https://docs.moodle.org/502/en/Installing_plugins)

[https://moodledev.io/general/development/tools/composer](https://moodledev.io/general/development/tools/composer)

As well as:

[https://docs.moodle.org/502/en/Composer_vendor_directory_not_found](https://docs.moodle.org/502/en/Composer_vendor_directory_not_found)

Moodle's routing system uses a **front controller** pattern (`r.php`) rather than being "front loading". Introduced in Moodle 5.1 and enhanced with strict configuration checks in Moodle 5.2, the router directs web traffic through a central script when clean URLs or specific components are called. [[1](https://moodle.org/mod/forum/discuss.php?d=473950), [2](https://docs.moodle.org/en/Configuring_the_Router), [3](https://pimenko.com/en/moodle-5-2-new-features-2026/), [4](https://www.cmsgalaxy.com/blog/complete-tutorial-upgrade-moodle-to-5-2-on-cpanel-shared-hosting-step-by-step/), [5](https://moodle.org/mod/forum/discuss.php?d=474506)]

Router and Front Controller Setup

- **Front Controller:** Moodle uses `r.php` in the main directory as the router target handler.

- **Web Root:** The server document root must point to the `/public` directory.

- **Server Rules:** Handlers like `FallbackResource /r.php` or URL rewrite rules map requests that do not match physical files directly into the routing system.

- **Environment Check:** Moodle 5.2 includes a specific system check to verify that your web server and router are set up correctly. [[1](https://www.cmsgalaxy.com/blog/complete-tutorial-upgrade-moodle-to-5-2-on-cpanel-shared-hosting-step-by-step/), [2](https://moodle.org/mod/forum/discuss.php?d=473950), [3](https://docs.moodle.org/en/Configuring_the_Router), [4](https://pimenko.com/en/moodle-5-2-new-features-2026/)]


    
  
    
    ![](data:image/png;base64,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

[Configuring the router](https://docs.moodle.org/502/en/Configuring_the_Router)

ftom [catalyst](https://www.catalyst-eu.net/blog/2024/04/02/the-xz-backdoor-and-moodle)
One area that should create pause for thought is third-party plugins. The [Moodle Plugins database](https://moodle.org/plugins) contains over 2000 plugins, including many that haven’t been updated in several years. It’s conceivable that a malicious party could pressure a maintainer of a popular plugin who is no longer able to work on it to hand maintainership over to them. They could then follow a similar path to the XZ exploit, releasing a compromised version and encouraging users to update.

When adding plugins to your Moodle site, it is important to perform due diligence and ensure that the plugin is being well-maintained. A well-maintained plugin will have regular updates from a variety of developers, meaning the pressure isn’t piled on one individual. Bug reports will be responded to, and Pull Requests will be merged or rejected. If a project has financial sponsorship or commercial backing, the developers can dedicate more of their time to it. In the Moodle community, you can often meet developers at in-person MoodleMoots or online events, so you may be able to associate on-screen names with real individuals.

With a site that uses plugins, you should also perform regular reviews to ensure the due diligence you performed at the start still holds up. Does the plugin have a release for the current Moodle version? Is it still actively maintained? Has it changed hands since you first installed it?

Catalyst can help with all of this. We offer a comprehensive plugin review service which will assess the quality and security of the code itself, but also the health and sustainability of the project that produces it.

During upgrades, we can help review your plugins and remove any that have become obsolete or aren’t well maintained. If an unmaintained plugin is crucial to your organisation, the open source nature of Moodle’s ecosystem allows Catalyst to take over maintenance on your behalf, ensuring you stay secure into the future.