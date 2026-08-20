

[from Mixologic #13](https://www.drupal.org/project/drupal/issues/2538090)

I've spent a fair amount of time in meetings with Jordi (the creator of composer) when he was helping advise us on building packages.drupal.org , and the issue at hand is not really a communications, engineering, or design problem with composer.

Composer is slow and resource heavy because it has to be. The job that it automates is fourfold, and its complicated. Like, the hardest problem in all of computer science complicated.

1. It needs to gather all of the metadata about all the components of your project (core, modules, php libraries, themes etc), including new pieces you are asking to install. - This gets data from your local filesystem (composer.json/composer.lock), gathering data from packagist.org, gathering data from packages.drupal.org, and gathering data from any other repositories you may have told it to use (maybe a local repo for custom modules etc). The only pieces of data that matters in all of this are what each component depends on, and which versions of those components will work. This is pretty much the same thing as a database query to get this data, except composer is hitting api's and caching things etc.

2. Once it has all of that data, it has to run an SAT solver on it. This is the hard part. In a world where a "module" had all of the functionality you needed, and everything was encapsulated inside and you didn't have to worry about overlap between modules, this used to be easy. But in a more modern world, nobody wants to rewrite functionality thats already written, and things become more and more overlappy. One module might use the aws sdk to stash files on S3 at aws, and a different module might want that same sdk to automatically run your email through amazons email service. Anyhow, point being is that once you have every single component, and all of their dependencies, and dependencies of dependencies and you have to figure out if there is a single set of versions that makes everyone happy - well, thats another example of a classic comp sci problem called the traveling salesman. Nobody has ever solved it, so it's hard and slow. Sometimes its easy and fast, but you cant know in advance. This is what takes up all the memory, attempts at making it faster.

3. Once if finally *does* know exactly which versions are needed, then It needs to download all of the files you are trying to add - the module, its other composer deps, those composer deps deps ad infinitum until you actually *have* everything. This might include upgrading some things as well. Downloading files from the internet can be slow, so again, no guarantee that can happen before php times out.

4. Finally, once everything is figured out, and in place, then composer generates one last PHP file, the Autoloader, so that we dont have to pull in every scrap of php into memory when we're running requests. It can grab them as they are asked for. And the autoloader essentially hands the application a mapping of files on disk to classnames being used. So generating a file, and writing it to a place that your webserver can execute, *from* your webserver itself is generally considered about as secure as a balsawood box full of bananas in a room full of chimpanzees.

So, all that being said, *none* of that is stuff that everybody should have to care about, and all of it implementation details. Which is why this:

> simply create a composer style dependency manager specifically written for Drupal but still kind of acts like composer and still uses packagist?

Other than the "simply" part, that is essentially what we are talking about doing, except it would use composer under the hood. An electron app is really just a browser in disguise (its what wordpress's admin tool is, slack is written in electron, and others: [http://electron.atom.io/](http://electron.atom.io/)). So the idea would be ** heres a tool that lets you get everything you need for your site in one place, and then it hands you a file or files to upload to your server **

My main suggestion in this is that composer isn't the only problem we should solve if we're going to do that, we should look at the same dependency problem that the front end faces (js assets, webpack, bower, npm, etc) and solve it all at once.

Anyhow, your pain is strongly justified, and not isolated - Im hearing a need for something like this from a lot of corners of the drupalsphere. Drupal 7->8 made an engineering tradeoff for better, more battle tested software engineering practices and less likely to break code, and in the process sacrificed some of the ease of site construction. For the people building sites with plenty of resources, that was fine. They have programmers/developers for whom things like composer were familiar, but for anybody without the resources to follow the same "best practices", that was a bummer.

But now we have an opportunity to make something great, that can address these issues, and put a layer of user experience into the site building process that is definitely lacking from where we are now.

## from mixologic #15

[#15](https://www.drupal.org/node/2845379)

There's a couple of constraints that we want to consider when designing a system that will allow us to abstract away the complexities of using composer:

Composer is designed to be a build time/development tool, and is not intended to exist or be available in production systems. What that means in practice is that the normal constraints of "code outside the firewall and open to the internet" are not baked into composer, and thus it could be considered a security risk to have it available on a production system that interfaces with threats. Thats not to say it cannot be on live sites, just that upstream, the developers are not likely to focus on making it hard to compromise. Even if we were to audit it now and find that its fine, that doesn't mean that the next version wont have some obvious remote code execution vulnerability baked into it.

The second aspect of composer is that it is designed to be executed in a command line php environment - which means it is expecting to have the ability to consume considerable amounts of memory, and take significantly longer execution time than most webserver php environments are geared for. Drupal 8's 64MB minimum requirement would probably need to bump to 256MB or higher, and we might need as much as 180 seconds worth of timeout for php. Or more.

I bring up these two points to underscore the difficulty we may encounter in attempting to bundle composer within the drupal application itself. There are likely ways we can cordon it off for security purposes, and ways we can ensure its only running on a cron or something similar. Another alternative is to construct a site management tool that is separate from your web app, much like wordpress's desktop administration application ([https://developer.wordpress.com/calypso/](https://developer.wordpress.com/calypso/)).

Composer isn't the only sitebuilding barrier, and we ought to address the "Constructing, Extending, Updating, and Maintaining" the codebase your website needs to run in a holistic manner - so that we can handle any other "Buildtime/Development tools"- i.e. we can add build tools like NPM or bower to solve the 'gather front end assets too' so as to provide a seamless, delightful experience for drupal users who do not fall into the classification of "working on enterprise sites for enterprise clients"

I also strongly believe that we should start by designing the ideal user experience we would like to see in a tool like this, and aim high enough to knock it out of the park.

## inevitable evoltion

Here is my perspective on the situation/goals/solution:

a) To run a simple Drupal 8 site, as things stand now, it is definitely possible without having Composer installed or even knowing about it. You can download the Drupal Core tarball, unzip it, and go. Same when you update Core.
ok a
b) For the vast majority of Drupal 8 contrib modules, you can also operate without Composer: download the tarball, unzip it, and go. Same when you update the contrib module.
ok b
c) We have a GUI in Core (in Update Manager module) that allows users to download/unzip files for contrib modules in the GUI, when first installing them or when updating them. IMO this is dangerous, and I personally never set this up or use it, but it is already part of Core and has been since the D7 days. Currently this only works for the modules that work via (b) in Drupal 8.
reponse from mixologic

