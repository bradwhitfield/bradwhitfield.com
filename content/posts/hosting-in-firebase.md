+++
Title = "Hosting This Site"
Date = "2017-10-15T10:58:12-04:00"
Description = "An outline of how I built my static sites, and deployed them to Google Cloud Platform for free."
Tags = ["development", "static", "hosting", "gcp", "firebase", "hugo"]
Categories = ["Development", "Cloud"]
menu = "main"
comments = false
+++

Before working on this project, I had a crappy site that I had been meaning to update for a
couple years. It was previously built to satisfy the requirements of a class I was taking at
university using a bloated bootstrap template, and it was hosted on a VPS that I was already
paying for. Well, I finally got around to updating this site, and now I'm going to ramble
about the tools I'm using to build it, and the hosting options that I've explored to make
it cost almost nothing (or free depending on traffic).

# Hugo and the HPSTR theme

To start, I'm a lazy person. My VPS isn't a lot of work, but it's work that I don't really
need to do anymore since the tooling for static sites is so great. I used to use the VPS for
more than just web hosting, but not anymore, so it's no longer worth the upkeep.

For creating content like a blog, there are lots of good static site generators. There is
even [this site](https://www.staticgen.com/) listing them, which is provided by a static
site hosting company (makes sense). Since I'm a bit of a golang fanboy, and since the tool
works well, I went with [Hugo](https://gohugo.io/) and the
[HPSTR Theme](https://dldx.github.io/hpstr-hugo-theme/).

But this is just some boring background information. The hosting is really the point of my
ramblings in this post.

# Static Site Hosting

For a site that doesn't get a lot of traffic, there are several options available now
where you can host your site for free, or really cheaply.

## AWS

AWS Annoys me, but I know my way around it
Hosting in S3 is really cheap
Route 53 for DNS

# GCP

Because AWS annoys me, I wanted to check out GCP.
Better look and docs in limited experience
Better free tier
No TLS, but you can use CloudFlare free tier to use HTTPS
Plus $300

# Azure

https://www.microsoft.com/middleeast/azureboxes/cloud-hosting-for-a-static-website.aspx

# Firebase

Basically GCP
1 GB of storage - 10 GB of traffic
That's a lot for static site
firebase --serve and firebase init are pretty nice
A ton of things to install to get it all to work
Pretty easy to use

Based on some basic metadata scraping, it looks like NGINX.
https://firebase.googleblog.com/2016/09/http2-comes-to-firebase-hosting.html
https://nginx.org/en/docs/http/ngx_http_v2_module.html
It uses Let's Encrypt for certs
You could theoretically recreate this using Google App Engine, but I have not tried.

Caveat with the GCP trial pay-as-you-go stuff

# The Winner is Firebase, For Now

That's what this rev is hosted with. I have two other sites to migrate, so I may host those
other places just to try it out.
