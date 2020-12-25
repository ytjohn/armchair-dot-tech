---

title: bits and bobs for 2017
author: ytjohn
date: 2017-12-23 18:34:09

layout: post

slug: bits-and-bobs-for-2017

---
Abusing my blog to record a few things. This is kind of a year-end wrap-up before Christmas, or a pre-pre New Years Eve post. I am off work till the end of the year, so this is kind of a good week to reflect and prepare for the upcoming year.

I cover a couple different things in this post, find the section you want.

* Home Security Cameras
* Financial Tools (quicken, ledger, hledger, beancount)


# Security Cameras

tl;dr - I recommend [Q-See NVR](http://a.co/gdkgDtR), refurbished 16-channel for $250 with 2TB drive

I've got a few cameras around my house. Two of them are analog cameras on a cheap LA View dvr bought off of woot (I've replaced it twice). I've ran zoneminder to pull the RTSP stream from it. 

<a href="https://static.yourtech.us/yt/uploads/2017/12/media-20150308.png"><img src="https://static.yourtech.us/yt/uploads/2017/12/media-20150308-300x149.png" alt="" width="300" height="149" class="alignleft size-medium wp-image-535" /></a>

I also have some really cheap ESCAM 720p cameras. These things are amazingly cheap, and can be had for [under $40](http://www.dx.com/p/escam-qd300-onvif-p2p-cmos-3-6mm-lens-waterproof-network-ip-bullet-camera-w-4-ir-led-white-324862).  Zoneminder can pull these in as well.

The problem is that I keep finding zoneminder in some broken state. I also am not a fan of the "each frame is a file" apprach. I've started using [shinobi](https://shinobi.video/). I like that better, but feel limited. Also, several times a week, one of the camera feeds goes dark and I have to re-enable it. I got a Windows PC setup and tried out [iSpy](https://www.ispyconnect.com/). [GenisuVision](https://geniusvision.net/), and [Xeoma](http://felenasoft.com/xeoma/en/). None of them have really stood out as a great system.

I decided to try out a more expensive hardware NVR. First I tried an [Amcrest NVR](http://a.co/5bAcDI4), but it couldn't work with any of my existing cameras. Returned. I have a colleague that is a big fan of Q-See Analog DVRs, and the mobile app is pretty slick. I found a good deal on a [refurbished 16-channel NVR for $250 with a 2TB hard drive included](http://a.co/gdkgDtR). This was an instant success. It picked up my ESCAM right away and starts recording.

<p>
<a href="https://static.yourtech.us/yt/uploads/2017/12/qseebigscreen.jpg"><img src="https://static.yourtech.us/yt/uploads/2017/12/qseebigscreen-300x169.jpg" alt="" width="300" height="169" class="size-medium wp-image-538" /></a>

</p>

**The mobile app is a dream.** 
<p>
<a href="https://static.yourtech.us/yt/uploads/2017/12/Screenshot_2017-12-23-13-29-05.jpg"><img src="https://static.yourtech.us/yt/uploads/2017/12/Screenshot_2017-12-23-13-29-05-300x169.jpg" alt="" width="300" height="169" class="size-medium wp-image-539" /></a> 
<a href="https://static.yourtech.us/yt/uploads/2017/12/Screenshot_2017-12-23-13-25-03.jpg"><img src="https://static.yourtech.us/yt/uploads/2017/12/Screenshot_2017-12-23-13-25-03-300x169.jpg" alt="" width="300" height="169" class="size-medium wp-image-540" /></a>
</p>

I do get higher resolution images, but when scaled for mobile playback and on a screenshot, obviously the resolution suffers.

Downsides: There is a downside in that accessing it with a web browser sucks - on MacOS it requires Safari and a binary plugin. However, they do have desktop clients for Mac and Windows. Once again, Linux is left out in the cold. I might still end up running Shinobi against the QTSee (or the cams directly) for a simple remote web interface. 

The other downside is that I couldn't get my analog dvr added to the system. This isn't too big of a downer, because I'm going to replace those analog 480p cams with the higher quality 720p ESCams (maybe eventually 1080p cams). 


# Finances

tl;dr - I'm going to switch from Quicken to [beancount](http://furius.ca/beancount/) for double-entry plain-text accounting. 

I am planning to get a better handle on my finances in 2018. We're meeting all our bill payments, but I'm definitely not where I want to be with knowing where our money is going and planning for the future. For years, I used nothing substantial. I would do some balancing in spreadsheets, or try to use Mint, and I had Quicken for my business. In 2016, I used [hledger](http://hledger.org/) for about 4 months to track finances. I really liked the concept of [plain text accounting](http://plaintextaccounting.org/), but ultimately ended up purchasing Quicken and using that through the end of 2016 and all of 2017. I can essentially take a Tuesday and sync all my transactions down and reconcile it. In the world of personal finance, there are several camps, but two big ones are those that prefer syncing historical data (mint, quicken) and those that want you to be budget every transaction in advance, such as You Need A Budget and Every Dollar. There is overlap, especially since YNAB and EveryDollar have added syncing to their offerings. Plain Text Accounting/ledger/hledger fall into the second camp, with no sync capabilities.

That being said, I have used a program called [reckon](https://github.com/cantino/reckon) to import my main bank account into hledger. You go onto your bank website, download a CSV for a certain date range, and import it in. Even with Reckon, it was time consuming, and that's what led me to switch to Quicken. However, after using Quicken for 1.5 years, that can get time consuming as well. My family and I have a handful of credit cards, a mortgage, a car loan, checking accounts, savings accounts, 401k, roth ira, college savings, student loans, and a lot of transactions. For the most part, the bulk of our activity centers around a joint checking account. Just maintaining that one account in Quicken is a big time sink. If I don't update every Tuesday, it can take several hours to catch it up. This is because Quicken might mis or duplicate a transaction from the bank. Or something weird will happen. I might have everything caught up perfectly, and then the next time I'm in, I'll discover my balances are off going back 3 months. I'll have to spend time comparing statements and daily balances, going almost transaction by transaction - finding the most recent time when the balances match, then moving forward and fixing whatever caused it to diverge. I'll get things looking correct, then I'll jump forward a month and realize I had missed a divergence somewhere and I'll have to go back. By the time I get the main account squared away, I don't really feel like validating all the other accounts. If my Discover card balance is off, then I'll just have to go in an add a `BALANCE ADJUSTMENT` entry to bring it in line. I was trying to split my loan payments between principal and interest, but that went by the wayside.

Since I'm spending all of this time on Quicken reading every statement anyways, I decided I wouldn't be loosing much by going back to ledger. In fact, some banks such as US Bank has stopped offering integration with Quicken. So I'm going to start a brand new file and start tracking. This time, I'm going to dig around into web scraping. There are a lot of people out there that write tools to automatically log into their bank and download their CSV files. If I can semi-automate their retrieval, that will be a big win. I will also continue to use quickbooks to at least sync the data, but mainly to keep it as a backup if I decide to stop using ledger.  I probably will not use it, but there is a [quickbook to ledger converter](https://gist.github.com/genegoykhman/3765100)

While I was reviewing hledger, I found another system called [beancount](http://furius.ca/beancount/). It is another plaintext double-entry accounting system, but it's designed to have [less trust in the user entering data]
 (https://docs.google.com/document/d/1dW2vIjaXVJAf9hr7GlZVe3fJOkM-MtlVjvCO1ZpNLmg/edit?pli=1#heading=h.2ax1dztqboy7). There is a [ledger2beancount](https://github.com/glasserc/ledger-to-beancount) tool, so I can import any ledger files I had previously or make along the way (though right now I'm looking at a fresh start), and beancount itself provides a solid export to ledger. 

I'm going to start with beancount and see where it takes me. I might bounce a bit between beancount and ledger/hledger along the way. Beancount has some really nice web reports, and their example user in the [tutorial](https://docs.google.com/document/d/1G-gsmwK551lSyuHboVLW3xbLhh99JfoKIbNnZSJxteE/edit) sounds rather familiar. 

Worst case, I can drift back to Quicken.
