---
title: "Docker Crash Course Part 2: Running and Building Containers"
teaching: 20
exercises: 0
questions:
- "How can I customize docker images and run them as containers on my machine?"
- "How do I give a running docker container access to files and directories on my machine?"
objectives:
- "Understand how containers help you encapsulate your environment"
- "Practice creating and running docker containers"
keypoints:
- "Dockerfiles let you to customize a base docker image to get the exact environment you want"
- "Volume-mounting lets you specify the exact files and directories on your machine that you want a running container to have access to"
---

# Running containers

To use a Docker image as a particular instance on a host machine you [run](https://docs.docker.com/engine/reference/run/)
it as a container.
You can run in either a [detached or foreground](https://docs.docker.com/engine/reference/run/#detached-vs-foreground) (interactive) mode.

Run the image we pulled as an interactive container

~~~bash
docker run -it matthewfeickert/intro-to-docker:latest /bin/bash
~~~

You are now inside the container in an interactive bash session. Check the file directory

~~~bash
pwd
~~~

~~~
/home/docker/data
~~~
{: .output}

and check the host to see that you are not in your local host system

~~~bash
hostname
~~~

## Monitoring Containers

Open up a new terminal tab on the host machine and
[list the containers that are currently running](https://docs.docker.com/engine/reference/commandline/ps/)

~~~bash
docker ps
~~~

~~~
CONTAINER ID        IMAGE         COMMAND             CREATED             STATUS              PORTS               NAMES
<generated id>      <image:tag>   "/bin/bash"         n minutes ago       Up n minutes                            <generated name>
~~~
{: .output}


## Volume Mounting

You can make files and directories accessible to the container by [mounting them as volumes](https://docs.docker.com/storage/volumes/) to the
container with the `-v` flag.
This allows for direct access to the host file system inside of the container and for
container processes to write directly to the host file system.

~~~bash
docker run -v <path on host>:<path in container> <image>
~~~

For example, to mount your current working directory on your local machine to the `data`
directory in the example container

~~~bash
mkdir intro_to_docker
chmod -R a+w intro_to_docker    # Give all users permission to write in this directory
cd intro_to_docker
touch made_on_machine.txt
docker run --rm -it -v $PWD:/home/docker/data matthewfeickert/intro-to-docker
~~~

From inside the container you can `ls` to see the contents of your directory on your local
machine

~~~bash
ls
~~~

and yet you are still inside the container

~~~bash
pwd
~~~

~~~
/home/docker/data
~~~
{: .output}

You can also see that any files created in this path in the container persist upon exit

~~~bash
touch created_inside.txt
exit
ls *.txt
~~~

~~~
created_inside.txt
~~~
{: .output}

This I/O allows for Docker images to be used for specific tasks that may be difficult to
do with the tools or software installed on only the local host machine.
For example, debugging problems with software that arise on cross-platform software, or
even just having a specific version of software perform a task (e.g., using Python 2 when
you don't want it on your machine, or using a specific release of
TeX Live when you aren't ready to update your system release).

> ## CMD Dockerfile Instruction
>
> If you're super observant, you may have noticed that the startup command `/bin/bash` was used to start the container in a bash shell when we first ran the `matthewfeickert/intro-to-docker:latest`, but the second time we ran the container (when we added the `-v $PWD:/home/docker/data`), we didn't specify a startup command. How did the container still know to start in a bash shell? 
> 
> No, docker doesn't have mind-reading functionality (yet...). Rather, this behaviour comes from the fact that the [Dockerfile used to create this image](https://hub.docker.com/r/matthewfeickert/intro-to-docker/dockerfile) includes an instruction `CMD ["/bin/bash"]` at the end, where the [`CMD` command](https://docs.docker.com/engine/reference/builder/#cmd) specifies that the container should start up in the bash shell by default if no other startup command is specified in the `docker run` command. If another startup command is specified, like 
>
> ~~~bash
> docker run -it matthewfeickert/intro-to-docker:latest python
> ~~~
>
> then this will override the default set by the `CMD` option, so the container will in this case start up with a python command-line prompt instead (try it out!).
>
{: .callout}


# Writing Dockerfiles to Build Images

Docker images are built through the Docker engine by reading the instructions from a
[`Dockerfile`](https://docs.docker.com/engine/reference/builder/).
These text based documents provide the instructions though an API similar to the Linux
operating system commands to execute commands during the build.
The [`Dockerfile` for the example image](https://hub.docker.com/r/matthewfeickert/intro-to-docker/dockerfile) being used is an example of
some simple extensions of the [official Python 3.6.8 Docker image](https://hub.docker.com/layers/python/library/python/3.6.8/images/sha256-d5028edbd2793f03125e76c0519b837306b63d7835efd8e7aa62b9d46126a495).

As a very simple way of extending the example image into a new image is to create a `Dockerfile`
on your local machine

~~~bash
touch Dockerfile
~~~

and then write in it the Docker engine instructions to add `cowsay` and
`scikit-learn` to the environment

~~~yaml
# Dockerfile

# Specify the base image that we're building the image on top of
FROM matthewfeickert/intro-to-docker:latest

# Build the image as root user
USER root

# Run some bash commands to install packages
RUN apt-get -qq -y update && \
    apt-get -qq -y upgrade && \
    apt-get -qq -y install cowsay && \
    apt-get -y autoclean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt-get/lists/* && \
    ln -s /usr/games/cowsay /usr/bin/cowsay
RUN pip install --no-cache-dir -q scikit-learn

# This sets the default working directory when a container is launched from the image
WORKDIR /home/docker

# Run as docker user by default when the container starts up
USER docker
~~~

> ## Dockerfile layers
>
>Each `RUN` command in a Dockerfile creates a new layer to the Docker image.
>In general, each layer should try to do one job and the fewer layers in an image
> the easier it is compress. When trying to upload and download images on demand the
> smaller the size the better.
{: .callout}

> ## Don't run as `root`
>
>By default Docker containers will run as `root`. This is a bad idea and a security concern.
>Instead, setup a default user (like `docker` in the example) and if needed give the user
>greater privileges.
{: .callout}

Then [`build`](https://docs.docker.com/engine/reference/commandline/build/) an image from the `Dockerfile` and tag it with a human
readable name

~~~bash
docker build -f Dockerfile -t extend-example:latest .
~~~

You can now run the image as a container and verify for yourself that your additions exist

~~~bash
docker run --rm -it extend-example:latest /bin/bash
which cowsay
cowsay "Hello from Docker"
pip list | grep scikit
python3 -c "import sklearn as sk; print(sk)"
exit
~~~


~~~
/usr/bin/cowsay
 ___________________
< Hello from Docker >
 -------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||

scikit-learn       0.21.3
<module 'sklearn' from '/usr/local/lib/python3.6/site-packages/sklearn/__init__.py'>
~~~
{: .output}

## `COPY`

Docker also gives you the ability to copy external files into a Docker image during the
build with the [`COPY`][docker-docs-COPY] Dockerfile command.
Which allows copying a target file from a host file system into the Docker image
file system

~~~yaml
COPY <path on host> <path in Docker image>
~~~
{: .source}

For example, if there is a file called `install_python_deps.sh` in the same directory as
the build is executed from

~~~bash
touch install_python_deps.sh
~~~
{: .source}

with contents

~~~bash
cat install_python_deps.sh
~~~
{: .source}

~~~
#!/usr/bin/env bash

set -e

pip install --upgrade --no-cache-dir pip setuptools wheel
pip install --no-cache-dir -q scikit-learn
~~~
{: .output}

then this could be copied into the Docker image of the previous example during the build
and then used (and then removed as it is no longer needed). 

Create a new file called `Dockerfile.copy`:

~~~bash
touch Dockerfile.copy
~~~
{: .source}

and fill it with a modified version of the above Dockerfile, where we now copy `install_python_deps.sh` from the local working directory into the container and use it to install the specified python dependencies:

~~~yaml
# Dockerfile.copy
FROM matthewfeickert/intro-to-docker:latest
USER root
RUN apt-get -qq -y update && \
    apt-get -qq -y upgrade && \
    apt-get -qq -y install cowsay && \
    apt-get -y autoclean && \
    apt-get -y autoremove && \
    rm -rf /var/lib/apt-get/lists/* && \
    ln -s /usr/games/cowsay /usr/bin/cowsay
COPY install_python_deps.sh install_python_deps.sh
RUN bash install_python_deps.sh && \
    rm install_python_deps.sh
WORKDIR /home/data
USER docker
~~~
{: .source}

~~~bash
docker build -f Dockerfile.copy -t copy-example:latest .
~~~
{: .source}

For very complex scripts or files that are on some remote, `COPY` offers a straightforward
way to bring them into the Docker build.

[docker-docs-COPY]: https://docs.docker.com/engine/reference/builder/#copy

{% include links.md %}
