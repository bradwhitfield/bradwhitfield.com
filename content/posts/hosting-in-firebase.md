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

I've been some form of Operations Engineer for a while. Although I know how to maintain and
secure a server, I also know that there isn't really a need to anymore for something simple
like a blog. There are many good [static site generators](https://www.staticgen.com/) out there
that make it easy to create blogs using only static HTML, CSS, and JS.

Static site generators
No need to secure and update something like wordpress or ghost
Since everything is static files, hosting becomes very easy

Chose hugo because I is golang fanboy
I liked the HPSTR theme

# Static Site Hosting

You can now host for free or close to it, unless you site is massive, or gets a ton of traffic

# AWS

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
