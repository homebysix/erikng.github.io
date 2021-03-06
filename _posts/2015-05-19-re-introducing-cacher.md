---
layout: post
title: "Re-Introducing Cacher"
description: "A helpful tool for Apple Caching Servers"
tags: [Cacher, Apple, Caching, Server]
comments: true
---

Several months ago I was tasked to prove that our brand new Caching Server was actually doing it's job. I had already setup [CacheWarmer](http://blog.fraserhess.com/2014/12/introducing-cachewarmer.html) with an hourly custom script/cronjob, so I knew when new iOS builds would come out, we wouldn't be crippled.

SNMP could technically be an option but the Caching Service relies on random high ports. We could make assumptions, but that still wouldn't be good enough to justify _why_ we had this installed in out data center. Anecdotally, I knew it was working: Our users noticed app store improvements, I noticed app store improvements, but our executives wanted metrics.

Apple's Server information is very minimal and that's being generous.
![Cacher Example](/images/2015/05/SoInformative.png)
E-mailing screenshots filled with squiggly line isn't good enough. Maybe the logs can help?

Reading the [Advanced Caching Documentation page](http://help.apple.com/serverapp/mac/4.0/#/apd5E1AD52E-012B-4A41-8F21-8E9EDA56583A), I went ahead and enabled "LogClientIdentity" by running the following command.

```bash
sudo serveradmin settings caching:LogClientIdentity = 1
```

After looking at the generated log files, I knew I was on to something. The logs gave me everything I needed, but with roughly 30,000 machines possibly hitting our caching server, the logs were enormous. I needed to figure out something.

## [Enter Cacher](https://github.com/erikng/Cacher)

Cacher is a bash script that if configured as a daily cronjob, will e-mail you daily statistics on your devices.

Cacher requires the following:

- OS X Yosemite 10.10.3
- Server 4.1 with [Alerts Enabled](http://krypted.com/mac-os-x/configure-alerts-in-os-x-yosemite-server/)

Currently there are two branches: Master and HTML:

1. Master -> Outputs a standard message to Server 4's alert mechanism.
2. HTML -> Adds some html to the outputted message to trick Server 4's alert mechanism into sending indented text.

Here is an example e-mail alert from the HTML _branch_.
![Cacher Example](/images/2015/05/Cacher_Example.png)

#### Installation
Installation is pretty simple:

1. Download Cacher from either branch (I personally use the HTML branch).
2. Place it somewhere on your OS X Caching Server.
3. Setup a cronjob/LaunchD

#### Example
Create a folder and download it

```bash
mkdir -p /Users/Shared/Cacher
curl "https://raw.githubusercontent.com/erikng/Cacher/HTML/Cacher" -o "/Users/Shared/Cacher/Cacher"
```

Make sure it can be executed

```bash
chmod a+x /Users/Shared/Cacher/Cacher
```

Create a root cronjob. This is required for the alert mechanism to work. Alternatively you could create a LaunchDaemon as well, but I find this easier.

```bash
sudo env EDITOR=nano crontab -e
```

Nano will open. Put in your preferred e-mail time alert. I personally have an e-mail sent at 6:30 AM every day.

```bash
30      6       *       *       *       /Users/Shared/Cacher/Cacher
```

After entering this into nano, use hit CTRL+X and save it. If successful, crontab will give you an installation status. If you need more information on setting up a different time, please see [this maclife article](http://www.maclife.com/article/columns/terminal_101_creating_cron_jobs)

Now sit back, wait until tomorrow to see your new statistics and wonder why Apple doesn't have something like already.

## Table Of Contents
* Do not remove this line (it will not be displayed)
{:toc}