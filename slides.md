---
# try also 'default' to start simple
theme: frankfurt
author: Albert Bezman
title: Docker
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides, markdown enabled
# title: Welcome to Slidev
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# apply any unocss classes to the current slide
class: text-center
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# https://sli.dev/guide/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/guide/syntax#mdc-syntax
mdc: true
---

# Docker

### MLX4

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub" title="Open in GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# What is Docker?

<br>

- **NOT** a virtual machine
- A way to **isolate** processes using Linux kernel features
- A declarative, OS level deployment environment management
- I.e. Reproducible environments for software packages to run in

<br>
<br>

![Docker Logo](https://upload.wikimedia.org/wikipedia/commons/e/ea/Docker_%28container_engine%29_logo_%28cropped%29.png)


<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

<!--
Here is another comment.
-->

---

# What problems does it solve?


- Declarative dependency management
- Deployment environment reproducibility + consistency
- Host environment agnostic (so long as Docker engine is accessible)
- Application dependency conflicts
- Deployment + scaling
- Encourages loose coupling
- More efficient than a VM (which has to visualize all hardware)

---

# What is it under the hood?

<br>
Docker utilises a number of Linux kernel features:
<br>

| Feature     | Description                           |
| ----------- | ------------------------------------- |
| `namespaces`| Isolation                             |
| `cgroups`   | Resource management                   |
| `overlayFS` | Filesystem isolation (think 'chroot') |

---

# Namespaces

Linux kernel namespaces allow you to isolate various aspects of your system, like:
- processes `pid`
- network `net`
- ...

Let's make our very own container using namespaces!

<br>

````md magic-move
```sh {*|1|3}
sudo ls -l /proc/1/ns/

ps aux
```

```plaintext
docker/tutorial
❯ ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  1.6  0.0 167304 12756 ?        Ss   17:14   6:15 /sbin/init
root           2  0.0  0.0   2280  1304 ?        Sl   17:14   0:00 /init
root           6  0.0  0.0   2324   132 ?        Sl   17:14   0:00 plan9 --control-s
root          46  0.0  0.0  48004 15844 ?        S<s  17:14   0:00 /lib/systemd/syst
root          73  0.0  0.0  21972  5868 ?        Ss   17:14   0:02 /lib/systemd/syst
root          84  0.0  0.0   4496   180 ?        Ss   17:14   0:00 snapfuse /var/lib
```

```sh {*|2|4|}
# Lets isolate some processes
sudo unshare --fork --pid --mount-proc bash

ps aux
```

```plaintext
.../docker/tutorial# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   5048  3876 pts/15   S    23:28   0:00 bash
root           8  0.0  0.0   7484  3280 pts/15   R+   23:28   0:00 ps aux
```

```sh
exit
```

```sh
ip a
```

```plaintext
...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:83:03:57 brd ff:ff:ff:ff:ff:ff
    inet 172.24.143.239/20 brd 172.24.143.255 scope global eth0
```

```sh
# Let's isolate some network resources
sudo ip netns add mynetns
sudo ip netns exec mynetns bash
```

```sh
ip a
```

```plaintext
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
```

```sh {*|1|3}
exit

sudo ip netns del mynetns
```

````

---

# Cgroups

Cgroups are a way to control how much resources your isolated processes take up (CPU, RAM, ...)

````md magic-move
```sh
# Prerequisites
sudo apt-get update
sudo apt-get install cgroup-tools
```

```sh
sudo cgcreate -g cpu:/example_group
```

```sh
sudo cgset -r cpu.cfs_quota_us=20000 example_group
sudo cgset -r cpu.cfs_period_us=100000 example_group
```

```sh
sudo cgexec -g cpu:/example_group yes > /dev/null &
```

```sh
top
```

<arrow v-click="[4, 5]" x1="350" y1="310" x2="195" y2="334" color="#953" width="2" arrowSize="1" />

```plaintext {*|2}
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 146461 root      20   0    3208    988    900 R  19.7   0.0   0:03.57 yes
```

```sh {*|1|3}
sudo ps aux | grep '[y]es' | awk '{print $2}' | xargs kill

sudo cgdelete -g cpu:/example_group
```

````

<br>
<div v-click>
Process never exceeds 20% of CPU!
</div>

---

# OverlayFS

Allows multiple file system layers to be overlaid and presented as a single file system.

OverlayFS enables filesystem sharing that allows multiple containers to share the same base image while maintaining their own file system changes in seperate layers.

Let's demonstrate this:

````md magic-move
```sh
# Prepare directories
sudo mkdir /overlayfs
sudo mkdir /overlayfs/lower
sudo mkdir /overlayfs/upper
sudo mkdir /overlayfs/work
sudo mkdir /overlayfs/mount
```

```sh
echo "Hello from the lower layer" | sudo tee /overlayfs/lower/hello.txt
```

```sh
sudo mount -t overlay overlay -o lowerdir=/overlayfs/lower,upperdir=/overlayfs/upper,workdir=/overlayfs/work /overlayfs/mount
```

```sh
cat /overlayfs/mount/hello.txt
```

```sh
echo "Hello from the upper layer" | sudo tee /overlayfs/mount/upper_hello.txt
```

```sh
ls /overlayfs/upper/
```

```sh
sudo umount /overlayfs/mount
sudo rm -r /overlayfs
```

````

<br>
<div v-click>
Changes made in the overlay filesystem are written to the upper layer, making it easy to track changes or revert to the original state without modifying the original data. This functionality is key in scenarios like container runtime environments where you need to maintain base image integrity while allowing changes during runtime.
</div>

---

# Congratulations!

## You've just built Docker (sort of ...)

<br>

<img src="https://p.turbosquid.com/ts-thumb/mh/ElwSXh/FHC2kEVf/1/jpg/1327598786/600x600/fit_q87/2cc78ac6045f1792c8800efe356142f08dfe9fb0/1.jpg" width="300" height="400">

---

# Components

Docker composed of:

### 1. Docker Engine 
  <br>a) Daemon (can be remote) `dockerd`
  <br>   Listens for API requests and manages Docker objects such as containers
  <br>b) Client (can connect to remote server) e.g. `docker exec`
  <br>   Control Docker via API calls



