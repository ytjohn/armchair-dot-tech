---

title: All The Changes
author: ytjohn
date: 2017-05-07 13:52:40

layout: post
tags:
- meta
- website
slug: all-the-changes

---
## tl;dr

Here's a quick summary of changes that have taken place:

* A new site [Nifty Noodle People] has been launched
* [BCARS] has been moved from [mezzanine] to [wordpress] and re-organized
* A new community forum site has been launched: https://community.yourtech.us/
* Comments for BCARS and YourTech use the community site now.
* YourTech.us and [YourTech Community] now live on a dedicated VPS.
* YourTech.us is now using [WordPress], though all content is still generated in [Markdown].
* Soon, other sites will migrate as well.


## Launching A New Site

Over the course of this last week (really a good bit of this year) I've been doing a lot more web work. In February, I launched [Nifty Noodle People], an event website to promote [BCARS]'s rebranded [Skills Night 2.0].  After trying many, many different systems, I settled on [WordPress]. WordPress is something I moved away from back in 2012. However, for a single purpose site, WordPress really impressed me. It impressed me so much, that I decided that I should redo BCARS site under wordpress as well. I had been using [Mezzanine], Django-based CMS to manage their site and mine. But Mezzanine has been showing its age and often causing more problems than it's worth when it comes to doing updates or adding things like an event calendar. 

## BCARS Changes

I setup a BCARS development wordpress site and started importing content into it. I spent a lot of time looking at different calendars. For Nifty Noodles, I had used [The Events Calendar], and it's a really nice calendaring system. But when I was trying to utilize it for BCARS, I ended up not liking the formatting options. I went back into research mode and ultimately settled on [time.ly]. I even picked up their Core+ package which lets me re-use vendors and organizers. This let me add in recurring events like meetings and weekly nets, and it allows people viewing the site to filter between regular and featured events (like a VE session). 

As I was secretly working on this, it was brought up at a club meeting that the club would like to see a way to buy and sell gear on the site. So I added [bbPress] forum to the development site. Then I launched it silently on April 24th. It has gotten pretty solid reviews from people visiting it. 

## Server Move

As I was doing all this work, I observed that my Dreamhost VPS was prone to crashing. I also made an alarming discovery that I was paying a lot more each year than I had remembered. Also, I often get issues with it running out of memory and getting rebooted. I decided it was time to go searching. I had stuck with Dreamhost because of their nice control panel. They made it easy to spin up new sites, sub-domains, and "unlimited" everything. But it's time to move on.

I looked at web hosts, then I looked at plain VPSes. I discovered that [OVH] had some really good pricing on SSD VPSes. A couple years ago, I would have bulked at "wasting time" managing a server in order to do something simple like pushing web content. But my skills with config management have come a long way over the last 5 years. I decided I would use [Ansible] to manage the VPS and use all the myriad of roles out there to do so. I'll hopefully write more on that later. But in short, I've got roles installing mongodb, mysql, [nginx], letsencrypt, and managing users. I couldn't find a suitable role to manage nginx vhosts, especially in a way to start with a vhost on port 80, and not clobber the information [letsencrypt] puts in when it acquires a certificate. I hope to make a role that maintains http and https config separately, only putting in the https configuration if the certificate files exist.

But I digress. 

## Community Forums

During all this, I have been giving lots of thought to moving YourTech to wordpress as well. It's a bit more challenging because I write all my notes in [Markdown], which I then convert into posts. I started [markdown blogging] in 2012, and have shifted platforms several times since, most recently on [Mezzanine]. I was also thinking of better ways to engage the audiences of YourTech, BCARS, and Nifty Noodles. I had come across [this article](https://news.ycombinator.com/item?id=14170041) about replacing Disqus (which I had used) with Github comments.  While I liked the idea, I knew it wouldn't work for my goals. I kept coming back to forum software. I found three modern variations [Discourse], [NodeBB], and [Flarum]. Of the three, I like Flarum the best. Unfortunately, Flarum is still under heavy development and not recommended for production use. The authors can't yet guarantee that you'll be able to preserve data through upgrades. They want the flexibility to make changes to the database structure as they develop features. So I went to the next best, which is [NodeBB]. 

NodeBB has a [blog comments plugin] that allows you to use a NodeBB to handle comments on your blog. The pieces all started coming together. I installed NodebB on my VPS as https://community.yourtech.us/. I changed the links on BCARS forums to point to this new community site, and integrated comments for BCARS. 

## YourTech Move

This weekend, I decided to pull the plug on YourTech.us and migrate it simultaneously into wordpress and into the new server. I new this would cause downtime, but since my blog is not commercian, and not exactly in the [Alexa Top 500], I wasn't too concerned. If anyone did notice downtime between the 5th and 7th, let me know below.

The move was not without hitches. I did have a markdown copy of all my posts, but I had to add [yaml frontmatter] to the top of them for [github wordpress sync] to work. Then I discovered that the plugin ignores my date and just makes all my posts match the time of the sync. Also, using the same repository I had been using in development caused issues as well. But eventually, I got all my posts imported with their original post date.

What I didn't import was my resume and personal history. My contact page I did import, but it is rather out of date, so I feel I should update it soon. I want to rethink what I have on all three pages and how I present them, so that's a future project.
 
Finally, I discarded the handful of disqus comments I had and integrated the comment system with [YourTech Community].

## Future Plans

 * I still need to migrate [BCARS], [Nifty Noodle People], and other sites away from Dreamhost. But I hope those moves will be pretty painless since it will be direct copy and DNS change. 
 * I made YourTech.us look similar to how it did before the move, but I am not sure I'll keep that look going forward. 
 * Once Flarum becomes more production like and they build a NodeBB importer (and comment integration), I'll quite possibly move to that.
 * Ultimately, I hope these changes will motivate me to write more frequently, now that I can easily post from my [phone](https://play.google.com/store/apps/details?id=org.wordpress.android&hl=en) or web.


---

[YourTech Community]: https://community.yourtech.us/
[Nifty Noodle People]: https://www.niftynoodlepeople.com/
[BCARS]: https://www.bcars.org/
[Markdown]: https://daringfireball.net/projects/markdown/syntax
[markdown blogging]: https://www.yourtech.us/2012/markdown-blogging
[Skills Night 2.0]: http://www.bcars.org/skillsnight/
[WordPress]: https://wordpress.org/
[Mezzanine]: http://mezzanine.jupo.org/
[The Events Calendar]: https://theeventscalendar.com/
[time.ly]: https://time.ly/
[bbPress]: https://bbpress.org/
[OVh]: https://ovh.com/
[Ansible]: https://www.ansible.com/
[letsencrypt]: http://letsencrypt.readthedocs.io/en/latest/
[nginx]: https://www.nginx.com/
[Discourse]: http://www.discourse.org/
[NodeBB]: https://nodebb.org/
[Flarum]: http://flarum.org/
[blog comments plugin]: https://github.com/revir/nodebb-plugin-blog-comments2#readme
[Alexa Top 500]: http://www.alexa.com/topsites
[github wordpress sync]: https://github.com/mAAdhaTTah/wordpress-github-sync
[yaml frontmatter]: https://jekyllrb.com/docs/frontmatter/
