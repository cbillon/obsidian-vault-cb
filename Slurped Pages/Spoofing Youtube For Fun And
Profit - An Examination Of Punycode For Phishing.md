---
link: https://grahamhelton.com/blog/punycode
site: Graham Helton
excerpt: |-
  Exploring some of the
  interesting edge cases when it comes to buying domains with non-standard
  characters!
twitter: https://twitter.com/@grahamhelton3
slurped: 2025-12-17T09:35
title: |-
  Spoofing Youtube For Fun And
  Profit: An Examination Of Punycode For Phishing
---

## Before we begin

Edge cases are often the reason infosec professionals have jobs and understanding those edge cases can be a career path all on its own. For me, exploring these edge cases can make for some very interesting research. This post is the result of nearly a year’s worth of pulling different threads trying to learn as much as I can about phishing, punycode, and spoofing domains. My hope for this post is to document some of the interesting information I’ve found over the last year or so and maybe teach you a thing or two. If you enjoy this post, let me know [on twitter](http://twitter.grahamhelton.com/)!

## Table of Contents

- Introduction To This Blog
- A Very Brief Overview of Domain Squatting
- Intro To Punycode
    - Punycode, Big Problems
- Punycode Proof Of Concept
    - Setting Up The Punycode Domain
    - Moving To EC2
    - Cloning Youtube
    - Getting A Valid SSL Certificate
    - The Meta Tag Problem
- Parting Thoughts

## A Very Brief Overview Of Domain Squatting

Before we get into the interesting part of this research, we very briefly need to talk about domain squatting/typo squatting. Domain squatting is a fairly common technique that is sometimes used by nefarious people for various purposes such as stealing business, preying on those who mistype a URL, or simply to cause chaos. In fact, my domain has been _squatted_ which you can [read about here](https://www.grahamhelton.com/blog/roundup5/) if you’d like. Some common techniques for domain squatting are as follows:

1. **Bitsquatting:** Swapping one or two letters in a domain name with similar characters. grahamhelton.com -> grahannhelton.com
2. **Repetition:** Duplicating a letter in a domain name. grahamhelton.com -> grahamhellton.com
3. **Omission:** Omitting a letter in a domain name. grahamhelton.com -> grahamheton.com

You’re smart, you get the point. These are some of the more well known techniques that everyone knows of. There is a famous case of someone registering a domain `goggle.com` (a very common misspelling of `google.com`) that for years was a website that had all kinds of sketchy redirects and adware pages. So how could this be taken one step further?

## Intro To Punycode

During a phishing engagement, it is fairly common to buy a domain very similar to the company you are phishing. This can be done through any of the aforementioned techniques domain squatting techniques, but what if you want to take this one step further? What if you wanted to get a domain that was nearly identical to your target? Well first we need to talk about ASCII (sorry…).

The internet was first created in the United States where we have an alphabet that contains characters A-Z with some numbers and special characters mixed in there. There was really no need for non-English letters to be used in domain names so ASCII characters were used when setting up protocols.

This was all fine and dandy until other countries began accessing the internet and people quickly realized that ASCII characters would not suffice. For example, what happens if someone wants to register a domain in a different language such as Vietnamese? This presents a problem because the Vietnamese alphabet has many characters that are not available in the ASCII character set such as:

```
Unmarked     | A/a, Ă/ă, Â/â, E/e, Ê/ê, I/i, O/o, Ô/ô, Ơ/ơ, U/u, Ư/ư, Y/y
Grave Accent | À/à, Ằ/ằ, Ầ/ầ, È/è, Ề/ề, Ì/ì, Ò/ò, Ồ/ồ, Ờ/ờ, Ù/ù, Ừ/ừ, Ỳ/ỳ
Hook Above   | Ả/ả, Ẳ/ẳ, Ẩ/ẩ, Ẻ/ẻ, Ể/ể, Ỉ/ỉ, Ỏ/ỏ, Ổ/ổ, Ở/ở, Ủ/ủ, Ử/ử, Ỷ/ỷ
Tilde        | Ã/ã, Ẵ/ẵ, Ẫ/ẫ, Ẽ/ẽ, Ễ/ễ, Ĩ/ĩ, Õ/õ, Ỗ/ỗ, Ỡ/ỡ, Ũ/ũ, Ữ/ữ, Ỹ/ỹ
Acute Accent | Á/á, Ắ/ắ, Ấ/ấ, É/é, Ế/ế, Í/í, Ó/ó, Ố/ố, Ớ/ớ, Ú/ú, Ứ/ứ, Ý/ý
Dot Below    | Ạ/ạ, Ặ/ặ, Ậ/ậ, Ẹ/ẹ, Ệ/ệ, Ị/ị, Ọ/ọ, Ộ/ộ, Ợ/ợ, Ụ/ụ, Ự/ự, Ỵ/ỵ
```

So how can you register domains with these characters if you can only have a domain with ASCII characters? Introducing punycode! Punycode is a funny little encoding syntax for non-ASCII domains. It was first defined in March 2003 in [RFC3492](https://datatracker.ietf.org/doc/html/rfc3492). Essentially, all this encoding scheme does is convert a non-ASCII character such as `ê` into a format that only contains ASCII characters, thus making it compatible with pre-existing protocols such as DNS. An example of this encoding would be changing `hêllo.com` to `xn--hllo-gpa.com` after punycode encoding. Essentially, punycode adds `xn--` to the beginning of the domain, removes the non-ASCII letter, and encodes it at the end with `-<encoded value>.com` This means when you’re buying a domain with a non-ASCII character, you’re actually buying the punycode equivalent. This can cause some unexpected issues down the line if you don’t know much about punycode domains before buying one. You can play around with punycode encoding using [this calculator](https://www.punycoder.com/).

## Punycode, Big Problems

Buying a domain with punycode is a novel concept but in practice, it can cause a few issues that you should be aware of before a phishing engagement.

1. You should know what software your target is using before deciding if a punycode encoded domain is right for you.

- Different software handles punycode encoded domains differently. For example, typing `https://ỵoutube.com` in slack will automatically render into `https://xn-outube-ot8b.com`. Interestingly, receiving a punycode link in protonmail does not show the punycode encoded version.

![](https://grahamhelton.com/assets/images/slack.gif)

![](https://grahamhelton.com/assets/images/Pasted-image-20220130142235.png)

- This behavior could end your campaign before it begins. If you spend time creating a phishing campaign with a specially crafted domain that has a non-standard character, your campaign can quickly blow up in your face if slack takes all your hard work and encodes it with punycode.

2. A savvy security team could force the translation of non-standard characters into punycode in URL bars.  
    

- This will make it so any link someone clicks will only show the encoded version of the domain which will be a red flag for most people.
    - In firefox this can be achieved by entering `about:config` -> searching for `IDN_show_punycode` -> Changing from False to True. This will make `https://ỵoutube.com` appear in its encoded format. `https://xn-outube-ot8b.com`

![](https://grahamhelton.com/assets/images/Pasted-image-20220129225104.png)

3. When conducting a phishing campaign, you’ll be sending emails. Some email sending services such as [mailgun](https://grahamhelton.com/blog/www.mailgun.com) do not support punycode encoded domains.

## Punycode Proof Of Concept

With the theoretical knowledge out of the way, I wanted to give a high level overview of setting up a domain with non-standard characters. This isn’t meant to be a step by step tutorial but should allow you to recreate my steps if you really want to.

When I first learned that you could buy domains using non-standard ASCII characters I found [AltCodeUnicode.com](https://altcodeunicode.com/). This is a site where you can find all kinds of non-standard ASCII characters. After some poking around I thought a good proof of concept project could be `ỵoutube.com` (Notice the non-standard _ỵ_ character) was available for $12. So of course I bought it. For science.

#### Setting Up The Punycode Domain

Originally I was trying to use the method I talk about in [this blog post](https://www.grahamhelton.com/blog/infosecblog2) to host a simple static site using an amazon S3 bucket but for various reasons this failed.

![](https://grahamhelton.com/assets/images/Pasted-image-20220128213103.png)

I will save you the hours of troubleshooting _why_ this wasn’t working, but the moral of the story is that [according to the AWS bucket naming requirements](https://grahamhelton.com/https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-s3-bucket-naming-requirements.html) this technically works for creating a bucket but doesn’t seem to work for hosting a static site in that bucket. This is probably for the best since I wanted to do more than just server web pages. I figured hosting a site using a classic LAMP stack on an EC2 instance would work well and I shouldn’t run into too many weird AWS specific errors.

#### Moving To EC2

Hosting the site using an EC2 instance was the logical next step because it _should_ have less bizarre constraints since you can just point your DNS A record to the IP address of your EC2 instance. Before I jumped head first into creating an nginx server, I wanted to test to see if I would encounter any issues routing the punycode domain (xn–outube-ot8b.com) to an EC2 IP. Fortunately, it was fairly straight forward to get this working now that I didn’t need to battle any odd S3 bucket nuances. Here is what my DNS record looked up after creating a hosted zone for `xn--outube-ot8b.com`.

![](https://grahamhelton.com/assets/images/Pasted-image-20220128214307.png)

After waiting nearly 24 hours for DNS to update and wondering if I was doing something wrong, I could finally issue `dig ỵoutube.com` and `dig xn--outube-ot8b.com`. Now that DNS was up, I could visit the site in firefox by creating a quick python server using `sudo python3 -m http.server 80`. I am still not sure if the punycode domain took longer to update in DNS servers around the world than a normal domain name but I’ve never had to wait longer than a few minutes for DNS. It was most likely just a coincidence though.

![](https://grahamhelton.com/assets/images/Pasted-image-20220128214604.png)

![](https://grahamhelton.com/assets/images/Pasted-image-20220128214515.png)

Knowing that I could create a server using this domain means I can do a lot in terms of what I actually wanted to host on the page. If you were creating a phishing engagement you could host the company’s login page, host malicious files, etc.

#### Cloning Youtube

Now that I’ve verified that I can host a web server using the punycode encoded domain `xn--outube-ot8b.com` (`ỵoutube.com`) I needed to figure out _what_ I wanted to host on this sever. Depending on your use case you may want to be creating a login page but for this example I just wanted to see how accurate of a clone I could make of youtube.com. My first thought was to use `wget` to download a youtube video page. This theoretically could have worked but messing with the HTML/CSS to get it looking just right was very tedious.

![](https://grahamhelton.com/assets/images/Pasted-image-20220130145247.png)

Eventually I was tired of fiddling around with those pages I downloaded using `wget` and found a [github repo](https://github.com/KaushikShivam/youtube_clone) with a decent looking clone of youtube that was easy to edit. Next, I cloned this into my EC2 instance and began making some quick changes to the HTML and devising a plan to host it.

You _could_ create an nginx/apache webserver (and I would recommend that if you’re not just doing research), but since this was more of a proof of concept, I decided to stick with a simple python server for now. The only problem was that when running `sudo python3 -m http.server 80`, you would see the youtube clone but the URL would be a dead giveaway since it was simply `ỵoutube.com` without a video ID. To remedy this, I just created a folder with the name corresponding to what I wanted in the URL. In this case `watch?v=D5iap5aO4i99`. Now all I needed to do was start the web server using the previously mention python command, and navigate to the site. It looks fairly realistic but will need a lot of altering as well as a valid SSL certificate.

![](https://grahamhelton.com/assets/images/Pasted-image-20220129164711.png)

#### Getting A Valid SSL Certificate

The biggest glaring issue is the lack of a valid SSL certificate and thus we might see a warning saying that this site might be insecure and we don’t see the lockpad in the browser. Fortunately, using LetsEncrypt makes this fairly simple even when you’re just using a python command to host your server. I found a great writeup from [CornerPirate](https://cornerpirate.com/2021/03/10/getting-a-letsencrypt-certificate-for-your-python-http-servers/) that explains some extra info about how this works. After installing certbot, run the `certbot` command, fill out the information required for a certificate and then create a new python web server using twisted web on port 443, making sure to specify the path to keys.

```
sudo python3 -m twisted web --https=443 --path=. -c /etc/letsencrypt/live/xn--outube-ot8b.com/fullchain.pem -k /etc/letsencrypt/live/xn--outube-ot8b.com/privkey.pem
```

That’s it! Super easy way to get a certificate for you web server!

#### The Meta Tag Problem

My original idea for this research actually had nothing to do with hosting a spoofed site, it actually was just supposed to be a way for me to figure out how [meta tags](https://en.wikipedia.org/wiki/Meta_element) worked. A meta tag (or meta element) is just an HTML element that allows you to add metadata to a web page. This can be almost anything but sites like twitter/facebook/linkedin do special things with meta tags. For example, whenever I post a blog to twitter you’ll notice that it is usually accompanied by a nice little image preview. This is accomplished through meta tags. To get this working, you use a standardized set of meta tags that twitter will parse to render the image. For example, here is how mine are set up.

![](https://grahamhelton.com/assets/images/Pasted-image-20220130151324.png)

**Meta tags I use for my site**

```
<meta name="twitter:card" content="summary_large_image"/>
<meta name="twitter:image" content="https://www.grahamhelton.com/roundup6Twittercard.png"/>
<meta name="twitter:title" content="Weekly Security Roundup #6: January 23rd-30th 2022"/>
<meta name="twitter:description" content="Reading/Writing but not executing (rw-)"/>
<meta property="og:title" content="Weekly Security Roundup #6: January 23rd-30th 2022" />
<meta property="og:description" content="Reading/Writing but not executing (rw-)" />
<meta property="og:type" content="article" />
<meta property="og:url" content="https://www.grahamhelton.com/blog/roundup6/" />
<meta property="og:image" content="https://www.grahamhelton.com/roundup6Twittercard.png" />
<meta property="article:published_time" content="2022-01-30T00:00:00+00:00" />
<meta property="article:modified_time" content="2022-01-30T00:00:00+00:00" />
```

Twitter has many different “Cards” that will display content in a number of different ways. When you post a link to youtube, you get a “player card”. When you post a link to github you get a “summary card”. You can read more [about them here](https://developer.twitter.com/en/docs/twitter-for-websites/cards/overview/abouts-cards). Wouldn’t it be cool if I could post a link to `ỵoutube.com` and have twitter automatically pull the meta tags to display whatever video was on that page in the “player” twitter card just like it does for youtube? You’ve surely seen this, but you might not have know it had anything to do with meta tags. Typically the player card looks something like this.

![](https://grahamhelton.com/assets/images/Pasted-image-20220130151820.png)

I attempted to add “player card” meta tags to `ỵoutube.com` but was running into strange errors when trying to get them working on twitter. Twitter has a very handy [card validator] (https://cards-dev.twitter.com/validator) that you can use to test your twitter cards before posting the links. Unfortunately, I could not get these working using a simple python server or an nginx server. I’m not totally sure how to fix this or if it is possible.

![](https://grahamhelton.com/assets/images/Pasted-image-20220130153301.png)

## The final product

After spending some time editing the HTML template, I even got some comments that weren’t placeholder text/images. Obviously being able to add your “own” comments/views/likes on a video people think is on youtube is not great but also there is not much you can do about it. Here is a quick comparison of a real youtube video to my spoofed one. Left is the spoofed site, right is the real one.

![](https://grahamhelton.com/assets/images/Pasted-image-20220130231403.png)

## Parting Thoughts

1. I do not like working with DNS/web servers.

- Working with DNS/web servers is very time consuming because a lot of times you’re waiting for DNS records to update which in this case took ~24ish hours.

2. Working with meta tags can be a pain.

- Working out the kinks with meta tags can also be frustrating since twitter’s card validator will sometimes cache meta tags for up to a week. This can make testing different ones very time consuming

3. Punysquatting can be very powerful, but it can also be quite frustrating.

- Setting up a domain with non-standard ASCII characters can be a pain to set up correctly and requires a lot of work. It might be a better time investment to buy a look-a-like domain

4. Generally I think one of the only valid uses for a punysquatting domain is phishing.

- Domain squatting typically is successful due to people mistyping a URL but with a punycode domain, its unlikely (at least for those with a standard keyboard) to mistype a domain with a non-standard character

5. Your browser typically automatically converts punycode encoded domains such as `xn--outube-ot8b.com` into their non encoded form `ỵoutube.com`.

- This allows for emoji domains such as i❤️tacos.ws.
- An interesting note is that if a webserver does not exist on that domain, most browsers only show the punycode encoded domain. For example, when I turn the web server on for `ỵoutube.com`, firefox will render it with the Vietnamese `ỵ` character. However, if I do not have a webserver turned on, Firefox does _not_ change `xn--outube-ot8b.com` to `ỵoutube.com`.

> Quick note: The web server used to host `ỵoutube.com` is no longer live which means you cannot visit the site since this was just for research!

I had a fun time over the past few months doing this research and I learned a ton. If you have any questions about this post, please don’t hesitate to reach out to me [on twitter](http://twitter.grahamhelton.com/).