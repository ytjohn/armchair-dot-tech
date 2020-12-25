---

title: First impressions with emacs
author: ytjohn
date: 2017-12-24 13:44:38

layout: post

slug: first-impressions-with-emacs

---
Learning emacs really sucks. Let's do this. 

## My current stack

Because I'm insane, I decided to give Emacs a try. While I'm still pretty pleased with [Omnifocus](https://www.omnigroup.com/omnifocus), I do find limitations with it. I also store all kinds of notes in [Quiver](http://happenapps.com/#quiver). Anytime I'm working on anything, I keep my notes in Quiver. I had been keeping a worklog in it, but it got goofy when work (and the associated notes) span multiple days. I also use [PyCharm](https://www.jetbrains.com/pycharm/) as my IDE. That I definitely like, but if I want to open a single file, I'm going to pop open vim, sublime text, or even just textedit.

## Why I'm looking at Emacs

For years, I've been hearing amazing things about [orgmode](http://orgmode.org/). It's used for todo lists, project planning, documents, and [journals](https://github.com/bastibe/org-journal). People also like using emacs as an [python ide](https://robots.thoughtbot.com/emacs-as-a-python-ide). Everyone that uses emacs seems to really like it, and it keeps showing up as a good looking solution to different problems I'd like to solve. There's even a [slack client](https://github.com/yuya373/emacs-slack) for it. I decided that I should really give emacs a shot before discarding any solution because it happened to be emacs based. You see, emacs has a super steep learning curve, and when you use it, you essentially join a cult. 

## Round 1

So I decided to dive in. I found a number of people recommending [How to learn emacs](http://david.rothlis.net/emacs/howtolearn.html) as a good place for beginners. The first page has this juicy tidbit:

>What I need from you is commitment (a couple of dedicated hours of study per week, and to use Emacs as your day-to-day editor) and patience (be willing to give up your favorite IDE’s features, at least for a month or two).

That's pretty intimidating, but to be fair, Vim takes a long time to master. Those firs starting out need to learn what a modal editor is, how to switch between insert and command mode, how to navigate, how to do search and replace, how to delete lines, and possibly how to use vim's internal clipboard operations. That's all before you get into customizing and extending the program to turn it into an [ide](http://coderoncode.com/tools/2017/04/16/vim-the-perfect-ide.html). 

I put a couple hours in the first weekend, and a little bit of time the following week going through the examples. But I got bored and real life kept me away.

## Round 2

Seeing sometime ahead of me, I figured I'd try again. I went back and forth between plain emacs, [spacemacs](http://spacemacs.org/) and [prelude](https://github.com/bbatsov/prelude). I did research all over about how people got started with emacs. Lots of heavy opinions on "starting pure", or using a starter pack like spacemacs/prelude. For those with vim leanings, there is an "[evil mode](https://www.emacswiki.org/emacs/Evil)" that provides some vim keybindings and emulation. I came across the [Mastering Emacs](https://www.masteringemacs.org/) book which gots some good feedback on [reddit](https://www.reddit.com/r/emacs/comments/3a21y2/anyone_bought_the_mastering_emacs_ebook/). 
I started reading a copy of the book with pure emacs open. It's 277 pages long and I got to about page 30 before I started falling asleep. However, here are some key things to know:

* Emacs isn't based on files, but based on buffers (which may contain a file).
* What we call a window, emacs calls a frame. 
* A frame contains multiple windows (or window panes)
* You can assign any buffer to any window, or even have multiple windows showing the same buffer.

## Round 3 - Just edit some python already

Screw reading books and working through tutorials. I'm just going to go ahead and start using emacs and then google every time I got stuck. That's how I learned vim back in the late 90s, except I didn't have google back then. In fact, I didn't even have Internet at home back then. I had to go to school, search altavista, download manuals and deb packages to floppy disk, then take them home and transfer them.

I figure, just use the stupid program this week and expect normal operations to take longer while I work things out. 

<img src="https://av.yourtech.us/photos/galleries/0harhar/aai.jpg" alt="I don't know how to exit vim" />

So first things first, I took the popular advice and installed spacemacs, which gives me fancy color themes and evil mode. 

1. If you fire up emacs, it's a decent gui, complete with helpful menus and mouse integration. You can open a file, edit, save it exit almost as easily as any other text editor. File -> Visit new file and File -> Open do make you type in the path the file instead of a file open dialog gui, but there is a sort of autocomplete/directory listing interface.
2. Emacs with spacemacs takes a long time to load. It's on par with pycharm slow load times. Kind of sucks if emacs is closed and you just want to open a file. The one book I says that I should run emacs in a a server mode and just use the emacsclient binary for all subsequent starts. Ok, fine - if I have emacs up all day long, that's doable.
3. Emacs can run a shell. You can run `shell` which runs your default shell (bash in my case) in a buffer. [Emacs fans call this the inferior shell](https://www.masteringemacs.org/article/running-shells-in-emacs-overview). The "emacs shell" or `shell` is promoted as superior. It's a bash-inspired shell written in elisp. **Both shells suck**.  I thought I'd be able to run a terminal in a window below the file I'm editing like I do in pycharm, but it's extremely frustrating working in this shell. Ctrl-C is "C-c C-c", and it's really easy to end up no typing over output from a pervious command. Worst of all, I could not activate a virtualenv in either shell. This means I couldn't drop to a shell and run python ad-hoc python tools. While there may be some amazing tweaks and features these shells bring, I found it much like working on a bad serial connection.
4. When I opened a python file, spacemacs detected this and asked if I wanted to install the python layer. This gave me nice syntax highlighting, but I didn't get any autocomplete like I was hoping for. I know that "helm" is enabled, but there is perhaps something else I have to do for autocomplete to work.


## projectile for projects

Spacemacs bundled an add-on called [projectile](http://projectile.readthedocs.io/en/latest/). This is pretty nice. Incidentally, "bbatsov" writes a lot of stuff for emacs, including the previously mentioned [prelude](https://github.com/bbatsov/prelude). People recommend prelude over spacemacs because they feel spacemacs adds could add complexity that could confuse a beginner once they get past the initial learning curve. Ie, spacemacs is good for super beginners, but bad for intermediate users. Or so I've heard.  

Anyways, this add some nice functionality. Open a file inside a directory that is under git control, and it establishes all files in the directory as a project. If you have all your projects in a directory like `~/projects`, you can teach emacs about all them at once.

    M-x projectile-discover-projects-in-directory ~/projects

Once you scan them all, you can run `C-c p F` to show a list of all known projects and select one to open. Open a file in any project and it puts you in project mode. There are shortcuts to see all files in the project, if you open a shell it drops you in the project directory. You can also quickly switch between recently opened files, perform in-project search and replace. 

## org-mode

So far, org-mode has been my most positive experience. I wrote up a general outline of a software I'm working on and I found it much easier to write with and organize than when I write in markdown.

It's not markdown, and expecting to be able to use things like markdown code blocks will disapoint you. But it's definitely learnable and I can see myself using it. 

You just go ahead and open a file ending in `.org` and start writing. Headers start with `*` instead of `#` but otherwise will be familiar to a markdown user.

The real nice bit of org mode is as you learn the hot keys and easy shortcuts. Key combinations will create new headings and list entry, or you can move an entire section up, down, indent or outdent. 

If you type `< s <TAB>`, it expands to a ‘src’ code block:

```
#+BEGIN_SRC 

#+END_SRC
```

I only did some basic outlining, but it seemed workable. I can see emacs/orgmode possibly replacing quiver as my primary notebook. It won't be easy, because quiver has a this nice feature were you just start writing a note and that note may or may not get a title. There is no need to save that note to a file, because it's saved within the quiver datastore. Emacs will want me to save a file for each note. 

Probably a next step is to test out the orgmode-journal. After that, dive into [orgmode and Getting Things Done](http://members.optusnet.com.au/~charles57/GTD/orgmode.html). If I can put my omnifocus tasks into emacs and use it as a daily work notebook, then this time invested so far won't be entirely put to waste.

Follow up: I came across this [orgmode day planner approach](http://newartisans.com/2007/08/using-org-mode-as-a-day-planner/), which seems even more workable than the GTD approach linked above.