---

# Components (continued)

### 2. Images
  <br>- A read-only (immutable) template composed of layered FS which contains all dependencies required to build a container
  <br>- An image is a read-only layer that is never modified, all changes are made in top-most writeable layer
  <br>- Built from ‘base’ image


<div v-click>

### 3. Containers
  <br> - Runnable, isolated instances of images
  <br> - When you launch a container from an image, a writeable layer is added on top of this image
  <br> - A set of linux namespaces with a limited view of the system
</div>

---

# Components (continued)

### 4. Docker registry
  <br> - Where Docker can source images from
  <br> - Can be public (e.g. Docker Hub https://hub.docker.com/)
  <br> - Can be private (e.g. your companies image registry)

<div v-click>

### 5. Docker-compose
  <br> - A way to combine containers and configure them (via `docker-compose.yml`)
</div>

<div v-click>

### 6. Docker Swarm
  <br> - Container orchestration for clusters
</div>

---

# Practical 

Prerequisites

1. Install Docker engine
<br>a) For Linux: https://docs.docker.com/engine/install/linux-install/
<br>b) For Windows: https://docs.docker.com/desktop/install/windows-install/
<br>c) For Mac: https://docs.docker.com/desktop/install/mac-install/

<br>
Either download Docker for your distribution if on Linux via a package manager (e.g. `apt install`), or download the desktop app for Mac/Windows. This will install docker engine and requisite tooling.

<br>
<br>

2. Sign up for docker hub: https://hub.docker.com/signup

---

# Practical

Basic workflow


1. Define your image via `Dockerfile` or `docker-compose.yml`.
<br>

