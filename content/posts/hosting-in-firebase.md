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

With AWS, it's really easy to [use S3 and Route 53](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html)
for hosting a static site. If you want to get fancy, you can even add CloudFront as a CDN to
make your website even faster. It's really cheap, and to host this site at the current size and
rate of traffic, the calculator shows that it would be pennies each month to host without CF, and
under $0.10 a month with CF.

Theses fees assume that you are out of the [Free Tier](https://aws.amazon.com/free/) usage. If you ran
your site for a year on AWS, you would probably get a good idea of what it will cost you later. My site
gets so little traffic that your bill will probably be a bit higher.

I've been working in AWS for a couple of years now. I love not having to rack and wire
servers anymore (yes I've racked servers), but the more I use AWS, the more it's
inconsistencies annoy me. Some services are better than others (S3 being a better one), but
I like the idea of seeing what else is out there, and if the other cloud providers are any better.

## GCP

As I mentioned previously, I wanted to check out other cloud providers because I get annoyed with
AWS sometimes. Google Cloud Platform (GCP) has always interested me for some reason, so I used this as an
excuse to play around with it. In my initial impression, I'm impressed. The UI is much prettier, and more
user friendly, and so far the docs have been great. The [always free tier](https://cloud.google.com/free/)
is better than the [AWS Offering](https://aws.amazon.com/free/) for my needs, and the $300 credit
to use in the first year is pretty compelling. Hopefully GCP will keep impressing me as I continue to
play around with it.

When you look at the docs for
[static site hosting on GCP](https://cloud.google.com/storage/docs/hosting-static-website), there is a
red flag (almost literally) that sticks out right away - GCP does not natively support hosting over HTTPS.
As they call out in the docs, you can use something like CloudFlare to secure your site, or you can use
Firebase Hosting. I'm kind of a nut about securing everything, regardless of what it is. I also like the
idea of simple management that's all in one place. For this reason, I added Firebase Hosting to the list
of options to check out.

![HTTP-Something-Here]()

## Firebase

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

## Azure

https://www.microsoft.com/middleeast/azureboxes/cloud-hosting-for-a-static-website.aspx

## The Winner is Firebase, For Now

That's what this rev is hosted with. I have two other sites to migrate, so I may host those
other places just to try it out.
