## 

- update for used with yum 
- settings from here: https://www.pitt-pladdy.com/blog/_20150720-132951_0100_Home_Lab_Project_apt-cacher-ng_with_CentOS/

- add curl to Dockerfile
- add miiror list to entrypoint: curl https://www.centos.org/download/full-mirrorlist.csv | sed 's/^.*"http:/http:/' | sed 's/".*$//' | grep ^http >/etc/apt-cacher-ng/centos_mirror


- add centos based config to acng.conf

```bash
VfilePatternEx: ^/\?release=[0-9]+&arch=
VfilePatternEx: ^(/\?release=[0-9]+&arch=.*|.*/RPM-GPG-KEY-examplevendor)$
Remap-centos: file:centos_mirrors /centos
PassThroughPattern: (mirrors\.fedoraproject\.org|some\.other\.repo|yet\.another\.repo):443
```

- add config file to compose-docker.yml

```yaml
- ./acng.conf:/etc/apt-cacher-ng/acng.conf
```

- change image name

```yaml
image: mathiasstadler/apt-cacher-ng-yum:latest
```

## build

```bash
docker build -t mathiasstadler/apt-cacher-ng-yum .
```

## run 

```bash
docker-compose -f docker-compose.yml up -d
```

## log

docker exec -it <container_id>  tail -f /var/log/apt-cacher-ng/apt-cacher.log


## setting on host

- /etc/yum.conf

```bash
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=True
proxy=http://192.168.178.29:3142
tolerant=1
errorlevel=1
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=0
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
gpgcheck=1
plugins=1
installonly_limit=5
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
distroverpkg=centos-release
```

- enable

```bash
yum clean expire-cache
yum update
```

HAVE FUN

## housekeeping

```bash
# found space
cd && cd playground
# clone 
git clone https://github.com/MathiasStadler/vagrant-openstack-k8s-kvm-nested.git
# start VSCODE inside browser 
CID=$(docker run -it -d -p 0:8080 -v "${PWD}:/home/coder/project" -u "$(id -u):$(id -g)" codercom/code-server:3.2.0  --cert)
#found port of container
docker inspect -f '{{ (index (index .NetworkSettings.Ports "8080/tcp") 0).HostPort }}' ${CID}
# found password
docker logs ${CID} |grep Password
# open with browser of your choice

# add to the terminal with your data
git config --global user.name "Mathias Stadler"
git config --global user.EMAIL "email@mathias-stadler.de"
# https://help.github.com/en/github/using-git/caching-your-github-password-in-git
git config --global credential.helper 'cache --timeout=3600'

# add extension
sudo apt-get update && sudo apt-get install -y rubygems build-essential
sudo gem install rufo
code-server --install-extension siliconsenthil.rufo-vscode
code-server --install-extension streetsidesoftware.code-spell-checker
code-server --install-extension davidanson.vscode-markdownlint
code-server --install-extension eamodio.gitlens
code-server --install-extension gruntfuggly.todo-tree
# create folder for todo-tree extension
mkdir -p /home/coder/.local/share/code-server/User/globalStorage/gruntfuggly.todo-tree
code-server --install-extension redhat.vscode-yaml
sudo apt-get install -y shellcheck
code-server --install-extension timonwong.shellcheck
code-server --install-extension foxundermoon.shell-format
code-server --install-extension  ms-azuretools.vscode-docker
code-server --install-extension rebornix.ruby
code-server --list-extensions
```


--  README.md from sameersbn/apt-cacher-ng ------------------------------------


