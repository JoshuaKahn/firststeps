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
Copy the pin generated in the terminal log and exit (via Ctrl + C). 
Then, navigate to the folder containing the .dockerignore and 'Dockerfile' files, then run the following command using the given directory:

`docker build -f /path/to/file/Dockerfile .`

Log into localhost:9090.html using the pin code generated from the log output, and set credentials (this can be disabled later in Manage Jenkins->Security). If Jenkins has not started (and thus the server page is unavaliable), then enter `docker ps -a` into the terminal and find the most recently created container, then `docker start *given name/Container ID*`.

It is recommended that you complete the last two commands regardless, as it will be these containers that you use to manage the server.

## BeagleBone setup
Connect the BeagleBone Black to the server via USB. Then, [download the following script to the server and run it as sudo.](beagleboard.org/static/Drivers/Linux/FTDI/mkudevrule.sh)

Make sure you can connect to your BeagleBone by navigating to http://192.168.7.2/ in your web browser, then type the following to connect to the BeagleBone:
`ssh 192.168.7.2 -l debian`
If connecting to Debian, the password is *temppwd*. 

Connect the Beaglebone Black to the internet via an Ethernet port, then install Java by typing:
`sudo apt-get update`
`sudo apt-get install default-jdk`

This is so Jenkins' slave software can use the BeagleBone as a node for testing/flashing. It is now fine to disconnect the BeagleBone Black from the internet.

## Jenkins Configuration Setup
### Easy Method
The easiest method of doing this is to use the already-configured XML files. To do this, go onto the server and execute the following command:
`wget http://localhost:9090/jnlpJars/jenkins-cli.jar`
This is to download the Command Line Interface for the server, which does not require anything running within the Docker container itself. Within the terminal, go to the folder where this was downloaded, import the given configuration files, and run the following commands:

```
java -jar jenkins-cli.jar -s http://localhost:9090 create-credentials-by-xml system::system::jenkins "(global)"< beagle.xml
java -jar jenkins-cli.jar -s http://localhost:9090 create-credentials-by-xml system::system::jenkins "(global)"< git.xml

java -jar jenkins-cli.jar -s http://localhost:9090 create-job STM < stmtest.xml
java -jar jenkins-cli.jar -s http://localhost:9090 create-node ARMslave < test_slave.xml
```
This will automatically install the beaglebone slave node's and the git's credentials, creates the server required for testing, and implements the BeagleBone as a slave.
