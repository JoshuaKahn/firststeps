# Readme
This document outlines the process of setting up a Jenkins Server via Docker with a BeagleBone Black slave.

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
We then verify that Docker's fingerprint is correct, by checking [online](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#set-up-the-repository):
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
Now, we need to install Jenkins via Docker. We first pull the image:
```
docker pull jenkins/jenkins:lts
```
Then, navigate to the folder containing the .dockerignore and 'Dockerfile' files, then run the following command using the given directory:

`docker build -f /path/to/file/Dockerfile .`

We now want to create a new container, using the image we just created. To find the name of the image, type `docker images` and find the most recently created image.

The following command creates the new container, but also sets the port for entry as the local host rather than an ip address broadcasted across the internet.
```
docker run \
-u root \
--name jenkins \
-p 127.0.0.1:9090:8080 \
-v jenkins-data:/var/jenkins_home \
-v /var/run/docker.sock:/var/run/docker.sock \
NEW_IMAGE_NAME_HERE
```
Copy the pin generated in the terminal log and exit (via Ctrl + C). 

Log into localhost:9090.html using the pin code generated from the log output, and set credentials (this can be disabled later in Manage Jenkins->Security). If Jenkins has not started (and thus the server page is unavaliable), then enter `docker start jenkins`.

The `docker start jenkins` command is how to start the server again if the machine was restarted.

## BeagleBone setup
Connect the BeagleBone Black to the server via USB. Then, [download the following script to the server and run it as sudo.](beagleboard.org/static/Drivers/Linux/FTDI/mkudevrule.sh)

Make sure you can connect to your BeagleBone by navigating to http://192.168.7.2/ in your web browser, then type the following to connect to the BeagleBone:
`ssh 192.168.7.2 -l debian`
If connecting to Debian, the password is *temppwd*. 

Connect the Beaglebone Black to the internet via an Ethernet port, then install Java by typing:
```
sudo apt-get update
sudo apt-get install default-jdk
```

This is so Jenkins' slave software can use the BeagleBone as a node for testing/flashing. It is now fine to disconnect the BeagleBone Black from the internet.

Finally, we must implement the ST-Link flashing software onto the beaglebone. Download the source from [here](https://github.com/texane/stlink) and transfer it over to the beaglebone via ssh transfer via SCP.

```sudo scp source_file_here.tar  debian@192.168.7.2:/home/debian```

Extract the tar file in its current directory and [follow the website's instructions on compiling from source.](https://github.com/texane/stlink/blob/master/doc/compiling.md)

## Jenkins Configuration Setup
### Easy Method
The easiest method of doing this is to use the already-configured XML files. To do this, go onto the server and execute the following command:
```wget http://localhost:9090/jnlpJars/jenkins-cli.jar```
This is to download the Command Line Interface for the server, which does not require anything running within the Docker container itself. Within the terminal, go to the folder where this was downloaded, import the given configuration files, and run the following:

```
java -jar jenkins-cli.jar -s http://localhost:9090 create-credentials-by-xml system::system::jenkins "(global)"< beagle.xml
java -jar jenkins-cli.jar -s http://localhost:9090 create-credentials-by-xml system::system::jenkins "(global)"< git.xml

java -jar jenkins-cli.jar -s http://localhost:9090 create-job STM < stmtest.xml
java -jar jenkins-cli.jar -s http://localhost:9090 create-node ARMslave < test_slave.xml
```
This will automatically install the beaglebone slave node's and the git's credentials, creates the server required for testing, and implements the BeagleBone as a slave. You will still need to enter the passwords for the credentials, however; to do this, head to Credentials->Global (Jenkins) then click on the given credential and enter it. 
### Hard Method
If the above does not work, you will have to implement the server manually. Log onto the server via the html address and follow the instructions below.
#### Node
Create a new node (Manage Jenkins→ Manage Nodes → New Node) with the following settings:
1. Permanent Agent
2. Name = Test_slave (or anything else to distinguish it)
3. Directory may change depending on name of BeagleBone and/or the account used.
5. Launch Slave Agents via SSH, and 'Manually trusted Key Verification Strategy'
6. For SSH credentials, use the credentials given for the debian account (or the account used.) This you will have to enter.
7. Use this node as much as possible and keep online as much as possible.

##Complete
You are done! Test that everything is complete by running a test build.
#### Job
Create a new job with the following settings:
1. Multibranch Pipeline
2. Select 'git' as a branch source. Enter the git's address and add an 'SSH username with private key' credential to access it (with a given private key.)
3. Build configuration by Jenkinsfile
4. Turn OFF Discard Old Items

## End
You are done!
