# Readme
This document outlines the process of setting up a Jenkins Server.

## What's required?
* Docker
* Jenkins (via Docker)
* BeagleBone Black

## Docker Installation
We first allow apt-get to install over https, and add Docker's official GPG key:
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
Log off and log on, then run `docker run hello-world` to test the group addition.

## Jenkins Initial Setup w/ Dockerfile
Now, we need to install Jenkins via Docker. The following command does this, but also sets the port for entry as the local host rather than an ip address broadcasted across the internet.
```
docker run \
-u root \
--name jenkins \
-p 127.0.0.1:9090:8080 \
-v jenkins-data:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
jenkins/jenkins:lts
```
Then, navigate to the folder containing the .dockerignore and 'Dockerfile' files, then run the following command using the given directory:

`docker build -f /path/to/file/Dockerfile .`

Log into localhost:9090.html using the pin code generated from the log output (which you exit from by using Ctrl+C), and set credentials (this can be disabled later in Manage Jenkins->Security). If Jenkins has not started (and thus the server page is unavaliable), then enter `docker start jenkins` into the terminal.

