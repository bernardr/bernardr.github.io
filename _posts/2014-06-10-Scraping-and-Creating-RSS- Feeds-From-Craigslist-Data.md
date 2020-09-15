---
layout: post 
title: Scraping and Creating RSS Feeds From Craigslist Data
---

## The Problem and it’s Obvious Solution

It’s been said, that when all you have is a hammer everything looks like a nail. Lately, I’ve been wielding a hammer called “web scraping”, and everywhere I look I see nails in need of hammering.

With that said, I’ve been doing some work with the great Ruby Gem [Nokogiri](nokogiri.org) lately and decided to put it to some use, by using it on Craigslist.

In the past, I’ve found work in the Craigslist Computer Gigs \[`/cpg/`\] section in various cities. But I’d been irritated by having to open tabs for each new city, and it’s /cpg/ listing.

The solution isn’t graceful, but is obvious: RSS.

## RSS’ Recent History

[RSS](http://en.wikipedia.org/wiki/RSS) isn’t discussed much anymore since Google shuttered it’s [Reader](http://en.wikipedia.org/wiki/Google_Reader) service a few months ago. I recall their reasoning being along the lines of, “RSS isn’t really used much anymore …”. The nerds were upset. I was one of them.

Shortly after that, Twitter stopped supporting RSS feeds as well, and RSS entered it’s current phase: that orange icon on many a site, but not fully utilized by visitors more comfortable with Google+ and Facebook “Like” buttons.

To me, the sight of it has the air of an older web. One dedicated to open formats, and open data. Which is probably why Craigslist still supports it.

Craigslist
----------

Craigslist.org features an RSS feed for nearly every page of it’s classified ads. From personals, event listings, and the especially the category we’re intereted in: job postings.

### Getting the Data

The main page of [Craigslist.org](Craigslist.org) (CL) lists over 400 different sites, which provide the base URL for each city’s particular CL. You _could_ take a few hours out of your day and copy-paste all of those URL’s into a text document **or** you could fire up an IRB session and tinker with Nokogiri for a few minutes.

I know what choice I made.

To make this process that much easier,I’ve put all the URL’s in a `.csv` for you, which you can [download here](https://dl.dropboxusercontent.com/u/493451/craigslist-scrape/craigslist-sites.csv) \[dropbox link\].

###Tweaking the URL’s

Here’s a quick peek of the list of URL’s:

URL’s

- http//auburn.craigslist.org
- http//bham.craigslist.org
- http//dothan.craigslist.org
- http//shoals.craigslist.org
- http//gadsden.craigslist.org

The url paths are the same for all of the craiglist clasified pages. Simply adding `/cpg/` to the end of any domain will get us the page we need. You can, obviously, add any sub-page — and search parameter — but for for my purposes all I needed was the `/cpg` added to the 400+ URLs.

This is where [SublimeText 2’s](http://www.sublimetext.com/2) multiple cursor feature eats the lunch of all text editors on the block. If you’re not hip to it, **[watch this video now](https://www.youtube.com/watch?v=WXuBgSpLpK4)** and forever change the way you see a text editor.

Now, we have a text document with the URL’s appended with exactly what we need. Something like this:

Updated URL’s

- http//auburn.craigslist.org/cpg/index.rss
- http//bham.craigslist.org/cpg/index.rss
- http//dothan.craigslist.org/cpg/index.rss
- http//shoals.craigslist.org/cpg/index.rss
- http//gadsden.craigslist.org/cpg/index.rss

## Creating an RSS Feed of all the sites

Creating an RSS feed will be the easiest part. There are lots of services online that will do the conversion of a text or .csv of urls into an OPML file.

I used [Feedshow](http://reader.feedshow.com/goodies/opml/OPMLBuilder-create-opml-from-rss-list.php), having used them in the past, but googling around can probably yield something comparable or out right better.

After importing my data, and exporting out an OPML file I was then able to take my 500+ RSS feeds on to the RSS reader of my choice.

I first tried [Feedly](http://feedly.com/), but discovered they’d had [issues updating their Craigslist feeds in the past](http://blog.feedly.com/2014/01/29/update-regarding-the-craigslist-feeds/). To Feedly’s credit, they worked hard and did a great job of [fixing the issue](http://blog.feedly.com/2014/03/14/fix-it-march-5-craigslist-pipe-repaired/), but I still decided to seek other options.

Right now, I’m switching back and forth between [Vienna](http://www.vienna-rss.org/) and [RSS Owl](http://www.rssowl.org/). I’m a long-time [NetNewsWire](http://netnewswireapp.com/) user, but decided that importing a file of 400+ feeds into a libary of over 600 might not be the best idea.

### A Few Notes

I’ve been using this little system for a few days now, and I’m finding it works for me. I’m able to view and respond to ads at a rate, I’m pleased with.

Also, I’m not certain, but I feel that Craigslist might be delaying RSS feeds in order to drive traffic to the site … but that might just be silly thinking.

The system could use a little tweaking. For example, I’m considering creating specialized search URL’s to increase the signal to noise ratio inherent in Craigslist ads.

I’m sure there are any number of refinements I’ll make, but as of now: I’m pleased.
