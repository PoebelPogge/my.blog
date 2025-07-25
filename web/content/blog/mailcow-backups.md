---
title: "New big love! mailcow: dockerized… but Backups?"
date: 2025-06-26
draft: false
# weight: 1
# aliases: ["/first"]
tags: ["self hosted"]
author: "PoebelPogge"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
# description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
ShowCodeCopyButtons: true
UseHugoToc: true
#cover:
#    image: "<image path/url>" # image path/url
#    alt: "<alt text>" # alt text
#    caption: "<text>" # display caption under cover
#    relative: false # when using page bundles set this to true
#    hidden: true # only hide on current single page
#editPost:
#    URL: "https://github.com/<path_to_repo>/content"
#    Text: "Suggest Changes" # edit text
#    appendFilePath: true # to append file path to Edit link
---


Okay, I know, its been a while since I texted here some stuff, and yeah, you are right I’ve never been that active here. But honestly I’m really willing to be more active.  
It’s not that I’m not doing fancy IT stuff in my free time, but that the point!  
  
Since I’ve become a father, free time isn’t much left. And building a house at the same time steals you even more of it.   
But I won’t complain!   
  
Becoming a father leads to my topic of this post. Caring about my personal data was always a point I was focused on. Since there is someone else who I have to care for, not having full control over my emails was an even bigger thorn in my side.   
  
First I moved to Proton Mail, but a big bill over +100€ made me thinking again about this step. Not that Proton Mail is not worth it, but I wasn’t willing to pay that. I like to control things on my own (and you might to like that, too).   
  
Proton Mail made it easy to manage emails for your family! Its fully encrypted! Definitely huge advantages compared to providers like GMail. But still, not everything under your control.   
  
Hosting an own mailserver was always a dream I had, but also always a huge effort to do so. Keep things updated, care about security. Not landing on any black lists. Maybe you know what I’m talking about.   
But then, mailcow appeared on my sight. And honestly? I totally love it!

But this article should not be about how to setup mailcow, or how to configure your DNS settings, there are plenty articles online and, I think, they would all explain it better then I am able to. What I’d like to share with you is how my backup setup is built up.  
  
  
  
Something I was worried about when I used mailcow more frequently was: What would happen if my system crashes. And honestly? That scared me a lot. So i dug a little into how to backup mailcow. And of course they have a solution even for that problem!   
Running: `  
/opt/mailcow-dockerized/helper-scripts/backup_and_restore.sh backup all --delete-days 3`  
backups everything and it even deletes backups older than 3 days. It asks you where you want to store the backup and runs smoothly through. But thats the point!   
  
I do not want to store my backups on the same machine as mailcow is running on. So what have I done?

First I created a cron job, pretty obvious:`  
0 1 * * * MAILCOW_BACKUP_LOCATION=/tmp/mailcow-backups /opt/mailcow-dockerized/helper-scripts/backup_and_restore.sh backup all --delete-days 3 >> /var/log/mailcow-backup.log 2>&1`  
Setting the variable `MAILCOW_BACKUP_LOCATION` has the advantage, that the mailcow backup script does not ask you where to store the backup file. Redirecting the output into a log file, makes it possible to debug afterwards. 

But I don’t want to ready the logs every morning if the backup was made or not. So what I though was: _Hey, I do have an mailserver running, why not sending notification mails about the backup status?_ So, my weapon of choice was `mutt`. It is a tool to send mails via console. It is pretty easy to setup and even easier to use. So the result of my cron job was:`  
0 1 * * * MAILCOW_BACKUP_LOCATION=/tmp/mailcow-backups bash -c 'if /opt/mailcow-dockerized/helper-scripts/backup_and_restore.sh backup all --delete-days 3 >> /var/log/mailcow-backup.log 2>&1; then echo "Mailcow backup successful" | mutt -s "✅ Mailcow Backup successfully created!" foo@bar.com; else echo "Mailcow Backup failed" | mutt -s "🛑 Mailcow Backup failed!" foo@bar.com; fi'`

So now I know every morning, if my backup was created or not. But still, the backups were on the wrong machine.   
Luckily my Synology NAS supports `rsync`. What have I done? I created another scheduled task which runs 2 hours after the first one (I assume for now the backup will not take so long) using rsync through an ssh connection to my mailcow server to sync all files between the backup location of the server and the mentioned folder of my NAS.  
`rsync -avz -e "ssh -i /volume1/homes/user/.ssh/id_rsa -p 22" root@123.234.345.456:/tmp/mailcow-backups/** /volume1/MailcowBackup`   
And, of course, sending another mail about the status, luckily that come ootb by using the Task Scheduler function of Synology.  
  
For now, everything runs fine! I hope you find this article a bit useful and inspiring. Feel free to leave comments below, I’m pretty sure there are a lot of things which can be improved!