2. Build the container via `docker build`
<br>

3. Run the container via `docker run`
<br>

4. Update the image, build, run, deploy

---

# Practical

Let's define a basic image and build it!


````md magic-move
```sh {2|3|4}
# Prepare directories
cd some-directory
mkdir docker_tut_1
cd docker_tut_1
```

```sh
# Create Dockerfile
touch Dockerfile
vim Dockerfile # OR open with your favourite editor
```

```docker
# Use an official base image from the Docker Hub
FROM ubuntu:latest

CMD ["echo", "Hello, Docker!"]
```


```sh
# Build your first image
docker build -t my-first-image . # -t flag tags the image with a label
```

````

<div v-click>
<br>
Let's now see that our container is available to `dockerd` locally:

```sh
docker images
```
</div>

<div v-click>

```plaintext
REPOSITORY       TAG       IMAGE ID       CREATED       SIZE
my-first-image   latest    f39894e3a078   7 days ago    77.9MB
```
</div>

<div v-click>
<br>
Can check the GUI app under 'images' section too.
</div>

---

# Practical

Let's run it!

````md magic-move
```sh
docker run my-first-image
```
```plaintext
$ docker run my-first-image
Hello, Docker!
```
````

<div v-click>
<br>
Let's see our container status

```sh
docker ps -a
```
</div>

<div v-click>

```plaintext
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS                        PORTS                      NAMES
021f5244985c   my-first-image   "echo 'Hello, Docker…"   3 minutes ago   Exited (0) 3 minutes ago                                 friendly_snyder
```
</div>

<div v-click>
<br>
Can check the GUI app under 'containers' section too.
</div>

---

# Practical

How do we update our image?

Dockerfiles composed of statements e.g. `FROM`, `RUN` ...
<br>
<br>Let's add a 'RUN' statement
<br>RUN executes a command in an FS layer. Each layer builds on the previous one.

````md magic-move
```docker
# Use an official base image from the Docker Hub
FROM ubuntu:latest

CMD ["echo", "Hello, Docker!"]
```

```docker {*|4}
# Use an official base image from the Docker Hub
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl

# Optional: Command to run when the container starts
CMD ["echo", "Hello, Docker!"]
```

```docker {*|6}
# Use an official base image from the Docker Hub
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl
# We can add as many RUN commands as we need
RUN apt-get install -y python3 python3-pip

# Optional: Command to run when the container starts
CMD ["echo", "Hello, Docker!"]
```
````

---

# Practical

`CMD` command is the entrypoint for all docker containers.
<br>There can only be 1 `CMD` in each Dockerfile.
<br>Let's modify our `CMD` statement to do something else.

````md magic-move
```docker
# Use an official base image from the Docker Hub
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl
# We can add as many RUN commands as we need
RUN apt-get install -y python3 python3-pip

# Optional: Command to run when the container starts
CMD ["echo", "Hello, Docker!"]
```


```docker {*|9}
# Use an official base image from the Docker Hub
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl
# We can add as many RUN commands as we need
RUN apt-get install -y python3 python3-pip

# Optional: Command to run when the container starts
CMD ["python3", "-c", "print('Hello World')"]
```
````

<v-click>

Let's test the changes:

````md magic-move

```sh
docker build -t my-first-image .
docker run my-first-image
```

```plaintext
=> [2/3] RUN apt-get update && apt-get install -y curl
=> [3/3] RUN apt-get install -y python3 python3-pip
```

```plaintext
docker run my-first-image
Hello World
```

````

</v-click>

---

# Practical

Hmmm, but what happened to our old image and container?
<br>
<br>
Let's check that with:

````md magic-move
```sh
docker images
```

```plaintext
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
my-first-image   latest    484411e8476f   21 minutes ago   480MB
<none>           <none>    f39894e3a078   7 days ago       77.9MB
```

```sh
docker ps -a
```

