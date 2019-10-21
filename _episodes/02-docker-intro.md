---
title: "Docker Crash Course Part 1: Images and Containers"
teaching: 10
exercises: 0
questions:
- "What's a docker container, and how is it related to a docker image?"
- "How do I pull docker images onto my machine and see which ones I've already pulled?"
objectives:
- "Get an idea of what a docker container is and what it can offer"
- "Practice pulling docker containers"
keypoints:
- "Containers offer a quick and accessible way to customize your environment!"
---

# Introduction

This docker crash course is basically a selection of material from an [awesome docker intro tutorial given by Matthew Feickert](https://matthewfeickert.github.io/intro-to-docker/) at the 2019 US ATLAS/First-HEP computing bootcamp at LBNL, where the material chosen is intended to provide the absolute minimum docker background needed to get started with RECASTing an analysis. I highly encourage anyone interested in getting a more complete docker intro to check out Matthew's full tutorial (linked above). 

# What's a Docker Container?

Docker images are executables that bundle together all necessary components for an application or an environment on any host machine. [Docker containers](https://www.docker.com/resources/what-container) are the runtime instances of images — they are images with a state.

Importantly, containers share the host machine’s OS system kernel and so don’t require an OS per application. As discrete processes containers take up only as much memory as necessary, making them very lightweight and fast to spin up to run.


[![Docker structure](https://www.docker.com/sites/default/files/styles/large/public/container-what-is-container.png)](https://www.docker.com/resources/what-container)

# Pulling Images

## Docker Hub

Much like GitHub allows for web hosting and searching for code, the [Docker Hub](https://hub.docker.com/)
image registry allows the same for Docker images.
Hosting and building of images is free for public repositories and
allows for downloading images as they are needed.

## Image Pulling

To begin with we're going to [pull](https://docs.docker.com/engine/reference/commandline/pull/) down the Docker image we're going
to be working in for the tutorial

~~~bash
docker pull matthewfeickert/intro-to-docker
~~~


and then [list the images](https://docs.docker.com/engine/reference/commandline/images/) that we have available to us locally

~~~bash
docker images
~~~

You can see here that there is the `TAG` field associated with the
`matthewfeickert/intro-to-docker` image.
Tags are way of further specifying different versions of the same image.
As an example, let's pull the buster release tag of the
[Debian image](https://hub.docker.com/_/debian).


~~~bash
docker pull debian:buster
docker images debian
~~~


> ## Pulling Python
>
> Pull the image for Python 3.7 (image name is python, tag is 3.7) and then list all the python docker images currently present on your computer.
>
> > ## Solution
> >
> > ~~~bash
> > docker pull python:3.7
> > docker images python
> > ~~~
> >
> > ~~~
> > REPOSITORY                        TAG                 IMAGE ID            CREATED             SIZE
> > python                            3.7                 e440e2151380        23 hours ago        918MB
> > ~~~
> > {: .output}
> {: .solution}
{: .challenge}




{% include links.md %}
