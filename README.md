# Readme
This document outlines the process of setting up a Jenkins Server.

## What's required?
* Docker
* Jenkins (via Docker)
* BeagleBone Black

## Docker Installation
We first add the repo key:
```
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
We then verify that Docker's fingerprint is correct, by checking [online](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#set-up-the-repository)
```
sudo apt-key fingerprint 0EBFCD88 	
```
Now we add the repository:
```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update 			#install docker from repo
sudo apt-get install docker-ce
```
Test that Docker works by running `sudo docker run hello-world`.

Finally, we need to add Docker to usergroups so that we don't need to write 'sudo' before every docker command:
```
sudo groupadd docker
sudo usermod -aG docker $USER 	#allows non-sudo users to run docker
```
Log off and log on, then run `docker run hello-world`
