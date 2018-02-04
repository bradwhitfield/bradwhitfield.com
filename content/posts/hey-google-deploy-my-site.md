+++
Description = "I automated deploying my Hugo site to Firebase with Google Assistant for the fun of it."
Tags = ["Development", "golang", "Firebase", "Google Assistant", "IFTTT"]
Categories = ["Development", "GoLang"]
menu = "main"
comments = false
Title = "Hey Google Deploy My Site"
date = 2018-02-04T13:33:10-05:00
draft = false
+++

Lately I've been playing with GCP and Firebase just for the sake of exploring the offerings
that cloud providers other than AWS have. As I mentioned in a [previous post](/posts/hosting-in-firebase),
this site is hosted on Firebase for almost nothing, and I'm playing around with GCP features while
using the free tier, and free trial credit. With the [free tier](https://cloud.google.com/free/) alone,
you get a lot of Google Cloud Functions executions, and plenty of Container Builder time before
you have to pay anything. So for shits and giggles, I decided to make it so Google Assistant will
deploy my site for me.

Note that this is not a full tutorial, and none of this is production ready. This is just the
ramblings of someone who likes to play with GCP and Golang. If you really wanted to use this
for important workloads, there are things to consider like security of the Firebase CI token.

# Breakdown and Flow

It's pretty simple when you break it down. Basically I have a custom IFTTT Applet that triggers
a web hook. That web hook calls a Google Cloud Functions URL. When triggered, the Cloud Function
will parse the request, and turn it into environment variables for a Container Builder build step.
Container build then runs an image I built that reads those environment variables, runs `Hugo`,
and deploys that to Firebase. Easy enough.

![Deployment Flow](/images/deployment-flow.jpg)

# IFTTT Setup

[IFTTT](https://ifttt.com) allows you to create an applet that is trigged by custom phrases
said to Google Assistant. That phrase can then trigger a web hook for custom integrations. An
example of what I have setup is below.

![IFTTT Setup](/images/ifttt.jpg)

You should be able to see now why I called out security above. For this simple personal project,
I'm not too worried about giving IFTTT my Firebase CI token, but I wouldn't do this for a company
that I worked for. Roles would be the best way to handle this, but I haven't yet figured out
how to allow Firebase Hosting access from a GCP Role.

# Google Cloud Functions Setup

I like writing Go code, so for this, I used Kelsey Hightower's [Golang shim](https://github.com/kelseyhightower/google-cloud-functions-go).
Following the http example provided in that repo, I created a quick and dirty bit of code that parses
a JSON request, feeds that into environment variables for the Container Builder build step, and
triggers a new build. The code for that can be [found here](https://github.com/bradwhitfield/FirebaseDeployBot).

# Container Builder Setup

In the FirebaseDeployBot repo mentioned above, I created a `docker` folder that contains a Dockerfile
and an entrypoint script. The Dockerfile creates an Alpine-based image that has Hugo and the Firebase CLI
installed. The entrypoint script clones the site repo and submodules from GitHub, builds the Hugo site, and
deploys to Firebase based on the environment variables passed to the container when it's started.

Container Builder doesn't really have anything setup, necessarily, but the GCP function will tell
Container Builder to run a container from the image defined in the Dockerfile (which I put
in [Docker Hub](https://hub.docker.com/r/bradwhitfield/firebasedeploybot/)), and set the environment
variables that the script requires to build and deploy.

# That's It

Now when I post this page, all I do is `hugo serve` locally to verify it looks good, commit it to GitHub,
and ask Google to build my site.

![Assistant Building My Site](/images/assistant.jpg)
