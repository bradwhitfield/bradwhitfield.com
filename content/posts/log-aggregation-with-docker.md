+++
Description = ""
Tags = ["Logging", "Docker"]
Categories = ["Ops", "Docker"]
menu = "main"
comments = false
Title = "Log Aggregation With Docker"
date = 2018-02-16T22:01:57-05:00
draft = true
+++

My experience with this.

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
Centralized tool
  Good for "Platforms"
  Many cache in the event of failure
  Can add more metadata
  Less configuration
  Can help enforce some standards
  Consumes disk space (but that can be, and should be, limited)
  More complexity, as it's one more thing to manage (minimal)
  Easier to migrate if you have platform  (Splunk to ELK seems common lately)
Logspout
  Multiple endpoints
  Very flexible
  Fast
  Docker Socket (I think it will even stream for things configure to go elsewhere)
Maybe fluentbit?
Filebeat
  JSON mapping
  Often companies will have shared infrastructure
Maybe Splunk?
