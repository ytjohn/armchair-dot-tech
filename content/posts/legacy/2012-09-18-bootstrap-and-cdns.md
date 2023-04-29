---

title: Bootstrap and CDNs
author: ytjohn
date: 2012-09-18 17:58:09

layout: post

slug: bootstrap-and-cdns

---
Often when creating a "modern" web page, it's very common to find yourself reinventing the wheel over and over again. I know any time I wanted to create a two-column layout, I would have to look at previous works of mine or search the Internet for a decent example. However, I recently came across Twitter's <a href="http://twitter.github.com/bootstrap/index.html" title="Twitter Bootstrap">Bootstrap</a> framework. At it's core, it's just a css file that divide your web page into a 12-column "<a href="http://twitter.github.com/bootstrap/scaffolding.html#grid">grid</a>". You create a "row" div, and inside that row you place your "span*" columns. Each span element spans from 1 to 12 columns, and should always add up to 12 for each row. You can also offset columns. There are css classes for large displays (1200px or higher), normal/default displays (980px), and smaller displays such as tablets (768px) or phones (480px). Elements can be made visible or hidden based on the device acessing the site (phone, tablet, or desktop). There is also a <a href="http://twitter.github.com/bootstrap/javascript.html" title="Bootstrap Javascript">javascript</a> component you can use for making the page more interactive.

If you download bootstrap, you get a collection of files to choose from. There's js/bootstrap.js, img/glyphicons-halflings.png, img/glyphicons-halflings-white.png, css/bootstrap.css, css/bootstrap-responsive.css. There is also a compress .min. version of the javascript and css files. You can read further about the <a href="http://twitter.github.com/bootstrap/scaffolding.html#responsive">responsive</a> version of the css, or how to use the <a href="http://twitter.github.com/bootstrap/base-css.html#icons">icons</a>.

Normally, one would take these downloaded files and put them into their own web application directory tree. However, there is a better way. Unless you are planning to use this on an Intranet with limited Internet access, you can use a copy of these files hosted on a "content delivery network (CDN)". A good example of this is the <a href="http://jquery.org/">jQuery</a> library hosted on Google's CDN. Google has a number of <a href="https://developers.google.com/speed/libraries/devguide">hosted libraries</a> on their network. This has several advantages, one of which being caching. If everyone is pointing at a common hosted library, that library gets cached on the end-user's machine instead of being reloaded on every site that uses that library.

While bootstrap is not hosted on google, there is another CDN running on <a href="http://www.cloudflare.com">CloudFlare</a> called <a href="http://cdnjs.com/">cdnjs</a> that provides a lot of the "less popular" frameworks, including bootstrap. Here are the URLs to the most current version of bootstrap files (they have version 2.0.0 through 2.1.1 currently).

<ul>
<li>http://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/css/bootstrap.css</li>
<li>http://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/css/bootstrap.min.css</li>
<li>http://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/css/bootstrap-responsive.css</li>
<li>http://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/css/bootstrap-responsive.min.css</li>
<li>http://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/bootstrap.js</li>
<li>http://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/bootstrap.min.css</li>
<li>http://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/img/glyphicons-halflings-white.png</li>
<li>http://cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/img/glyphicons-halflings.png</li>
</ul>

All one has to do in order to use these is to add the css and the javascript (optional) to their page. Since most CDNs support both http and https, you can leave the protocol identifier out.

<pre><code>&lt;link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/css/bootstrap.min.css"&gt;
&lt;script src="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/bootstrap.min.js"&gt;
&lt;script src="//ajax.googleapis.com/ajax/libs/jquery/1.8.3/jquery.min.js"&gt;&lt;/script&gt;
</code></pre>

Here's an example you can use on your own.

<pre><code>&lt;!DOCTYPE html&gt;
&lt;html lang="en"&gt;

&lt;body&gt;
&lt;div class="container-fluid"&gt;
        &lt;div class="row-fluid"&gt;
         &lt;div class="span12 label label-info"&gt;
                &lt;h1&gt;Header&lt;/h1&gt;
         &lt;/div&gt;
        &lt;/div&gt;

        &lt;div class="row-fluid"&gt;
         &lt;div class="span2"&gt;
                left column
                &lt;i class="icon-arrow-left"&gt;&lt;/i&gt;
         &lt;/div&gt;
         &lt;div class="span6"&gt;

                &lt;p&gt;center column

                &lt;i class="icon-tasks"&gt;&lt;/i&gt;&lt;/p&gt;

                &lt;div class="hero-unit"&gt;
                 &lt;h1&gt;This is hero unit&lt;/h1&gt;
                 &lt;p&gt;It is pretty emphasized&lt;/p&gt;
                &lt;/div&gt;

                &lt;p&gt;still in the center, but not so heroic&lt;/p&gt;

         &lt;/div&gt;
         &lt;div class="span4"&gt;
                right column
                &lt;i class="icon-arrow-right"&gt;&lt;/i&gt;
         &lt;/div&gt;
        &lt;/div&gt;
&lt;/div&gt;&lt;!-- end container --&gt;

&lt;!-- load everything at end for fast content loading --&gt;
&lt;link rel="stylesheet" href="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/css/bootstrap.min.css"&gt;
&lt;script src="//cdnjs.cloudflare.com/ajax/libs/twitter-bootstrap/2.1.1/bootstrap.min.js"&gt;
&lt;script src="//ajax.googleapis.com/ajax/libs/jquery/1.8.3/jquery.min.js"&gt;&lt;/script&gt;
&lt;/body&gt;
&lt;/html&gt;
</code></pre>

Finally, I found that <a href="http://www.netdna.com/">NetDNA</a> also hosts bootstrap on their CDN at [www.bootstrapcdn.com]. I would say that either CDN would be fairly reliable, as they are both sponsored by their CDN they are running on. One advantage of this site is that they provide a lot more than just the basic bootstrap hosting such as custom themes and fonts.

To use them, you can simply swap out your css and js scripts.

<pre><code>&lt;link href="//netdna.bootstrapcdn.com/twitter-bootstrap/2.1.1/css/bootstrap-combined.min.css" rel="stylesheet"&gt;
&lt;script src="//netdna.bootstrapcdn.com/twitter-bootstrap/2.1.1/js/bootstrap.min.js"&gt;&lt;/script&gt;
&lt;script src="//ajax.googleapis.com/ajax/libs/jquery/1.8.3/jquery.min.js"&gt;&lt;/script&gt;
</code></pre>

<strong>UPDATE:</strong> I added jquery into the above examples because several parts of bootstrap rely on it (such as the Modal dialogs).

20120918
