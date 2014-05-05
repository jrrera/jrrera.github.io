---
layout: post
title: "Preventing MySQL from Randomly Stopping on Ubuntu 12.04"
description: ""
category: 
tags: ["Bash", "Ubuntu 12.04", "Digital Ocean", "MySQL", "WordPress"]
---
{% include JB/setup %}

I recently migrated my WordPress blog from GoDaddy to [Digital Ocean](http://www.digitalocean.com). Digital Ocean is a fantastic way to set up your own server at low cost, but it does require you to do a lot of the maintenance work yourself. 

I was able to follow Digital Ocean's [guide](https://www.digitalocean.com/community/articles/how-to-migrate-wordpress-from-shared-hosting-to-a-cloud-server-with-zero-downtime) for migrating my previously existing WordPress blog, [LivingForImprovement.com](http://www.livingforimprovement.com), from GoDaddy to Digital Ocean, but one pain point I couldn't resolve was MySQL randomly terminating itself due to low memory on my Digital Ocean droplet. Without MySQL running, blog posts couldn't be fetched when people visited my blog. No bueno :(.

![My blog isn't happy when MySQL shuts off](/assets/img/mysql_off.png)

I made sure a [swap file](https://www.digitalocean.com/community/articles/how-to-add-swap-on-ubuntu-12-04) was active on my droplet, but every week or so, MySQL would still terminate itself. Working with memcache also seemed like a viable option, but it looked a little bit over my head. While I do plan to learn memcache eventually, I decided to get my toes wet in the world of shell scripting to help manage the problem until I do.

It's nothing special, but here's what I did:

### Step 1 - Create a simple shell script ###

	sudo nano /usr/local/sbin/mysql_check.sh

Then, I wrote the following: 

	#!/bin/bash
	if [[ ! "$(service mysql status)" =~ "start/running" ]]
	then
	    service mysql start
	    # echo "Restarting MySql" # Uncomment for debugging
	else
	    # echo "Looks like MySql is running. No action taken" # Uncomment for debugging
	fi

Finally, make sure your script is executable:

	sudo chmod +x /usr/local/sbin/mysql_check.sh

### Step 2 - Set up a cron job to check MySQL's status every minute ###

Until I sit down and learn the ins and outs of memcache, I'd prefer that this cron check every minute to make sure MySQL is running. I had to do this in root's crontab, since the commands I'd need the script to run require root.

	sudo crontab -e

And within your root crontab:

	PATH=/usr/sbin:/usr/bin:/sbin:/bin
	*/1 * * * * /usr/local/sbin/mysql_check.sh >> /home/jon/cronlog.log 2>&1

I had to add the PATH variable to the script to allow crontab to run the commands in my shell script, such as `service mysql start` as per [this article's discussion](http://ubuntuforums.org/showthread.php?t=2022708).

Just for debugging purposes, I'm routing any output to a log file in my home directory, just to initially measure how often the cron needs to restart MySQL. If you're not curious about this, you can change the above to:

	PATH=/usr/sbin:/usr/bin:/sbin:/bin
	*/1 * * * * /usr/local/sbin/mysql_check.sh

### Step 3 - Improvement ###

I realize that this isn't the most elegant solution, but it works for now, until I have the bandwidth to learn to use memcache like a boss. Any suggestions / feedback? Leave it in the comments section below. :)

