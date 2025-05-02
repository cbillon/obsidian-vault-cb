---
link: https://vsupalov.com/tools/watchtower/
site: vsupalov.com
date: 2018-06-21T00:00
excerpt: Watch for new image versions and restart start new containers.
slurped: 2024-07-11T15:30:53.195Z
title: "Tool Spotlight: Watchtower"
tags:
  - docker
  - tools
  - image
---

Watchtower helps you use the most recent Docker images.

## What is it for?

Basically, an automated way to do very simple continuous deployment.

Watchtower repeatedly asks the Docker registry whether one more more Docker tags point to a new image. If a newer image exists, Watchtower pulls it and restarts the corresponding containers.

You can use it for all containers on a machine, or only for some of them.

## You might be interested if…

You are running a containerized non-production environment which you want to keep updated.

Sometimes there’s a new image version with the same image name and tag.

Usually you’d need to remove the old image, pull the new one and restart the containers **by hand**.

Instead, you’d like that to happen **automatically**. A slight delay is okay. You don’t mind automatic unattended updates, and nobody will mind an accidental downtime.

## What you need to know

It’s …

- Free.
- Open source.
- Very simple to setup.
- Able to keep itself updated.

You can …

- Run it in a Docker container.
- Make sure oyur containers use new images.
- Run it for all containers on the machine or limit it to a few container names.
- Specify the interval at which Watchtower checks for image changes.
- Label single containers to exclude them from what Watchtowers does.
- Send out email and Slack notifications when there’re updates.

## Take a look

The GitHub project README goes into [more details](https://github.com/v2tec/watchtower).

## Try it

Simply run it on your local machine, and give it a spin. I wouldn’t suggest to use it in an important production environment, but it’s fine for a simple testing setup.

## Links

- [GitHub](https://github.com/v2tec/watchtower)
- [Docker Hub](https://hub.docker.com/r/centurylink/watchtower/)

## An alternative

Creating an automated deployment script for your containerized application is somewhat similar, but not quite. You’d need to trigger it somehow, or run it manually.

If you’re only working with images which you’re building yourself, you could create a deployment pipeline, which takes care of redeploying your application after pushing a new image to the repository.

Subscribe to my newsletter!  

You'll get notified via e-mail when new articles are published. I mostly write about Docker, Kubernetes, automation and building stuff on the web. Sometimes other topics sneak in as well.

Your e-mail address will be used to send out summary emails about new articles, at most weekly. You can unsubscribe from the newsletter at any time.

I agree that my personal data will be used to receive commercial e-mails, and I know that I can unsubscribe at any time.

Für den Versand unserer Newsletter nutzen wir rapidmail. Mit Ihrer Anmeldung stimmen Sie zu, dass die eingegebenen Daten an rapidmail übermittelt werden. Beachten Sie bitte auch die [AGB](https://www.rapidmail.de/agb) und [Datenschutzbestimmungen](https://www.rapidmail.de/datenschutz-kundenbereich) .

Thanks for your subscription!  

Please check your inbox, you should have received a confirmation email. Your subscription will only be valid once you confirm it.  

© 2024 vsupalov.com. All rights reserved.