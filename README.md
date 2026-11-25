# Docker Install

This document contains all of the steps I took to install Docker and create a basic container. I did this on a VM running Fedora server, and the app I containerized was BookStack. This app actually requires two containers, one for BookStack and one for MariaDB.

## Part 1: Install Docker Engine

For this part, I mostly followed the Docker Docs installation tutorial for Fedora, as that was the distro my VM was running.

https://docs.docker.com/engine/install/fedora/#install-using-the-repository

### Step 1: Check OS Version

Docker requires fedora 45, 44 or 43. To check my OS version I ran `cat /etc/os-release` which showed that I was on fedora 43.

### Step 2: Uninstall old versions

I ran this command according to the tutorial just in case but it reported there was nothing to do.

```
sudo dnf remove docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine\
```

### Step 3: Setup RPM Repository

There are other ways to install docker, but the tutorial recommended this way for ease. 

First I ran the command `sudo dnf config-manager addrepo --from-repofile https://download.docker.com/linux/fedora/docker-ce.repo`. This adds the official docker repo to your list of repos, allowing you to get the correct version of docker and it's dependencies.

### Step 4: Install Docker

Using the repo I installed the Docker with `sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

During this download, I was prompted to check the PGP fingerprint "060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35". This was the same as the one on the tutorial so I went ahead with the download.

### Step 5: Start Docker Engine and confirm

To actually start the Engine I ran `sudo systemctl enable --now docker`. This created a symbolic link. To make sure it was running I ran `sudo docker run hello-world`. This printed a message saying my Docker install seemed to be working!

I rebooted and tried again to see if it was running on startup and that also worked.

I also ran the following commands to check docker status.

`systemctl is-enabled docker` returns `enabled`
`docker compose version` returns `Docker compose version [most recent docker release]`

Finally I ran `sudo usermod -aG docker $USER` to add myself to the docker group. This is not necessary but makes using docker more convinient. If you skip this, following commands must be done with sudo.

## Part 2: Create Container

The application I chose to containerize was BookStack. To do this I used the BookStack documentation and a youtube tutorial

https://www.bookstackapp.com/ 

https://www.youtube.com/watch?v=w-cC9cMLIyg

### Step 1: Create BookStack directory

It is most convinient to create the files related to the container in the same directory. For my example, I made a directory called BookStack.

### Step 2: create yaml file

Within the directory, you must make a file called `docker-compose.yaml`. Docker will look for this file whenever you start a container within the directory. I started by copying the complete example yaml from https://codeberg.org/bookstack/devops/src/branch/main/config/lsio-docker/docker-compose.yml#. 

The comments on this file are very helpful for setting up your compose file, but the specific steps I took are as follows.
- updated bookstack and mariadb images to latest available versions
- changed APP_URL from localhost to my VM ip and app port number
- generate new app key with `docker run -it --rm --entrypoint /bin/bash lscr.io/linuxserver/bookstack:latest appkey`
- change default passwords, assure that mariadb variables are the same as their bookstack equivalents
- change port under ports to be the same as previously selected app port

You may also want to change the name, location or type of the volumes used. This can be done by changing the options under the volumes sections of both apps. This compose file creates a bind mounted volume for each app in the same directory as the compose file. 

### Step 3: Start container

This is done by simply running `docker compose up -d`. Docker then pulls the images and creates the containers and network. The app can then be accessed by typing the server IP and port into a browser.

### Step 4: login as admin

The default email is "admin@admin.com" and the default password is "password"

After logging in for the first time, you can change the password and email by clicking on the profile in the top right, selecting "my account" and chaning the respective settings in "profile details" and "access and security"

Now, your BookStack is ready to use.
