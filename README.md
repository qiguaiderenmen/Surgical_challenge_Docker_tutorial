## Before we start

1. Install [Docker Engine](https://docs.docker.com/engine/install/ubuntu/) on your machine. Please make you have the supported Ubuntu version in your PC, either being a desktop or on a virtual machine.

You can verify the installation by running:
```bash
docker run hello-world
```

If the system somehow hints that you need to use 'sudo' to run Docker commands, please kindly check [post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/) here.

Now we're ready to create our own image/container using Docker.

2. Sign up to [Docker hub](https://hub.docker.com/) to get a remote access to manage your containers. You'll then be able to push/pull images from/to Docker, which will also allow us to check your remarkable efforts in the end of this challenge. Please take care that for free account there're pull/push limits.


## Setup environment

The main proposition of using container here is to communicate with AMBF installed on your local machine, control it via some APIs and perform automation by your sparkling ideas. Generally speaking, it should involve ROS Melodic and some Python scripts, for which you'll have two options to set them up.

- **Via a few commands**

1. Open a terminal, pull the ROS image and create a local container:

```bash
docker run -it --name <container name> --network host ros:melodic /bin/bash
```

This command automatically pulls the official ROS Melodic image from Docker hub, creates a local container with the name you specify, sets the container's network being exactly the same as your local machine, and opens a bash terminal inside container for interaction.

2. Clone controller API in container:

```bash
# cd to your workspace. Here I use home
cd ~/home
git clone https://github.com/adnanmunawar/surgical_robotics_challenge 
```

- **Via Docker build**

## Tests

Now we can do some simple tests to verify whether we have the environment ready.

1. Check Docker status:
```bash
# open a new local terminal
# check if you have the correct image, ros:melodic installed. 'ros' is under 'RESPOSITORY', 'melodic' is under 'TAG'
docker images
# check if you have the cotainer created
docker ps -a
```

2. Publish a test node locally
```bash
# run ros master on local machine
roscore
# open another local terminal and publish the node
rostopic pub /test std_msgs/Bool "data: false" -r10
```
3. Subscribe in container ROS
```bash
# in container's bash
# list all the topics. You should see a node called '/test'
rostopic list
# subscribe the node, and you'll see 'data: False' occurs repeatedly
rostopic echo /test
```

4. In case you want to open another bash in container:
```bash
# in local terminal
docker exec -it <container name> bash
```
This command populates another bash terminal in the specified container for interaction.

Congrats! Now you're all set for the initial setup, basically a ROS bridge between local machine and the container. You can play with the API in container to control the AMBF in local, and then elaborate your own development for surgical automation.

## Push & submit

Once you're done with the development, please refer to the following commands to submit your awesome jobs to Docker hub, so that everyone can review and we'll be able to test it locally.

1. Use 'docker commit' to make a new image
```bash
docker commit <container ID> <new image name>
```
Container ID can be seen by using 'docker ps -a' command. **Example**:
```bash
docker commit a554ba6ed056 ros-ambf:melodic
```
'a554ba6ed056' is the container ID, while 'ros-ambf' is the image respository, 'melodic' is the image tag. Command 'docker images' can help you check them.

2. Save a local image
```bash
docker save -o <output image name.tar> <image repo>:<image tag>
```
**Example**:
```bash
docker save -o ros-ambf.tar ros-ambf:melodic
```
In case you want to use a local image:
```bash
docker load -i <local .tar file>
```

3. Publish your image!
```bash
# login to docker hub
docker login -u <username> -p <password>
# tag the image
docker tag <image repo>:<image tag> <username>/<image repo>:<image tag>
# push the image
docker push <username>/<image repo>:<image tag>
```
**Example**:
```bash
docker tag ros-ambf:melodic strangeman1234/ros-ambf:melodic
docker push strangeman1234/ros-ambf:melodic
```

All set! If you now login to the Docker hub website, you'll see this image repo staying there.

## Some extra information

1. Somehow in a new ROS container bash, when you do 'rostopic list' or other commands, it may occur: 'bash: rostopic: command not found'

That case we just need to source it. Do either:
``` bash
source /opt/ros/melodic/setup.bash 
```
for every new bash, or simply add this line to .bashrc file once for all.

2. If you would prefer a graphical desktop in conatiner, a vnc/novnc (in browser) solution might be what you need. Refer to the following link for further information:

https://github.com/Tiryoh/docker-ros-desktop-vnc

However, it is not recommended since vnc needs a port (such as 6080:80) for local machine to expose, and in this setting, Docker can no longer allow us to designate host network for container. In that case, you'll only be able to run ROS MASTER in local. You'll not be able to see any rostopics vise versa.

3. Please do take a look at Docker's website for more useful commands, such as 'docker rmi', 'docker stop', etc. Also, the Dockerstation APP may help you manage your containers via GUI.

https://docs.docker.com/engine/reference/commandline/docker/
https://dockstation.io/

**Enjoy your surgical challenge tour!** 