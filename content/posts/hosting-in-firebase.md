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

I had previously explored firebase for very simple projects, but never really their hosting. During my
explorations of GCP I did learn that Firebase is basically a gateway drug to GCP, so that's intriguing.

If you look at the [pricing page](https://firebase.google.com/pricing/), you get 1 GB of storage and
10 GB of traffic for Hosting in the free Spark Plan; that's quite a bit for my static site needs. The
[documentation](https://firebase.google.com/docs/hosting/) makes it look straight forward too.

There is a caveat with using Firebase. If you already have a GCP account on the email you are using
for Firebase, it appears that have to use the pay-as-you-go plan. Still, it's relatively inexpensive
unless you get a lot of traffic.

## Azure

For Azure, I didn't explore this as much. This was one of the last ones on my list, and I found that
GCP and Firebase would both give me free hosting (for my site anyways). Although Azure is still another
really cheap competitor, I'm incredibly stingy. If I was already Azure, I would probably explore this
more, though. [Here are the docs](https://www.microsoft.com/middleeast/azureboxes/cloud-hosting-for-a-static-website.aspx)
for if you want more info.

## The Winner is Firebase, For Now

This version of the site is currently hosted on Firebase. So far, I like it. In practice, the tooling
was pretty easy to use (but a lot to install), and it meets my needs. Basically my workflow is what's
below. I do have some automation in there, but this is the gist of what happens.

```bash
hugo serve

# write stuff

hugo
firebase --serve

# make sure it looks good
firebase deploy
```

After deploying, I started scraping some metadata and checking searching around a bit, and I was pretty
happy to find that they are probably using NGINX to host the sites, the site
[supports HTTP2](https://firebase.googleblog.com/2016/09/http2-comes-to-firebase-hosting.html), and Firebase
hosting gets an "A" on SSL Labs. As a side note, they're using Let's Encrypt as the certificate authority.

![SSL Labs Results](/images/firebase_hosting_ssl_labs.jpg)

I have two more sites to migrate off of this VPS, at the time or writing, so I might explore using GCP and
CloudFlare for one of them. For my wife's site, I'm going to migrate it to Firebase, because it will be easy
enough for her to do on her own before I get around to automating everything. Overall, I'm happy with Firebase,
but it's still very early days.
