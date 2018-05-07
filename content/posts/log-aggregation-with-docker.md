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

This is all a bit over simplified, but it should help clear up some of the tools that are covered later.

# Approaches to Unified Log Aggregation

At the highest level, you can divide up log aggregation into two categories - container level
or centralized tool level.

## Container level

Managing logs at the containers level, is the least complex, and least rewarding approach.

Logspout Stuff
https://docs.docker.com/engine/api/latest/
