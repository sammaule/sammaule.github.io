---
layout: post
title: "Technology of the site"
date: 2019-03-25
tags: test1 test2 test3
---

Building my own website has been on my to-do list for a long time. After having paid for this domain for longer than I'd care to admit I finally did something about it. This first post explains briefly the technology behind the site and the reasons for those choices. I don't intend to write many posts like this, but I thought this might be useful for anyone who wants to create their own site but is currently procrastinating!

The main decision that I had to make was whether to host my own site, or use a service like WordPress or Squarespace. There are two main considerations that led me to choose to host my own site:

1. Simplicity. Most managed services still require the user to have some knowledge of the innards of how websites work. For the sake of a very simple site that will contain a few blog posts, understanding databases and web servers seemed excessive. Whilst it's true there is some learning curve to hosting your own site, this is relatively short. I had a simple website up and running in an afternoon.

1. Cost. Simply put, hosting my own site is free. This compares favourably to managed services which of course ask for a fee to maintain your site for you. Whilst these aren't particularly expensive (about £35 a year for a Wordpress hosted site, maybe three times that for Squarespace) I didn't feel like I was getting much for that money. Yes the site would no doubt look a bit better, but that's about it.

Taken together, hosting my own site scores fairly high on simplicity (although lower than something like Squarespace) and high on cost (it's free!), so seemed like the best overall option.

One of the most popular options for static sites is GitHub pages used in combination with Jekyll. I liked the look of [example sites](https://jekyllrb.com/showcase/) that use this approach, and they are similar to the sort of site  I want to create, so I jumped straight in.

There were basically only two resources I needed to get everything up and running. The first is a great [tutorial in setting up Jekyll based sites on GitHub Pages](http://jmcglone.com/guides/github-pages/), which is super easy to follow, and gives you a working website in as little as half an hour of work. After a bit of fiddling around changing some of the layout of the site, the layout and adding a bit of my own content, the site was done!

The default domain for github pages is username.github.io. Since I wanted to add my own custom domain There was one final step to follow, which is to use a custom domain with GitHub Pages. Since I'd originally bought my domain with Gandi, I followed the [good instructions of Daniel Spector](http://spector.io/how-to-set-up-github-pages-with-a-custom-domain-on-gandi/) which worked well. The only change that needs to be made is to the DNS Records in Gandi as the IP addresses you need to point to must have changed since that blog post was written. At the time of writing the correct settings are in Figure 1.

![alt text]({{ sammaule.github.io }}/assets/technology-of-the-site-figure-1.png)

Figure 1: DNS settings on Gandi for GitHub Pages

That's all you need to do to get a custom domain site up and running with GitHub Pages and Gandi. I have also added a couple of other features to my page including Google Analytics. For those curious about the underlying code, here's the link to the [GitHub repository](https://github.com/sammaule/sammaule.github.io).
