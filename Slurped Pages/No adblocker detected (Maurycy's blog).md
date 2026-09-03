---
link: https://maurycyz.com/misc/ads/
excerpt: >-
  Internet ads are horrible: 

  They waste your time, and the industry behind them is actively making the
  internet a worse place.

  Payouts are so small that the only way to survive is to turn your site into an
  ad-filled hellhole with no real substance.

  (see: Modern social media)
slurped: 2026-08-31T09:19
title: No adblocker detected (Maurycy's blog)
---

**2025-09-08** — **2026-02-23**

Internet ads are horrible: They waste your time, and the industry behind them is actively making the internet a worse place. Payouts are so small that the only way to survive is to turn your site into an ad-filled hellhole with no real substance. (see: [Modern social media](app://obsidian.md/misc/starting_a_blog/))

_If you want to support your favorite authors: send them money_. A dollar helps more than they will ever make of the ads you watch.

_However, most people view advertising as an in­herent part of the inter­net exper­ience_, so I added this message to my site:

No adblocker detected. Consider using an extension like [uBlock Origin](https://github.com/uBlockOrigin/uBOL-home/blob/main/README.md) to save time and bandwidth. Click here to close.

It's shown off to the side, and never covers content: it won't be shown if there isn't enough space. The close button actually works and it stays closed.

_The specific recommendation is important_ because a lot of people have only heard of adblockers from ads. Commercial adblockers range from sketchy to outright scams: If they are paying to be promoted, they must expect to make money from users.

## Technical details:

The page itself contains a placeholder element and tries to load a script:

```
<!-- Rest of the page goes here -->

<script defer src="/nativeads.js"></script>

<div
  id="ad-note-hidden"
  class="ftf-dma-note ad native-ad native-ad-1 ytd-j yxd-j yxd-jd aff-content-col aff-inner-col aff-item-list ark-ad-message inplayer-ad inplayer_banners in_stream_banner trafficjunky-float-right dbanner preroll-blocker happy-inside-player blocker-notice blocker-overlay exo-horizontal ave-pl bottom-hor-block brs-block advboxemb wgAdBlockMessage glx-watermark-container overlay-advertising-new header-menu-bottom-ads rkads mdp-deblocker-wrapper amp-ad-inner imggif bloc-pub bloc-pub2 hor_banner aan_fake aan_fake__video-units rps_player_ads fints-block__row full-ave-pl full-bns-block vertbars video-brs player-bns-block wps-player__happy-inside gallery-bns-bl stream-item-widget adsbyrunactive happy-under-player adde_modal_detector adde_modal-overlay ninja-recommend-block aoa_overlay message"
>
  <p id="ad-note-content-wrapper">
  </p>
</div>
```

The script adds the actual message into the document:

```
// /nativeads.js

function hide() {
        document.getElementById("ad-note").id = 'ad-note-hidden';
	document.getElementById("ad-note-content-wrapper").innerHTML = "";
        document.cookie = "notice-shown=true;path=/";
}

if (!document.cookie.includes("notice-shown")) {
	document.getElementById("ad-note-hidden").id = 'ad-note';
	document.getElementById("ad-note-content-wrapper").innerHTML = "No adblocker detected. " + 
	"Consider using an extension like <a href=https://github.com/uBlockOrigin/uBOL-home/blob/main/README.md>uBlock Origin</a> to save time and bandwidth." +
	 " <u onclick=hide()>Click here to close.</u>";
}
```

Finally, there's a bit of CSS to make it look nice:

```
#ad-note-hidden, #ad-note {
        display: none;
}

@media (min-height: 30em) { @media (min-width: 75em) {
        #ad-note {
                display: block;
                position: fixed;
                bottom: 1em;
                right: 1em;
                width: 14em;
                border: white 1px solid;
                background-color: #111111;
                padding: 1em;
        }
        #ad-note-content-wrapper {
                margin-top: 0em;
                margin-bottom: 0em;
        }
}}
```

_The message won't be shown_ if an adblocker removes the <div> element with or blocks the network request for "nativeads.js". The network component ensures that it doesn't miss blockers like uBlock Origin Lite which only filter network requests.

_There's no way to detect DNS based blocking short of loading an actual ad_. Instead, I made the message unobtrusive and easy to close.

Thanks to [Stefan Bohacek](https://stefanbohacek.com/project/detect-missing-adblocker-wordpress-plugin/)  for the original idea.