[![Circle CI](https://circleci.com/gh/sameersbn/docker-apt-cacher-ng.svg?style=shield)](https://circleci.com/gh/sameersbn/docker-apt-cacher-ng) [![Docker Repository on Quay.io](https://quay.io/repository/sameersbn/apt-cacher-ng/status "Docker Repository on Quay.io")](https://quay.io/repository/sameersbn/apt-cacher-ng)

# sameersbn/apt-cacher-ng:3.1-3

- [Introduction](#introduction)
  - [Contributing](#contributing)
  - [Issues](#issues)
- [Getting started](#getting-started)
  - [Installation](#installation)
  - [Quickstart](#quickstart)
  - [Command-line arguments](#command-line-arguments)
  - [Persistence](#persistence)
  - [Docker Compose](#docker-compose)
  - [Usage](#usage)
  - [Logs](#logs)
- [Maintenance](#maintenance)
  - [Cache expiry](#cache-expiry)
  - [Upgrading](#upgrading)
  - [Shell Access](#shell-access)

# Introduction

`Dockerfile` to create a [Docker](https://www.docker.com/) container image for [Apt-Cacher NG](https://www.unix-ag.uni-kl.de/~bloch/acng/).

Apt-Cacher NG is a caching proxy, specialized for package files from Linux distributors, primarily for [Debian](http://www.debian.org/) (and [Debian based](https://en.wikipedia.org/wiki/List_of_Linux_distributions#Debian-based)) distributions but not limited to those.

## Contributing

If you find this image useful here's how you can help:

- Send a pull request with your awesome features and bug fixes
- Help users resolve their [issues](../../issues?q=is%3Aopen+is%3Aissue).
- Support the development of this image with a [donation](http://www.damagehead.com/donate/)

## Issues

Before reporting your issue please try updating Docker to the latest version and check if it resolves the issue. Refer to the Docker [installation guide](https://docs.docker.com/installation) for instructions.

SELinux users should try disabling SELinux using the command `setenforce 0` to see if it resolves the issue.

If the above recommendations do not help then [report your issue](../../issues/new) along with the following information:

- Output of the `docker version` and `docker info` commands
- The `docker run` command or `docker-compose.yml` used to start the image. Mask out the sensitive bits.
- Please state if you are using [Boot2Docker](http://www.boot2docker.io), [VirtualBox](https://www.virtualbox.org), etc.

# Getting started

## Installation

Automated builds of the image are available on [Dockerhub](https://hub.docker.com/r/sameersbn/apt-cacher-ng) and is the recommended method of installation.

> **Note**: Builds are also available on [Quay.io](https://quay.io/repository/sameersbn/apt-cacher-ng)

```bash
docker pull sameersbn/apt-cacher-ng:3.1-3
```

Alternatively you can build the image yourself.

```bash
docker build -t sameersbn/apt-cacher-ng github.com/sameersbn/docker-apt-cacher-ng
```

## Quickstart

Start Apt-Cacher NG using:

```bash
docker run --name apt-cacher-ng --init -d --restart=always \
  --publish 3142:3142 \
  --volume /srv/docker/apt-cacher-ng:/var/cache/apt-cacher-ng \
  sameersbn/apt-cacher-ng:3.1-3
```

*Alternatively, you can use the sample [docker-compose.yml](docker-compose.yml) file to start the container using [Docker Compose](https://docs.docker.com/compose/)*

## Command-line arguments

You can customize the launch command of Apt-Cacher NG server by specifying arguments to `apt-cacher-ng` on the `docker run` command. For example the following command prints the help menu of `apt-cacher-ng` command:

```bash
docker run --name apt-cacher-ng --init -it --rm \
  --publish 3142:3142 \
  --volume /srv/docker/apt-cacher-ng:/var/cache/apt-cacher-ng \
  sameersbn/apt-cacher-ng:3.1-3 -h
```

## Persistence

For the cache to preserve its state across container shutdown and startup you should mount a volume at `/var/cache/apt-cacher-ng`.

> *The [Quickstart](#quickstart) command already mounts a volume for persistence.*

SELinux users should update the security context of the host mountpoint so that it plays nicely with Docker:

```bash
mkdir -p /srv/docker/apt-cacher-ng
chcon -Rt svirt_sandbox_file_t /srv/docker/apt-cacher-ng
```

## Docker Compose

To run Apt-Cacher NG with Docker Compose, create the following `docker-compose.yml` file

```yaml
---
version: '3'

services:
  apt-cacher-ng:
    image: sameersbn/apt-cacher-ng
    container_name: apt-cacher-ng
    ports:
      - "3142:3142"
    volumes:
      - apt-cacher-ng:/var/cache/apt-cacher-ng
    restart: always

volumes:
  apt-cacher-ng:
```

The Apt-Cache NG service can then be started in the background with:

```bash
docker-compose up -d
```

## Usage

To start using Apt-Cacher NG on your Debian (and Debian based) host, create the configuration file `/etc/apt/apt.conf.d/01proxy` with the following content:

```config
Acquire::HTTP::Proxy "http://172.17.0.1:3142";
Acquire::HTTPS::Proxy "false";
```

Similarly, to use Apt-Cacher NG in you Docker containers add the following line to your `Dockerfile` before any `apt-get` commands.

```dockerfile
RUN echo 'Acquire::HTTP::Proxy "http://172.17.0.1:3142";' >> /etc/apt/apt.conf.d/01proxy \
 && echo 'Acquire::HTTPS::Proxy "false";' >> /etc/apt/apt.conf.d/01proxy
```

## Logs

To access the Apt-Cacher NG logs, located at `/var/log/apt-cacher-ng`, you can use `docker exec`. For example, if you want to tail the logs:

```bash
docker exec -it apt-cacher-ng tail -f /var/log/apt-cacher-ng/apt-cacher.log
```

# Maintenance

## Cache expiry

Using the [Command-line arguments](#command-line-arguments) feature, you can specify the `-e` argument to initiate Apt-Cacher NG's cache expiry maintenance task.

```bash
docker run --name apt-cacher-ng --init -it --rm \
  --publish 3142:3142 \
  --volume /srv/docker/apt-cacher-ng:/var/cache/apt-cacher-ng \
  sameersbn/apt-cacher-ng:3.1-3 -e
```

The same can also be achieved on a running instance by visiting the url http://localhost:3142/acng-report.html in the web browser and selecting the **Start Scan and/or Expiration** option.

## Upgrading

To upgrade to newer releases:

  1. Download the updated Docker image:

  ```bash
  docker pull sameersbn/apt-cacher-ng:3.1-3
  ```

  2. Stop the currently running image:

  ```bash
  docker stop apt-cacher-ng
  ```

  3. Remove the stopped container

  ```bash
  docker rm -v apt-cacher-ng
  ```

  4. Start the updated image

  ```bash
  docker run --name apt-cacher-ng --init -d \
    [OPTIONS] \
    sameersbn/apt-cacher-ng:3.1-3
  ```

## Shell Access

For debugging and maintenance purposes you may want access the containers shell. If you are using Docker version `1.3.0` or higher you can access a running containers shell by starting `bash` using `docker exec`:

```bash
docker exec -it apt-cacher-ng bash
```
