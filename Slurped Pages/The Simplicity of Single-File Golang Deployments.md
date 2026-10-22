---
link: https://www.amazingcto.com/simplicity-of-golang-systemd-deployments/
byline: Stephan Schmidt
site: Amazing CTO
date: 2025-08-01T00:00
excerpt: Deploy Go apps as single static binaries with systemd. No Docker, no
  npm, zero-downtime via socket activation. Modern complexity is often
  unnecessary.
twitter: https://twitter.com/@KingOfCoders
slurped: 2026-10-20T21:11
title: The Simplicity of Single-File Golang Deployments
---

I’ve gained some experience with deployments over the last three decades Starting without a VCS, copying files by zip to servers, sharing files by zip and manually tracking changes.

Fast-forward some years and we deployed large Java applications with build scripts manually with a release train and release notes. Each releases a huge event with coordinating many people in the company. Fast-forward and we were automatically deploying Scala applications from CI bundled in Docker in the startup of my wife.

Last forward and I have deployed a Golang application to a cloud server. The result of the build is one executable. The Go web application had all files like configurations (no credentials), static css and html templates embedded with embedfs (and proxied through a CDN). So no jar or ear files, no npm or pip and no Docker. Just [one binary](app://obsidian.md/launch-a-project-in-one-day/) copied to the server, started and monitored with Systemd. Systemd also restarts the app daily to make sure it works properly long term. As Go starts up tremendously fast, and as [Systemd is keeping the TCP connections](app://obsidian.md/self-healing-for-cto-solo-founder/), there is no impact to users.

The build pipeline is simple; deployments are syncing a binary file to a server, with Systemd holding connections and restarting the new binary. Systemd is also used to degrade privileges for the running binary to make it more secure.

The binary checks if it has changed and restarts

```
go func() {
  watcher, err := fsnotify.NewWatcher()
	if err != nil {
	  logger.Error().Err(err)
	}
	defer watcher.Close()
	// Check for our binary
	_ = watcher.Add(os.Args[0])
	<-watcher.Events
	loh.Info().Str("File", os.Args[0]).Msg("executable changed, exiting...")
	// shutdown
	shutdown()
}()
```

The binary checks itself if it’s working, otherwise exits

```
for {
  client := &http.Client{}
  req, _ := http.NewRequest("GET", "http://127.0.0.1:4000", nil)
	// if we don't set the host to www
	// we'll get a REDIRECT in production
	req.Host = "www.example.com"
	_, err := client.Do(req)
	if err == nil {
	   // notify Systemd we're ok
	   _, _ = daemon.SdNotify(false, daemon.SdNotifyWatchdog)
		 logger.Info().Msg("Health check OK.")
	} else {
	  logger.Error().Msg("No response - ERROR - going to restart.")
	}
	time.Sleep(interval)
}
```

Standing here it looks like Docker was invented to manage dependencies for Python, Javascript and Java. It looks strange from a platform that deploys as one single binary.

This one binary feels pure and simple, compared to large NPM or JAR deployments inside a Docker container. It feels manageable and [not overwhelmed by complexity](app://obsidian.md/the-developer-productivity-yoyo-effect/). [Radical Simplicity](https://www.radicalsimpli.city/)

When sharing this on Mastodon, others chimed in with “yeah, like copy a single file and that’s the command to run”, “I see you are slowly joining the fan club :)” and “yeah, so nice and simple! I still mostly package in containers for ease of deployment to the cloud, but it’s a pretty thin wrapper.”

_So I’m not alone with my joy and happiness._

**About me:** Hey, I'm Stephan, I help [CTOs with Coaching](app://obsidian.md/cto-coaching-from-experienced-cto/), with 40+ years of software development and 25+ years of engineering management experience. I've coached and mentored 100+ CTOs and founders in more than 1200 sparring and coaching sessions. I've founded 3 startups. 1 nice exit. I help CTOs and engineering leaders grow, scale their teams, gain clarity, lead with confidence and navigate the challenges of fast-growing companies.

Most of the CTOs I coach didn't know CTO coaching was a thing until they were already drowning. It is a thing - here's [what it is](app://obsidian.md/what-is-cto-coaching/).