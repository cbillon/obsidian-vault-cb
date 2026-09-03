---
tags:
  - moodle
  - plugin
---
a Moodle plugin is legally considered a **derivative work** under the terms of the GNU General Public License (GPL) and **must be licensed under the GNU GPL v3** (or a compatible license) if it is distributed. [[1](https://moodle.org/mod/forum/discuss.php?d=126755), [2](https://moodledev.io/general/community/plugincontribution/checklist)]

Why is it a Derivative Work?

- **Core Interaction:** Moodle plugins make direct function calls, extend classes, and share data structures with the Moodle core. [[1](https://moodle.org/mod/forum/discuss.php?d=275906)]

- **GPL Interpretation:** According to the Free Software Foundation (FSF) and Moodle’s official developer policy, any code that intimately links or interfaces with GPL core software to run as a single combined work inherits the GPL requirement. [[1](https://moodledev.io/general/community/plugincontribution/checklist), [2](https://moodle.org/mod/forum/discuss.php?d=126755)]

- **Official Requirement:** Moodle's [Plugin contribution checklist](https://moodledev.io/general/community/plugincontribution/checklist) explicitly states that all files implementing the interface between Moodle core and the plugin must be licensed under the GNU GPL v3 or later. Any third-party libraries bundled inside the plugin must also use a GPL-compatible license. [[1](https://moodledev.io/general/community/plugincontribution/checklist)]

Distribution vs. Internal Use

- **Distribution:** The GPL requirements strictly apply when you **distribute** or sell/send the plugin code to others.

- **Private Use:** If you write a custom plugin and run it solely on your own private server without distributing the source code to outside users or clients, the GPL distribution clause is not legally triggered. However, publishing it or sharing it publicly means it must be open-source under the GPL. [[1](https://moodle.org/mod/forum/discuss.php?d=135896), [2](https://moodle.org/mod/forum/discuss.php?d=364803), [3](https://moodle.org/mod/forum/discuss.php?d=458235)]

** _Legally_ the source must be disclosed to the people you are distributing it to, not to everyone in the world. **

If you're planning to release a plugin, tell me:

- Is it **open-source or commercial**?
- Are you planning to list it on the **Moodle Marketplace/Directory**?


**Structuring a dual-pricing model (free core + paid services) safely under the GNU General Public License (GPL) requires decoupling your monetization from the software code itself.** Because the GPL grants users the absolute freedom to run, copy, modify, and redistribute software code, you cannot use the license to lock features inside a combined codebase. [[1](https://www.youtube.com/watch?v=xTbZuHIg11I&t=15), [2](https://chat2db.ai/resources/blog/understanding-the-gpl-license), [3](https://softwareengineering.stackexchange.com/questions/161762/confused-on-selling-gpl-how-is-it-consistent-with-this-section)]

To build a legally compliant and profitable business around the GPL, you must shift your financial value proposition to **services, hosting, access mechanisms, and non-software assets**. [[1](https://sbomify.com/2025/12/22/gpl-license-guide/), [2](https://www.termsfeed.com/blog/dual-license-open-source-commercial/)]

---

1. Monetize via Managed Hosting and SaaS (The WordPress/Ghost Model)

The safest and most common way to monetize GPL software is to offer the core code for free while charging for a fully managed environment. [[1](https://www.termsfeed.com/blog/dual-license-open-source-commercial/), [2](https://sbomify.com/2025/12/22/gpl-license-guide/)]

- **How it works:** The customer is not paying for the code; they are paying for infrastructure, uptime, backups, maintenance, and zero-configuration setup. [[1](https://www.termsfeed.com/blog/dual-licensing-vs-open-core/), [2](https://www.termsfeed.com/blog/dual-license-open-source-commercial/)]

- **GPL Safety:** The GPL’s copyleft triggers _only_ upon distribution. Running GPL software on your own servers to serve users over the web does not require you to share your custom backend infrastructure code (unless you use the stricter **AGPL**, which treats network access as distribution). [[1](https://www.opensourcealternatives.to/blog/open-source-license-guide), [2](https://sbomify.com/2025/12/22/gpl-license-guide/)]

2. Monitizer via API-Driven Cloud Services (The Hybrid Model)

Keep the entire core local application 100% free and open-source under the GPL, but offload high-compute, complex, or data-heavy features to an external web API that you control. [[1](https://www.youtube.com/watch?v=xTbZuHIg11I&t=15)]

- **How it works:** A security or SEO plugin might be entirely GPL. However, the data audits, image-compression servers, or AI processing happen on your private cloud servers.

- **GPL Safety:** Anyone can fork your free local client code, but they cannot legally force access to your private, paid API cloud servers without an authorized subscription key. [[1](https://freemius.com/blog/selling-wordpress-plugins-gpl/)]

3. Monetize Support, SLAs, and Maintenance (The Enterprise Model)

You distribute the full, unrestricted software under the GPL for free, but restrict peace-of-mind resources to paying clients. [[1](https://www.termsfeed.com/blog/dual-licensing-vs-open-core/), [2](https://www.termsfeed.com/blog/dual-license-open-source-commercial/)]

- **How it works:** Sell prioritized support tickets, custom integration consultation, and Service Level Agreements (SLAs) for production environments.

- **GPL Safety:** The software remains free. You are exclusively charging for human time, expertise, and operational guarantees. [, [2](https://sbomify.com/2025/12/22/gpl-license-guide/)]

4. Separate Codebases via the "Open Core" Model

If you want to sell a "Pro" version of software that integrates with your free GPL "Core" version, you must structure them as separate, independent packages. [, [2](https://www.termsfeed.com/blog/dual-license-open-source-commercial/)]

- **The Danger:** If your paid features are deeply integrated or modified directly inside the GPL core code, the GPL's "viral" property dictates that the entire combined program must become GPL. [[1](https://forum.image.sc/t/licensing-a-plugin-for-gpl-software-different-interpretations/92224), [2](https://www.youtube.com/watch?v=xTbZuHIg11I&t=15)]

- **The Safe Architecture:** Keep your proprietary extensions completely modular (e.g., separate plugins, themes, or standalone modules). Ensure they interact with the GPL core strictly via public APIs, hooks, or command-line interfaces rather than deep code-subclassing. [[1](https://forum.image.sc/t/licensing-a-plugin-for-gpl-software-different-interpretations/92224), [2](https://www.termsfeed.com/blog/dual-licensing-vs-open-core/)]

- _Note:_ In ecosystems like WordPress, even separate plugins are legally argued by the Free Software Foundation to be derivative works if they require the core to run. Therefore, many companies use a "100% GPL Open Core" model where they sell the Pro plugin under the GPL but gatekeep **automatic updates, marketplace delivery, and license keys** behind a paywall. [[1](https://freemius.com/blog/selling-wordpress-plugins-gpl/), [2](https://www.youtube.com/watch?v=xTbZuHIg11I&t=15)]

---

GPL Compliance Cheat Sheet for Commercialization

|Strategy|Legally Compliant under GPL?|Risk of "Troll" Redistribution|
|---|---|---|
|**Gating Automatic Updates**|**Yes.** You are charging for access to your repository delivery system, not the code itself.|Low (users can manually download, but miss out on seamless security updates).|
|**Charging for Technical Support**|**Yes.** Charging for labor and response-time guarantees is fully protected.|Zero (human time cannot be copied or redistributed).|
|**Enforcing a Paid API Subscription**|**Yes.** Connecting to an external, hosted cloud architecture is entirely legal.|Zero (keys can be revoked server-side).|
|**Restricting Code Redistribution**|**No.** Telling a paying user they cannot share your GPL plugin code violates the GPL.|High (legal nullification of your distribution terms).|

To tailor this architecture to your business, could you share:

- What **type of software** you are building (e.g., a desktop app, web plugin, SaaS)?
- Which **specific premium features** you want to keep behind the paid tier?