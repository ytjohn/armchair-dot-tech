---

title: Expanding a raw image file
author: ytjohn
date: 2011-01-13 17:58:09

layout: post

slug: expanding-a-raw-image-file

---
On one of my customer's devices, the backup software requires that the
backups go to a separate partition (or drive).  However, the customer
only has one raid array and the bulk of the space is in /home.   To work
around this limitation, I created a raw image file called backup.img,
which gets mounted as /backup.   After the software performs its local
backup, I use duplicity to backup /backup remotely to a backup server at
my location (with encryption).

Today I got an alert that /backup was running low on space.  It was an
80GB image and 61GB was in use, leaving only 15GB free.  Now, this
amount of free space should last quite a while.  However, the software
(cPanel)  has a known issue for years that the 80% limit is hardcoded
into the program.  I can change this, but every time cPanel updates, it
overwrites that change.

So to be proactive, I decided to go ahead and increase the image size.

In order to increase the size of an image, you simply unmount your raw
image and use the dd command.

<blockquote>
# Increase by ~20GB<br />
dd if=/dev/zero bs=1M count=20480 &gt;&gt; backup.img<br />
# 20,480 is 20,480 MB or ~20GB  </br></br>

# check the filesystem<br />
/sbin/e2fsck -f backup.img<br />
# resize the filesystem<br />
/sbin/resize2fs backup.img<br />
# check the filesystem again<br />
e2fsck -f backup.img               </br></br></br></br></br>
</blockquote>
