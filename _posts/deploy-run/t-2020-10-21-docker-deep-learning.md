---
layout: post
title: "Docker 101"
categories: [deploy & run]
tags: [bash, ide, linux, docker, "deploy & run"]
icon-photo: docker.svg
keywords: "pybash tania rascia CI CD continuous integration deployment pipeline docker idea how to use airflow kubernetes k8s k apache container images dangling images vscode vsc visual studio code ssh container env environnement file variable"
---

{% assign img-url = '/img/post/deploy/docker' %}

{% include toc.html %}

## What and Why?

{% hsbox Intuitive images %}
{:.img-80}
![What's docker]({{img-url}}/docker.png)
_Souce [rollout.io](https://rollout.io/blog/is-docker-secure/)._

{:.img-100}
![Container  vs Virtual Machine]({{img-url}}/vm-vs-docker.png)
_Container  vs Virtual Machine, souce [docker.com](https://www.docker.com/resources/what-container)._

{:.img-80}
![RAM usage: Docker  vs Virtual Machine]({{img-url}}/ram_docker_vm.png)
_RAM usage: Docker  vs Virtual Machine, souce [eureka.com](https://www.edureka.co/blog/what-is-docker-container)._

{% endhsbox %}

## Abbreviate

<div class="two-columns-list" markdown="1">
- `ps` = process status
- `-i` = interactive
- `-t` = terminal
- `-m` = memory
</div>

## Installation

For all platforms, check [this](https://docs.docker.com/get-docker/).

### Linux

{:.noindent}
- For Linux, check [this](https://docs.docker.com/engine/install/)!
    <div class="hide-show-box">
    <button type="button" markdown="1" class="btn collapsed box-button" data-toggle="collapse" data-target="#box1ct">
    Show codes
    </button>
    <div id="box1ct" markdown="1" class="collapse multi-collapse box-content">
    ``` bash
    # uninstall old versions
    sudo apt-get remove docker docker-engine docker.io containerd runc
    sudo apt-get update
    sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

    # make sure: 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
    sudo apt-key fingerprint 0EBFCD88
    sudo add-apt-repository \
      "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
      stable"

    # install docker engine
    sudo apt-get update
    sudo apt-get install docker-ce docker-ce-cli containerd.io

    # check if everything is ok
    sudo docker run hello-world

	# incase docker-compose isn't installed
	sudo apt install docker-compose
    ```
    </div>
    </div>

	If you use Ubuntu 20.04+, replace `$(lsb_release -cs)` with `eoan` because docker currently (17 May 20) doesn't support 20.04 yet!

- If wanna run docker without `root`, check [this](https://docs.docker.com/engine/install/linux-postinstall/).
  ``` bash
  sudo groupadd docker # create a docker group
  sudo usermod -aG docker <user> # add <user> to group
  newgrp docker # activate the changes
  ```
- Configure docker start on boot (Ubuntu 15.04 or later)
  ``` bash
  sudo systemctl enable docker
  ```

### MacOS

Check [this](https://docs.docker.com/docker-for-mac/install/).

### Windows

If meet the error `Failed to construct a huffman tree using the length array. The stream might be corrupted.`

{:.noindent}
1. You must have Windows 10: Pro, Enterprise, or Education (Build 15063 or later). Check [other requirements](https://docs.docker.com/docker-for-windows/install/#what-to-know-before-you-install).
  ~~~ bash
# POWERSHELL
# check window version
Get-WmiObject -Class Win32_OperatingSystem | % Caption
# check window build number
Get-WmiObject -Class Win32_OperatingSystem | % Buildnumber
  ~~~
2. Active **Hyper-V** and **Containers** (you can do it manually in **Turn Windows features on or off**)
  ~~~ bash
# Open PowerShell with Administrator and run following
Enable-WindowsOptionalFeature -Online -FeatureName containers ???All
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V ???All
# restart
  ~~~
1. [Download](https://docs.docker.com/docker-for-windows/install/) and install.
2. Check `docker version`.
3. Try `docker run hello-world`.

### Make NVIDIA work in docker (Linux)

**Idea**: Using NVIDIA driver of the base machine, don't install anything in docker!

{:.noindent}
1. First, [maker sure](/pytorch#installation) your base machine has an NVIDIA driver.

	``` bash
	# list all gpus
	lspci -nn | grep '\[03'

	# check nvidia & cuda versions
	nvidia-smi
	```
2. Install [`nvidia-container-runtime`](https://github.com/NVIDIA/nvidia-container-runtime)

	``` bash
	curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | sudo apt-key add -
	distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

	curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list

	sudo apt-get update

	sudo apt-get install nvidia-container-runtime
	```
3. Note that, <mark markdown='span'>we cannot use `docker-compose.yml` in this case!!!</mark>
4. Create an image `img_datas` with `Dockerfile` is

	``` docker
	FROM nvidia/cuda:10.2-base

	RUN apt-get update && \
		apt-get -y upgrade && \
		apt-get install -y python3-pip python3-dev locales git

	# install dependencies
	COPY requirements.txt requirements.txt
	RUN python3 -m pip install --upgrade pip && \
		python3 -m pip install -r requirements.txt
	COPY . .

	# default command
	CMD [ "jupyter", "lab", "--no-browser", "--allow-root", "--ip=0.0.0.0"  ]
	```
5. Create a container,

	``` bash
	docker run --name docker_thi --gpus all -v /home/thi/folder_1/:/srv/folder_1/ -v /home/thi/folder_1/git/:/srv/folder_2 -dp 8888:8888 -w="/srv" -it img_datas

	# -v: volumes
	# -w: working dir
	# --gpus all: using all gpus on base machine
	```

[This article](https://towardsdatascience.com/how-to-properly-use-the-gpu-within-a-docker-container-4c699c78c6d1) is also very interesting and helpful in some cases.

## Uninstall

### Linux

``` bash
# from docker official
sudo apt-get remove docker docker-engine docker.io containerd runc
```

``` bash
# identify what installed package you have
dpkg -l | grep -i docker

# uninstall
sudo apt-get purge -y docker-engine docker docker.io docker-ce docker-ce-cli
sudo apt-get autoremove -y --purge docker-engine docker docker.io docker-ce
```

``` bash
# remove images containers
sudo rm -rf /var/lib/docker /etc/docker
sudo rm /etc/apparmor.d/docker
sudo groupdel docker
sudo rm -rf /var/run/docker.sock
```

## Login & Download images

``` bash
docker login
# using username (not email) and password
```

{:.noindent}
- Download at [Docker Hub](https://hub.docker.com/).
- Download images are store at `C:\ProgramData\DockerDesktop\vm-data` (Windows) by default.

## Check info

``` bash
# docker's version
docker --version
```

### Images


<div class="flex-50" markdown="1">
``` bash
# list images on the host
docker images
```

``` bash
# check image's info
docker inspec <image_id>
```
</div>

### Containers

<div class="flex-50" markdown="1">
~~~ bash
# list running containers
docker ps
docker ps -a # all (including stopped)
~~~

~~~ bash
# only the ids
docker ps -q
docker ps -a -q
~~~

``` bash
# container's size
docker ps -s
docker ps -a -s
```

``` bash {% raw %}
# container's names only
docker ps --format '{{.Names}}'
docker ps -a --format '{{.Names}}'
{% endraw %}```

``` bash {% raw %}
# Check the last command in container
docker ps --format '{{.Command}}' --no-trunc
{% endraw %}```

``` bash
# check log
# useful if we wanna see the last running tasks's
docker container logs <container_name>
```

``` bash
# get ip address
docker inspect <container_name> | grep IPAddress
```
</div>

### Others

<div class="flex-50" markdown="1">
``` bash
# RAM & CPU usages
docker stats
docker stats <container_name>
```
</div>

## Attach / Start / Stop

We can use sometimes interchangeable between `<container_id>` and `<container_name>`.

<div class="flex-50" markdown="1">
~~~ bash
# get info (container's id, image's id first)
docker ps -a
~~~

~~~ bash
# start a stopped container
docker start <container_id>
~~~

~~~ bash
# stop a container
docker stop <container_id>
~~~

~~~ bash
# going to running container env
docker exec -it <container_name> bash
~~~

``` bash
# stop all running containers
docker stop $(docker ps -a -q)
```
</div>

## Delete

Read more [here](https://www.digitalocean.com/community/tutorials/how-to-remove-docker-images-containers-and-volumes).

### Everything
<div class="flex-50" markdown="1">
~~~ bash
# any resources
docker system prune
~~~

~~~ bash
# with all unused images
docker system prune -a
~~~
</div>

### Images

<div class="flex-50" markdown="1">
~~~ bash
# list all images
docker images -a
~~~

~~~ bash
# remove a specific image
docker image rm <IMAGE_ID>
~~~
</div>

**Dangling images** are layers that have no relationship to any tagged images.

<div class="flex-50" markdown="1">
~~~ bash
# list dangling images
docker images -f dangling=true
~~~

~~~ bash
# remove dangling images
docker images purge
~~~
</div>

### Containers

<div class="flex-50" markdown='1'>
``` bash
# remove a specific containers
docker rm -f <container-id>
```

``` bash
# remove all containers
docker rm -f $(docker ps -a -q)
```
</div>

## Build an image

### Create

<div class="flex-50" markdown="1">
~~~ bash
# build image with Dockerfile
docker build -t <img_name> .

# custom Dockerfile.abc
docker build -t <img_name> . -f Dockerfile.abc
~~~

~~~ bash
# with docker-compose
docker-compose up
~~~

~~~ bash
# if success
# service name "docker_thi"
docker run -it <service_name> bash
~~~

``` bash
# from current container
docker ps -a # check all containers
docker commit <container_id> <new_image_name>
```
</div>

### `Dockerfile`

``` bash
FROM nvidia/cuda:10.2-base

# encoding
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# fix (tzdatachoose)
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get -y update && \
    apt-get -y upgrade && \
    apt-get install -y openssh-server && \
    apt-get install -y python3-pip python3-dev locales git r-base

# ssh server
RUN mkdir /var/run/sshd
RUN echo 'root:qwerty' | chpasswd
RUN sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
# SSH login fix. Otherwise user is kicked off after login
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
# need?
ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

# create alias
RUN echo 'alias python="python3"' >> ~/.bashrc
RUN echo 'alias pip="pip3"' >> ~/.bashrc

# create shortcuts
RUN ln -s /abc/xyz /xyz/xyz

# install python's requirements
COPY requirements_dc.txt requirements.txt
RUN python3 -m pip install --upgrade pip && \
    python3 -m pip install -r requirements.txt
COPY . .

# export port ssh
EXPOSE 22

COPY script.sh starting_script.sh
# run
CMD ["sh","-c","cd /data_controller/utils/ && sh generate_grpc_code_from_protos.sh && cd /srv/ && sh starting_script.sh"]
```

<div class="hide-show-box">
<button type="button" markdown="1" class="btn collapsed box-button" data-toggle="collapse" data-target="#box1ct_ssh">
Connect to container via ssh
</button>
<div id="box1ct_ssh" markdown="1" class="collapse multi-collapse box-content">
``` bash
FROM ubuntu:20.04

RUN apt-get update && apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:THEPASSWORDYOUCREATED' | chpasswd
RUN sed -i 's/#*PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config

# SSH login fix. Otherwise user is kicked off after login
RUN sed -i 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```

For more about ssh connection, check [official doc](https://docs.docker.com/engine/examples/running_ssh_service/).

If `.env` doesn't work? => This is expected. SSH wipes out the environment as part of the login process. {% ref https://github.com/moby/moby/issues/2569#issuecomment-27973910 %}

``` bash
# For example, all environement variables are stored in a
# /home/thi/.env

# add them to container's env
cat /home/thi/.env >> /etc/environment

# exit current ssh session
# connect again

# check
env
```
More references: [here](https://askubuntu.com/questions/58814/how-do-i-add-environment-variables) and [here](https://stackoverflow.com/questions/34630571/docker-env-variables-not-set-while-log-via-shell).
</div>
</div>

<div class="hide-show-box">
<button type="button" markdown="1" class="btn collapsed box-button" data-toggle="collapse" data-target="#box1ct_dockerfile">
Meaning of terms
</button>
<div id="box1ct_dockerfile" markdown="1" class="collapse multi-collapse box-content">
- `FROM`{:.tpink}: the base image you use, can be obtained from [Docker Hub](https://hub.docker.com/). For example, `FROM ubuntu:18.04` (`18.04` is a tag, `latest` is default)
- `WORKDIR app/`{:.tpink}: Use `app/` as the working directory.
- `RUN`{:.tpink}: install your application and packages requited, e.g. `RUN apt-get -y update`.
    - `RUN <command>` (shell form)
	- `RUN ["executable", "param1", "param2"]` (exec form)
- `CMD`{:.tpink}: sets default command and/or parameters, which can be overwritten if docker container runs with command lines. If there are many `CMD`s, the last will be used.
	<div class="hide-show-box">
	<button type="button" markdown="1" class="btn collapsed box-button" data-toggle="collapse" data-target="#box2ct">
	Example
	</button>
	<div id="box2ct" markdown="1" class="collapse multi-collapse box-content">
	```` bash
	# in Dockerfile
	CMD echo "Hello world"
	# run only
	docker run -it <image>
	# output: "Hello world"

	# run with a command line
	docker run -it <image> /bin/bash
	# output (CMD ignored, bash run instead): root@7de4bed89922:/#
	````
	</div>
	</div>
	- `CMD ["executable","param1","param2"]` (exec form, preferred)
	- `CMD ["param1","param2"]` (sets additional default parameters for `ENTRYPOINT` in exec form)
	- `CMD command param1 param2` (shell form)
	- Multiple commands: `CMD ["sh","-c","mkdir abc && cd abc && touch new.file"]`
- `ENTRYPOINT`{:.tpink}: configures a container that will run as an executable. Look like CMD but ENTRYPOINT command and parameters are not ignored when Docker container runs with command line parameters.
	<div class="hide-show-box">
	<button type="button" markdown="1" class="btn collapsed box-button" data-toggle="collapse" data-target="#box3ct">
	Example
	</button>
	<div id="box3ct" markdown="1" class="collapse multi-collapse box-content">
	```` bash
	# Dockerfile
	ENTRYPOINT ["/bin/echo", "Hello"]
	CMD ["world"]
	# run
	docker run -it <image>
	# produces 'Hello world'

	# but run
	docker run -it <image> John
	# produces 'Hello John'
	````
	</div>
	</div>
	- `ENTRYPOINT ["executable", "param1", "param2"]` (exec form, preferred)
	- `ENTRYPOINT command param1 param2` (shell form)
- `EXPOSE 5000`{:.tpink}: Listen on the specified port
- `COPY . app/`{:.tpink}: Copy the files from the current directory to `app/`
</div>
</div>

{% hsbox Example of run mutiple commands %}
If we run multiple interative commands (wait for action), for example, a jupyter notebook with a ssh server, we cannot put them directly in `CMD` command.

__Solution__: using a file .sh and put `&` at the end of the 1st command like:

``` bash
$(which sshd) -Ddp 22 &

jupyter lab --no-browser --allow-root --ip=0.0.0.0 --NotebookApp.token='' --NotebookApp.password=''
```
{% endhsbox %}

## Create a container

### CLI

``` bash
# container test from an image
docker create --name container_test -t -i <image_id> bash
docker start container_test
docker exec -it container_test bash
```

``` bash
docker run  --name <container_name> -dp 3000:3000 -v todo-db:/etc/todos <docker_img>
```

``` bash
# run a command in a running docker without entering to that container
# e.g. running "/usr/sbin/sshd -Ddp 22"
docker exec -it -d docker_thi_dc /usr/sbin/sshd -Ddp 22
# "-d" = Detached mode
```

### `docker-compose.yml`

Use to create various services with the same image.

~~~ bash
# docker-compose.yml
#------------------------------

# run by `docker-compose up`
version: '3'
services:
  dataswati:
    container_name: docker_thi
    image: docker_thi_img:latest
    ports:
      - "8888:8888"
    volumes:
      - "/local-folder/:/docker-folder/"
	working_dir: /srv
~~~

<div class="hide-show-box">
<button type="button" markdown="1" class="btn collapsed box-button" data-toggle="collapse" data-target="#box1ct_cmpose">
Meaning of terms
</button>
<div id="box1ct_cmpose" markdown="1" class="collapse multi-collapse box-content">
- `depends_on`{:.tpink}:{% ref https://docs.docker.com/compose/compose-file/#depends-on %} Express dependency between services (with orders).
	<div class="hide-show-box">
	<button type="button" markdown="1" class="btn collapsed box-button" data-toggle="collapse" data-target="#box4ct">
	Example
	</button>
	<div id="box4ct" markdown="1" class="collapse multi-collapse box-content">
	In `docker-compose.yml`: `db` and `redis` start before `web`.

	```` yaml
	version: "3.8"
	services:
	web:
		build: .
		depends_on:
		- db
		- redis
	redis:
		image: redis
	db:
		image: postgres
	````
	</div>
	</div>
- `command`{:.tpink}:{% ref https://docs.docker.com/compose/compose-file/#command %} Override the default command.
- `stdin_open: true`{:.tpink} and `tty: true`{:.tpink}: keep container alive!
- `volumes`{:.tpink} (outside containers): volumes controlled by docker. They're located on different places,
``` bash
# check info of a volume
docker volume inspect <volume_name>
```
- `restart: always`{:.tpink} (auto start container after logging in), `no` (default), `on-failure`.
</div>
</div>

## Errors

_Docker can't connect to docker daemon`_.

``` bash
# check if daemon is running?
ps aux | grep docker

# run
sudo /etc/init.d/docker start
```

---

`sudo systemctl restart docker`  meets _Job for docker.service failed because the control process exited with error code_.

1. Try to remove failed `daemon.json` file in `/etc/docker/` (if the problem comes from here)
2. Try running either `sudo /etc/init.d/docker start` or `sudo service docker restart` (twice if needed).

---

_perl: warning: Please check that your locale settings:_ when using below in the Dockerfile,

``` bash
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Replace them by
RUN echo "LC_ALL=en_US.UTF-8" >> /etc/environment
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
RUN echo "LANG=en_US.UTF-8" > /etc/locale.conf
RUN locale-gen en_US.UTF-8
```



## Reference

- **Play with Docker** -- [right on the web](https://labs.play-with-docker.com/).
- **Yury Pitsishin** -- [Docker RUN vs CMD vs ENTRYPOINT](https://goinbigdata.com/docker-run-vs-cmd-vs-entrypoint/).