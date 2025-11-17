# CYB3353 - Lab 2
Docker Installation

## Introduction
I initially attempted to do what I normally do for Linux-adjacent projects in my classes, that is, run it natively within my macOS system. However, the challenges associated with this became apparent very quickly. There are various ways to run Docker containers on macOS, many of which are unfortunatley paid or nonfree options. Since the goal of this projects was primarily to install the Docker Engine, I needed to install Docker itself. Even so, there were some ways proposed online, all of which involved running a lightweight Linux hypervisor. As seems to be the case with many things on macOS, though, the installation instructions for the `colima` hypervisor simply did not work and no solutions online were able to help. Thus, out of options, I installed Asahi Linux, an experimental remix of Fedora built for Apple Silicon, and began the standard process of installing Docker on Fedora.

## Installation
Fedora uses `dnf` to install packages, so that's what I'll be using here. Since this is a fresh install, we _shouldn't_ have any existing Docker packages to remove, but for completeness sake, we can run
```bash
$ sudo dnf remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-selinux docker-engine-selinux docker-engine
```
After ensuring no leftover packages remain, we can then set up the Docker repository by first installing the `dnf-plugins-core` package allow us to manage respositories:
```bash
$ sudo dnf install dnf-plugins-core
```
With that installed, we can then add the Docker repository:
```bash
$ sudo dnf-3 config-manager --add-repo https://download.docker.com/linux/fedora/docker-ce.repo
```
From that repository, we can then install all of the reccomended packages:
```bash
$ sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Finally, we can enable the Docker Engine to continually run:
```bash
$ sudo systemctl enable --now docker
```

## Testing Docker
Now that the Docker Engine is running, we can run the `hello-world` image to ensure it is properly working.
```bash
$ sudo docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
198f93fd5094: Pull complete 
Digest: sha256:f7931603f70e13dbd844253370742c4fc4202d290c80442b2e68706d8f33ce26
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.
```
Correctly, Docker downloads the `hello-world` image and runs it.

## Installing Ollama
Ollama is a project that allows you to run a LLM locally, either within the CLI or with a web-based interface. It provides a `Dockerimage` and I was able to find a `docker-compose.yml` file on GitHub [here](https://github.com/mythrantic/ollama-docker). First, I needed to install `git-core` on my system, and then I could clone the repository. I then ran the following command to set up Ollama and the dependencies:
```bash
$ sudo docker compose up -d
```
Once Ollama was running after a few minutes, I could then use `docker exec` to run the `qwen2:0.5b` model (a lightweight model since Ollama does not support Apple GPU acceleration under Linux) and tell it hello:
```bash
$ sudo docker exec ollama ollama run qwen2:0.5b Hello
Hello! How can I assist you today? Is there anything specific that you would like to know or discuss? I'm here to help.
```