There are many modules that rely on third party dependencies, and that number will only grow as more and more essential functionality becomes based off of php libraries that contain solutions that the drupal community should not be re-inventing. Right now, it covers things like 3rd party integration API's, and we are gradually moving to a point where you will not be able to build an effective, functional drupal site without using modules that rely on composer. There are already some modules that are commonly used that fall into this category, such as the address, devel, backup_migrate, colorbox. Right now about 10% of all d8 modules have a 3rd party dependency.

that quote perfectly summarizes why we really need to do this and why we shouldnt harbor notions like "everybody should be using the command line and following 'best practices' of professional engineering workflows"

There is deep concern about composer as the primary means of installing Drupal from many audiences, strong enough that people pick other solutions instead of Drupal. Composer is the bee's knees -- for PHP devs and devops people like you and me who can support it -- but for a significant chunk of our current market, it's actually a barrier. Not because they're not "trained". This is offtopic here, but it is a big risk for Drupal right now and finding a way to support both audiences is a problem that's going to take a lot of resources to solve. 

## other
When you type composer require package/name, you implicitly trust both packagist.org and the package owner on packagist.org, who is unverifiable and not vetted. This default chain of trust is not made obvious to many users, and the package upstream may be essentially uninvolved. The circumstances in which packagist.org makes package changes are not documented, the changes are not signed, and these changes are not auditable. Package owners on packagist.org are not verifiable, changes they make are not signed, and their changes are not auditable. There is no chain of trust between the package upstream and packagist.org. None of this is very clear to the average user.

You can find more details on a specific case of this at: https://github.com/phacility/xhprof/pull/40

We may support Composer in the future, but this upstream's attitudes toward security are currently very different from Composer's attitudes toward security.

We understand that a lot of users don't care about this, and Composer works well and is easy to use, but this is important to us.

## composer security

When you type composer require package/name, you implicitly trust both packagist.org and the package owner on packagist.org, who is unverifiable and not vetted. This default chain of trust is not made obvious to many users, and the package upstream may be essentially uninvolved. The circumstances in which packagist.org makes package changes are not documented, the changes are not signed, and these changes are not auditable. Package owners on packagist.org are not verifiable, changes they make are not signed, and their changes are not auditable. There is no chain of trust between the package upstream and packagist.org. None of this is very clear to the average user.
## API Composer

[API Composer](https://packagist.org/apidoc#best-practices)