```plaintext
CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS                           PORTS                      NAMES
a43ccf28841a   my-first-image   "python3 -c 'print('…"   21 minutes ago   Exited (0) 21 minutes ago                                   vigilant_bouman
021f5244985c   f39894e3a078     "echo 'Hello, Docker…"   53 minutes ago   Exited (0) 53 minutes ago                                   friendly_snyder
```

````

<v-click>

The old image and containers still exist! We must clean them up.

````md magic-move
```sh
# For containers, can use:
docker rm container_name_or_id
# OR
docker run --rm container_name # --rm flag removes the container after it finishes running (for dev)
# Don't delete images and containers if you're planning to deploy them!
```

```sh
# For images
docker rmi image_name
# OR
docker image prune # removes all dangling images
```

```sh
# For both
docker system prune -a --volumes # Be careful! This removes all images and containers that are stopped!
```
````

</v-click>

---

# Practical

Let's set an environment variable via the `ENV` command.

````md magic-move

```docker
# Use an official base image from the Docker Hub
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl
# We can add as many RUN commands as we need
RUN apt-get install -y python3 python3-pip

# Optional: Command to run when the container starts
CMD ["python3", "-c", "print('Hello World')"]
```

```docker {*|8}
# Use an official base image from the Docker Hub
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl
# We can add as many RUN commands as we need
RUN apt-get install -y python3 python3-pip

ENV APP_HOME /app

# Optional: Command to run when the container starts
CMD ["python3", "-c", "print('Hello World')"]
```
````

---

# Practical

Let's copy some files from host to container via the `COPY` command and set a working directory via `WORKDIR`.

```sh
echo "print('Hello World')" > hw.py
```

<br>

<v-click>

````md magic-move

```docker
# Use an official base image from the Docker Hub
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl
# We can add as many RUN commands as we need
RUN apt-get install -y python3 python3-pip

ENV APP_HOME /app

# Optional: Command to run when the container starts
CMD ["python3", "-c", "print('Hello World')"]
```

```docker {*|10-15}
# Use an official base image from the Docker Hub
FROM ubuntu:latest

RUN apt-get update && apt-get install -y curl
# We can add as many RUN commands as we need
RUN apt-get install -y python3 python3-pip

ENV APP_HOME /app

COPY ./hw.py /app/hw.py

WORKDIR /app

# Optional: Command to run when the container starts
CMD ["python3", "hw.py"]
```

```sh
docker build -t my-first-image .
docker run --rm my-first-image
docker image prune
```

````

</v-click>

<v-click>

More Dockerfile commands at: https://docs.docker.com/reference/dockerfile/
</v-click>

---

# Practical

Let's integrate several containers together using `docker-compose.yml`.
<br>
We'll use this to build an container API and make it talk to a database container (MongoDB) in part 2 of this pres.

````md magic-move
```sh
cd ..
mkdir docker_tut_2
cd docker_tut_2
```

```sh
touch docker-compose.yml
```

```yml
#docker-compose.yml
version: '3.9'

services:
  app:
    image: python:3.10
    volumes:
      - ./app:/usr/src/app
    working_dir: /usr/src/app
    command: python app.py
    environment:
      - MONGO_URI=mongodb://mongo:27017/
    depends_on:
      - mongo
    ports:
      - "5000:5000"
#...
```

```yml
#...
  mongo:
    image: mongo:latest
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

```sh
mkdir app
sh
echo "from flask import Flask, request, jsonify\nimport pymongo\napp = Flask(__name__)\nclient = pymongo.MongoClient('mongodb://mongo:27017/')\ndb = client.testdb\n@app.route('/')\ndef hello():\n    return 'Hello from Flask and MongoDB!'\nif __name__ == '__main__':\n    app.run(host='0.0.0.0', port=5000)" > app/app.py
```

```sh
docker-compose up
```

````

---

# Practical

How can you debug issues in your container?
<br>

```sh
# Enter the container
docker exec -it mycontainer bash
```

<br>
```sh
# See logs
docker logs --tail 20 mycontainer
```