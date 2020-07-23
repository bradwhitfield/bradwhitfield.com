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
  Can be (and should be) configured at the host level
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

When reading this, it's worth noting that this is based on a presentation I gave at a meetup. The sections of
this post should follow [the slides](/log-slides.pdf) pretty closely, and they are much less dense. Instead
of reading this article all the way through, I would recommend skimming the slides first, and then reference
this post for more details. You can check out the [scripts on Github](https://github.com/bradwhitfield/logging-talk)
used during the talk as well.

# Abstract: Unifying Log Aggregation in a Containerized World

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

# Brief Docker Background Info

The common analogy for is that it's like a lightweight virtual Machine. In many ways, that's really a
bad analogy. Docker is really just a way to package and run a single process in an isolated way. An image
represents all the binary bits needed for a process to run (think Linux package), and the container is
the runtime configuration of the application, such as environment variables, CPU share, and network.

From a logging standpoint, it helps t
<Insert stuff here> to understand how PID 1 is managed, and that it's output and error is captured

## Process ID 1

When a Docker container is started, the entry point or command of the container is given Process ID (PID) 1.
You can see evidence of this by running a container and issuing a `ps -ef` command through `docker exec`.

```bash
> docker run -d --name httpd httpd
b162d1dfd6c1e37f9b85054677b48d74be47ea8b430e1b9b7ecf310d0a7b24e0
> docker exec -it  httpd ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  1 22:55 ?        00:00:00 httpd -DFOREGROUND
daemon        6      1  0 22:55 ?        00:00:00 httpd -DFOREGROUND
daemon        7      1  0 22:55 ?        00:00:00 httpd -DFOREGROUND
daemon        8      1  0 22:55 ?        00:00:00 httpd -DFOREGROUND
root         90      0  0 22:55 pts/0    00:00:00 ps -ef
```

You'll see here that according to the container, httpd and it's child processes are the only things
running in the container (besides the `ps` command that was just run). How the isolation works in
docker is out-of-scope for this topic, but it's important to note that PID 1 and it's children are the
processes that are tracked and managed by Docker.

## STDOUT/STDERR

By default, everything that is sent to standard out and standard error (e.g. printing to the screen) is handled
by Docker, provided it's from PID 1 or it's children. What Docker does with this output is dictated by the
logging drivers, but by default it's logged to a JSON file on disk. We'll discuss more on logging drivers later.

The reason we covered PID 1 is because it's important to know that other processes output will not be captured
by Docker. Most of the time this isn't a problem. Were you'll see issues is if you are doing some abnormal
process spawning, or if you're trying to run a daemon manager in a container (which defeats the purpose
of Docker anyways).

To illustrate this point, you can see that the `ps -ef` command run in the previous example does not
get captured by Docker logging drivers.

```bash
> docker run -d --name httpd httpd
b162d1dfd6c1e37f9b85054677b48d74be47ea8b430e1b9b7ecf310d0a7b24e0
> docker exec -it httpd ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  1 22:55 ?        00:00:00 httpd -DFOREGROUND
daemon        6      1  0 22:55 ?        00:00:00 httpd -DFOREGROUND
daemon        7      1  0 22:55 ?        00:00:00 httpd -DFOREGROUND
daemon        8      1  0 22:55 ?        00:00:00 httpd -DFOREGROUND
root         90      0  0 22:55 pts/0    00:00:00 ps -ef
# Note you don't see `ps -ef` results in here because it's not a child of pid 1, but PID 0
> docker logs httpd
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.3. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.3. Set the 'ServerName' directive globally to suppress this message
[Sun May 06 22:55:56.471987 2018] [mpm_event:notice] [pid 1:tid 140121111680896] AH00489: Apache/2.4.29 (Unix) configured -- resuming normal operations
[Sun May 06 22:55:56.472067 2018] [core:notice] [pid 1:tid 140121111680896] AH00094: Command line: 'httpd -D FOREGROUND'
```

## Location on Disk

In a typical Docker installation, the default method Docker uses to handle logs is via JSON log files. If a
container is run without specifying a logging driver, the log file will be found at
`/var/lib/docker/containers/<container-id>/<container-id>-json.log`. You can find the exact path using the
`docker inspect` command, and searching for `LogPath`.

