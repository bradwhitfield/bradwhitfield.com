+++
Description = ""
Tags = ["Logging", "Docker"]
Categories = ["Ops", "Docker"]
menu = "main"
comments = false
Title = "Log Aggregation Strategies With Docker"
date = 2018-02-16T22:01:57-05:00
draft = true
+++

My experience with this.
Many ways to do this.

Cloud Native or Cloud First?

Probably why you should care about what I say
use what you have
Use what is right for you goals
Go over what I think is appropriate at what scale
Talk about the paradigm change
  STDOUT
  Avoid volumes
  /var/lib/docker/container/ & ?/var/run/docker.sock?
  Unify management and remove the need to handle that per app
Logging drivers are easiest
  Easy to setup
  flexible
  Sucks if you have to configure it everywhere
  Containers can't start if endpoint is down
Volumes if You Must
Centralized tool
  Good for "Platforms"
  Many cache in the event of failure
  Can add more metadata
  Less configuration
  Can help enforce some standards
  Consumes disk space (but that can be, and should be, limited)
  More complexity, as it's one more thing to manage (minimal)
  Easier to migrate if you have platform  (Splunk to ELK seems common lately)
  Can be overridden with logging drivers or the sidecar pattern
Logspout
  Multiple endpoints
  Very flexible
  Fast
  Docker Socket (I think it will even stream for things configure to go elsewhere)
Fluentd
Filebeat
  JSON mapping
  Often companies will have shared infrastructure
Maybe Splunk?

# Abstract: Unifying Log Aggregation in a Conterized World

As containers become more popular, our industry is changing. Although this change can feel radical to
some, when thought about correctly, we gain a lot of benefits such as developers having more freedom to use
the right tool for the job and Sys Admins gaining more control and stability over the infrastructure.
Tools like Kubernetes, Swarm and Elastic Container Service make it easier for engineers to ship code
and maintain the infrastructure it lives on. The control plane is moving a layer higher than the
cloud providers and this allows us to standardize tools such as monitoring, alerting and logging.

In this talk, we will focus on what is arguably the most important thing for anyone on-call, maintaining
applications or debugging code - an application's logs. Leveraging Docker and an orchestration solution,
teams can aggregate logs for all of their applications and funnel them into a unified platform of choice.
We will go over several strategies that teams can use to simplify log aggregation that meets the
scale and business needs beyond volumes and logging drivers can provide.

# Why Care About What I Say?

<!-- 
I've been running containers in production since 2015.
I'm DCA certified if you care about that sort of thing.
  Most of this talk doesn't come from that knowledge.
I've managed logs for clusters using several approaches.
I've been on the development and sysadmin side of things.
These are things I've learned over time.
 -->

Basically it's because I've been doing this for a while, and I've learned a few things over time.

Since 2015, I've been helping run containers in production. I've been on both team ops, and team dev since
then. Over the years, I've tried a number of different strategies with varying success. So rather than
you having to learning the hard way, I'll explain some of the different ways things work well, and when.

If you care about credentials, I am a Docker Certified Associate. It's a cool title and all, but what
most of what is in this post does not come from that studying.

# Approaches to Unified Log Aggregation

At the highest level, you can divide up log aggregation into two categories - container level
or centralized tool level.

## Container level

Managing logs at the containers level, is the least complex, and least rewarding approach.

# Background on How Docker Logs Work

https://docs.docker.com/engine/api/latest/

For Docker, there are two really important concepts to be aware of in order to understand how logs
can be captured. 