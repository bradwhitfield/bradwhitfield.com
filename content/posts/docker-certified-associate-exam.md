+++
Title = "Docker Certified Associate Exam"
Date = "2018-01-27T18:23:00-05:00"
draft = true
Description = "My thoughts on the Docker Certied Associate Exam (sorry, no questions from the test)."
Tags = ["Docker", "Exam", "DCA", "Certified", "Associate"]
Categories = ["Ops", "Docker"]
menu = "main"
comments = false
+++

Today I took and passed the Docker Certified Associate exam and I wanted to write up my thoughts on it (mostly just to make me write something).
The short version is that it's harder than I expected and it is well thought out.

# Background

I've been working with Docker professionally for just over two years. Most of this time has been spent maintaining Amazon ECS clusters. For a little while,
I was helping support a Docker Swarm cluster pre-SwarmKit. Recently, I've been given an opportunity to work with Kubernetes. Most of these
clusters have been part of a larger platform to support developers in a PaSS-like way so they can deploy their services with less friction.
With this, I've become quite familiar with Docker and the paradigms that come with it.

At work, I'm expected to be somewhat of an expert in Docker and the surrounding technologies, so when the exam was announced,
I figured it couldn't hurt to try for it.

# Thoughts on the Exam

As I mentioned at the top, the exam was tougher than I thought. I still did quite well, but before taking it, I was beginning to think I over-prepared.
My advantage going in is that I have been working with it for a while. Several of the questions I did not know the answer to off the top of my head,
but through the process of elimination, and having familiarity with other Docker tooling, I was often able to derive the answer.

Many of the questions were somewhat specific. I can't give examples, but you are expected to know how to do a process, and what the process is doing —
at least at a high level. There weren't any questions that dove into how the network works at the Linux Kernel level, or anything that deep, but if it's on
the study guide, you should have an idea of how to do the thing, and what is happening when it does that thing.

Based on the study guide, I was worried that I would have to be fairly proficient in DTR and UCP. Although these were definitely topics on my version of the
exam, it was not at the same ratio as what they have on the study guide. Don't take this as fact, though. Your exam could be different questions than mine,
so it's possible someone will have a ton of questions on these two. But either way, once you learn the rest of the concepts in the study guide, DTR and UCP
are pretty easy to get your head around.

Overall, I thought the exam was pretty well done, but it may not be for everyone. If you are using Docker to ship relatively simple apps on something
like Amazon ECS, or just using it for your development work, then this cert is probably overkill.

# Study Material

I used these two things.

* [DevOps Academy DCA Prep Guide](https://github.com/DevOps-Academy-Org/dca-prep-guide)
* [Linux Academy DCA Course](https://linuxacademy.com/linux/training/course/name/docker-certified-associate-prep-course)

Most of my studying was done with the top link (I started well before the Linux Academy course existed). Both were pretty good sources, but the one
caution I have with using any video training is to make sure you still read the appropriate documentation, and try the stuff out yourself. Linux Academy
hit almost everything that came up on my exam, but with the questions presented, I know just watching the videos and doing the labs would not have
been enough for me to remember some of the specifics required by the questions.

Everything on the exam was called out on the study guide Docker put out (which DevOps Academy mirrors). Make sure you understand all of the concepts
on the list before going into the exam. If you can speak to everything on there, and you can perform any of the processes they mention, you'll probably
be fine.

I'm not sure these notes will make sense to anyone else, but my notes are [available on github](https://github.com/bradwhitfield/docker-cert-study).

Good luck, to anyone who decides to take the exam!