```bash
> docker inspect httpd -f '{{ .LogPath }}'
/var/lib/docker/containers/b162d1dfd6c1e37f9b85054677b48d74be47ea8b430e1b9b7ecf310d0a7b24e0/b162d1dfd6c1e37f9b85054677b48d74be47ea8b430e1b9b7ecf310d0a7b24e0-json.log
```

When you look at the log file created on disk, you'll see the log message, the timestamp, and the stream the message
originates from (STDOUT or STDERR).

```bash
> cat /var/lib/docker/containers/b162d1dfd6c1e37f9b85054677b48d74be47ea8b430e1b9b7ecf310d0a7b24e0/b162d1dfd6c1e37f9b85054677b48d74be47ea8b430e1b9b7ecf310d0a7b24e0-json.log
{"log":"AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message\n","stream":"stderr","time":"2018-05-07T00:51:17.273889465Z"}
{"log":"AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message\n","stream":"stderr","time":"2018-05-07T00:51:17.275769048Z"}
{"log":"[Mon May 07 00:51:17.276841 2018] [mpm_event:notice] [pid 1:tid 139820362606464] AH00489: Apache/2.4.33 (Unix) configured -- resuming normal operations\n","stream":"stderr","time":"2018-05-07T00:51:17.276927337Z"}
{"log":"[Mon May 07 00:51:17.276951 2018] [core:notice] [pid 1:tid 139820362606464] AH00094: Command line: 'httpd -D FOREGROUND'\n","stream":"stderr","time":"2018-05-07T00:51:17.276967037Z"}
```

## Logging Drivers

Docker provides several logging drivers as a way to ship STDOUT and STDERR from the containers it manages to a
variety of services, like Syslog, Amazon CloudWatch, and Splunk. You can see the entire list of them by searching
for "Docker logging drivers" or by visiting the [Docker page](https://docs.docker.com/config/containers/logging/configure/) directly.
Considerations for when and how to use these will be covered later. For now, the important thing to know is that
the default is almost always JSON.

## The Docker Socket

When considering what people use most often, Docker is composed of two major pieces, the daemon, and the CLI.

The daemon is the part of Docker that runs and manages your containers throughout their life cycle. It does this
by making system calls to the Linux kernel. The daemon itself is controlled by a ReSTful interface served on the
Docker socket. This socket is just a Unix socket for local network traffic.

The CLI is what interfaces with the daemon. In it's most basic form, the CLI maps user input to ReST API calls.
By default the CLI will route traffic to the local Docker socket, but it can be configured to point ot a properly
configured remote Docker daemon. If you have ever used Docker Machine, you'll probably have seen evidence of this.

Many of the tools that will be discussed later use [this API](https://docs.docker.com/engine/api/latest/), and
more specifically, the logs endpoint. From a Linux machine with curl and Docker installed, you can issue GET request
to the Docker socket to see the logs of a container. This is similar to what happens when you run a `docker logs`
command.

<!-- Illustration here. -->

This is all a bit over simplified, but it should help clear up some of the tools that are covered later.

# Approaches to Unified Log Aggregation

At the highest level, you can divide up log aggregation into two categories - container level
or Host Level.

## Container Level

Managing logs at the containers level, is the least complex, and least rewarding approach. Ultimately with the
container level approach, you have the option to let Docker do it for you through logging drivers, or you can
do it yourself.

### Docker Logging Drivers

Logging drivers were touched on above, so here we'll just talk about when they do and don't make sense.

In my experience, logging drivers are a great place to start if you only have a few services you manage, or if
have no logging strategy in place yet, but you have a platform you can leverage, such as AWS CloudWatch or
a Splunk setup. To use them, you add a few flags to your Docker run command, and Docker will forward the output
to the specified platform. Below is an example of using AWS CloudWatch. It's one of the simplest to take
advantage of.

<!-- Illustration here. -->

There are quite a few logging drivers, and the list gets longer from time-to-time, so I won't list them all here.
To see the full list, and the customizations available for each one. Check out
[Dockers documentation](https://docs.docker.com/config/containers/logging/configure/) for more info.

So here is where they don't make sense. Think of the scenario where you have 10s, or even 100s. Let's say your
company decides to change it's internal logging infrastructure, or you decide to move regions in your cloud
provider. Now for each one of those services configured, you have to update the log driver options. It's possible
to automate this task, but it's going to take time all the same.

<!-- Illustration here. -->

The other place I've had things break on me is when the configured endpoint is down. If a container is already
running, Docker will keep some fixed buffer of your logs, but you will be unable to view them until that endpoint
comes back up and starts receiving logs. If a container is restarted, Docker will not start that container at all.

<!-- Illustration here. -->

Although these drawbacks can be quite a burden, logging drivers really are worth exploring if they fit your needs.
If you have only a few services running in Docker, and your logging infrastructure is pretty stable, logging drivers
are probably a good candidate for you. If you do have larger scale clusters, we'll talk about alternative methods
in the Host Level section.

### Volumes

You probably don't want to do this.

In practice, no matter what we do, some applications will not log to STDOUT in a sane way. This is somewhat common
with vendor applications. In cases like this, volumes are usable, but they come with some considerations, like
portability. Essentially your host would have volumes configured, and some logging infrastructure would pick up
those files. I'm not going to get into this solution too much here. Hopefully it's not common.

<!-- Please Don't Illustration here. -->

### In Code

If you must? \/0.o\/

It sounds crazy (and it is), but I've seen multiple projects where people insist on writing there own logging driver
that funnels logs to Splunk or something. If you are on those projects, I'm sorry.

<!-- Please Don't Illustration here. -->

## Host Level

In my experience, host level logging is really where things start to get interesting. By configuring Docker itself
or a third party tool, you can get all of your services running in your cluster to funnel logs to the same tool. If
your team is already using automation to build new host, then operators can modify the startup script to setup log
aggregation on the host, and all services will take advantage of it by default.

This comes with a lot of benefits. Below is a list of a few that really stand out to me.

* Great for building internal platforms as a service
* Many of the third party tools cache logs so when your log infrastructure is down, your services can still come up, unlike logging drivers.
* You can set things up to always have the logs on disk in a worse case scenario, meaning `docker logs` will still work
* Many third party logging tools can add Docker metadata to log events
* You only need to configure log aggregation in one place
* Having centralized logging infrastructure can help enforce standards
* If your company changes logging infrastructure, reconfiguring nodes should be easier than reconfigured services
* Host level can still be overridden by logging drivers or a sidecar, for those snowflakes

To get all of these benefits, there are a few drawbacks, however small they are. There is no free lunch in computer
science, after all.

* More disk space is required, but this can be limited
* More complex to setup initially
* The infrastructure is now more complex
* In the case of some third party tools the Docker socket is mounted, and this can be a security concern

<!-- transition later -->

### Logging Drivers

By default, the Docker daemon streams logs to JSON files on disk. This is because the default logging driver is
the JSON file log driver. Docker gives us the ability to configure the default logging driver when the daemon
starts. This is done by modifying the daemon.json file, typically located at `/etc/docker/daemon.json`. Add to
this file the same configuration you would add to your service, and restart Docker. Now every new container to
start will use that configuration as the default.

The one drawback with configuring the default logging driver is still that your containers cannot start if the
logging infrastructure is not receiving traffic. This may or may not be a concern though, depending on how reliable
your logging infrastructure is.

### Third Party Tool

Most of the benefits from having log aggregation configured at the host level really start to show when you add
the right third party tool. At the highest level, these tools get installed on the Docker host, watch the Docker
socket/container logs directory, process the log events, and ship them to one or more configured endpoints. If
your company has a centralized log infrastructure (such as ELK or Splunk), these tools can be a great way to
leverage them.

There are a lot of different tools out there, but I'll cover some of the more useful and common ones that I have
seen. Hopefully one of these tools will be useful for you, or they will give you an idea of why I like third party
tools so much.

For the next several examples, the logs will be streamed to an ELK stack that is running in containers. It is far
from a production ready ELK stack, but it will help illustrate what can be done with third party tools, and it's
open source. You can find the [code for running it here](https://github.com/bradwhitfield/logging-talk/tree/master/docker-elk).

**Note:** If you use a third party tool, you should configure your logging driver to JSON file, but with a
reasonable max size so you avoid filling the disk if a container starts writing too many log messages.

#### Logspout



#### Filebeat

#### Fluentd

#### Splunk
