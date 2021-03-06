
## Create your own Deep Learning development environment using Docker

Create up your own DL environement based on docker containers 

## Index 

- 1. Introduction 
	* Why Deep Learning ? 
	* Why Docker ?
   	* Prerequisite

- 2. Install Docker 
	* What is Docker ? 
	* Docker installation
	* Shared volume
	* Run your container
- 3. Install Jupyterhub
	* What is Jupyterhub ? 
	* Jupyterhub installation 
	* Jupyterhub configuration
- 4 : Installing a Keras kernel for Jupyter
	* Virtual environments with Conda
	* installing Keras 
	* Using Keras in Jupyter 
- 5: Bonus : docker tips for improving your container
	* Create your Docker image
	* Share a host directory with your container 
	* run your container as a service 

---------------------------------------

## 1. Introduction 

### Why Deep Learning ? 

Appart from being surrounder by a lot of hype, **Deep Learning** (DL) is a powerfull set of methods that will help you adress many complex problematic. One of the most famous and groundbreaking application of Deep Learning is related to the field of **Computer Vision** and image classification problems. 

Classifying an image is a difficult task for computers, that have to face many complex issues : viewpoint variation, illumination conditions, etc. Deep Learning techniques have proven particularly powerfull for adressing such issues. 

![alt text](http://cs231n.github.io/assets/challenges.jpeg "computer vision challenging issues" )

This tutorial will help you to get started with Deep Learning by creating your own DL environment from scratch. 

### Why Docker ?

One important difficulty that comes with Deep Learning is the environment setup. There is a variety of DL oriented libraries, but they all require very specific dependances which makes them difficult to plug to your already existing development environment. That is where **Docker** will be of great help. 

We will use the docker container technology in order to easily build an isolated Deep Learning platform, and preserve your current environment from any regression. 

In addition, Docker is a particularly trending technology that is worth trying, as it will help you with any kind of application development and deployment. 


### Prerequisite  

All you need to get started is a Linux machine. If you do not have any, I encourage you to launch a cloud instance with **Amazon Web Service** (AWS) . Even though optimal Deep Learning environment are based on GPU architectures, using a free AWS machine is enough to get you started and adress problems of modes complexity. 

This tutorial is designed for being used with **Red Hat Entreprise Linux** or **CentOS** distributions. 


## 2. Install Docker 
### What is Docker ? 

![alt text](https://images.mondedie.fr/s9ZCIrQ4/ErDsRisu.png "Docker Architecture illustration")


### Docker installation
```
sudo yum update -y
#install docker 
sudo yum install -y docker
#start docker service
sudo service docker start
#add ec2 user to the docker group to avoir using sudo 
sudo usermod -a -G docker ec2-user
```

### Other prerequisite : sharing data and ports

When adressing Deep Learning problems you will have to deal with potentially large datasets. In order to avoid duplicating your data in your container, you can use a share volume to easily read host data from any of your running container.  
Thus, create a new directory that will be mounted as a shared volume : 
```
mkdir /home/share_docker
chmod 555 /home/share_docker
```
In addition, you must open a port on your server in order to access your future development platform. Open the port 8000 using : 
```
sudo firewall-cmd --zone=public --add-port=8000/tcp --permanent
sudo firewall-cmd --reload
```

### Run your container

You are now ready to run your container. The `docker run` commands allows to create and run a container : 

```
docker run --name DL_platform -dti -p 8000:8000 -v /home/share_docker:/home/share_docker:z centos 
```

This needs a few comments. It is important that you understand all the argument passed to the previous command. They will help you understand all the potential of docker and adapt it to your own problems.
* `--name`  :sets the name of your container
* `--dti` : the -d  option is used for running the container  in the background. Using -ti allows to use an interactive terminal with your container. 
* `-p`  : used fro  to map a port of the host to a container port. 
* `-v` : used for sharing volumes between host and container. Notice the `:z` options that allows to pass SE linux tags from the host, solving many issues. 
* `centos` : the last argument is the image used for creating a container. Here we run an container from a raw centos image. We will see that you can create your own docker images. 


Your containter is now up and runing. You can have a look at all your containers with the command `docker ps -a`. If for some reason your container is not running anymore, you can start it with the command `docker start DL_platform`.

You can now start a terminal in your container with the command : 

```
docker exec -it DL_platform bash
```

## 3. Install Jupyterhub

Now that our container is up and running start by installing Jupyterub. 

### What is Jupyterhub ? 

The **Jupyter** is a web application that allows you to create and share documents that contain live code, equations, visualizations and explanatory text. Using **Jupyterhub** allows you to run a multi user Jupyter server.
A very usefull feature of Jupyter is the use of **kernels**. They can be considered as isolated virtual environement in which you can execute several language programming. Thus, you can create a kernel for your Python 3 scripts, and another for your R or Julia script. 
![alt text](https://www.dataquest.io/blog/images/jupyter/interface-screenshot.png)
### Anaconda installation 

For installing Jupyter and creating python environment we will use the very popular **Anaconda** python distribution
Install several compenents for later needs : 
```
yum update -y  
yum  install -y wget bzip2 git
```
Download and install Anaconda.

```
#download Anaconda
wget --quiet http://repo.continuum.io/archive/Anaconda3-4.1.1-Linux-x86_64.sh -O /tmp/Anaconda3-4.1.1-Linux-x86_64.sh 
#make the script executable 
chmod +x /tmp/Anaconda3-4.1.1-Linux-x86_64.sh
#installing to /opt/anaconda
/tmp/Anaconda3-4.1.1-Linux-x86_64.sh  -b -p /opt/anaconda
#cleanup
rm -f /tmp/*
```
Add anaconda binaries repository to your path : 
```
echo 'PATH=$PATH:/opt/anaconda/bin' >> ~/.bashrc
```

### Jupyterhub installation 
#### prerequisite installations
;
Install **nodejs/npm** (Node Package Manager)
```
curl --silent --location https://rpm.nodesource.com/setup_4.x | bash -
yum install -y nodejs
```
Then, install the **configurable http proxy** used by Jupyterhub : 
```
npm install -g configurable-http-proxy
```

### Installing Jupyterhub  : 

Install jupyterhub 
```
/opt/anaconda/bin/pip install jupyterhub
```
Then we must upgrade the jupyter notebook. There is a small workarround here in order to avoid a bug in upgrading python setuptools using anaconda pip. Using the `--ignore-installed` argument we ignore the installed package  and force the  reinstalling instead. 
```
opt/anaconda/bin/pip install --upgrade --ignore-installed  setuptools
opt/anaconda/bin/pip install --upgrade notebook
```
## Start Jupyterhub 

You can now start your Jupyterhub server with the following command line : 
```
cd /srv/
/opt/anaconda/bin/jupyterhub --no-ssl
```
By default, the Jupyterhub server is configured on port 8000. You can access it from `http://yourdomainname:8000` and must login with a unix acount existing in the container. You can create new accounts with the `useradd myuser` command and edit the password with `passwd myuser`. 

![alt text](https://nb4799.neu.edu/wordpress/wp-content/uploads/2015/05/787.png)

## 4. Install a Keras Kernel

### Virtual environment with conda : 
```
/opt/anaconda/bin/conda create --name keras_kernel python=3.5.2  -y 
source /opt/anaconda/bin/activate keras_kernel 
/opt/anaconda/bin/conda install  -y theano=0.8.2
/opt/anaconda/bin/conda install -y h5py=2.6.0 
pip install keras==1.1.1 
/opt/anaconda/bin/conda install -y scikit-learn=0.18.1 pandas=0.19.1 ipykernel=4.5.1 matplotlib=1.5.3 pillow=3.4.1
source /opt/anaconda/bin/deactivate 
```
### Install keras

### Using Keras in Jupyterhub : 


### Jupyterhub configuration
- 4 : Installing a Keras kernel for Jupyter
	* Virtual environments with Conda
	* installing Keras 
	* Using Keras in Jupyter 
	
- 5: Bonus : docker tips for improving your container
	* Create your Docker image
	* run your container as a service 



```

FROM centos

MAINTAINER Robin KIPS <robin.kips@gmail.com>

# install Python with Anaconda
RUN yum install -y wget-1.14 bzip2-1.0.6 gcc-c++-4.8.5  && \
    wget --quiet http://repo.continuum.io/archive/Anaconda3-4.1.1-Linux-x86_64.sh -O /tmp/Anaconda3-4.1.1-Linux-x86_64.sh && \
    chmod +x /tmp/Anaconda3-4.1.1-Linux-x86_64.sh && \
    /tmp/Anaconda3-4.1.1-Linux-x86_64.sh  -b -p /opt/anaconda && \
    rm -f /tmp/* 

# install npm, nodejs & js dependencies
RUN curl --silent --location https://rpm.nodesource.com/setup_4.x | bash - && \
    yum install -y nodejs-4.6.0 && \
    npm install -g configurable-http-proxy@1.3.0 && \ 
    opt/anaconda/bin/pip install jupyterhub==0.6.1  && \ 
    opt/anaconda/bin/pip install --upgrade --ignore-installed  setuptools==28.3.0 && \ 
    opt/anaconda/bin/pip install --upgrade notebook==4.2.3
    
#ssl setup, edit details to match your organization
RUN yum install -y openssl && \
	country=XX && \
	state=XX && \
	locality=XX && \
	organization=XX && \
	organizationalunit=XX && \
	email=XX && \
	openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/my_key.pem -out /etc/ssl/my_cert.pem -subj "/C=$country/ST=$state/L=$locality/O=$organization/OU=$organizationalunit/CN=$commonname/emailAddress=$email"

# install keras kernel
RUN /opt/anaconda/bin/conda create --name keras_kernel python=3.5.2  -y && \
    source /opt/anaconda/bin/activate keras_kernel && \
    /opt/anaconda/bin/conda install  -y theano=0.8.2 && \
    /opt/anaconda/bin/conda install -y h5py=2.6.0 && \
    pip install keras==1.1.1 && \
    pip install flask==0.11.1 && \
    /opt/anaconda/bin/conda install -y scikit-learn=0.18.1 pandas=0.19.1 ipykernel=4.5.1 matplotlib=1.5.3 pillow=3.4.1 && \
    source /opt/anaconda/bin/deactivate 

#set alias : 
RUN echo 'alias conda="/opt/anaconda/bin/conda"' >> ~/.bashrc && \
    echo 'alias activate="/opt/anaconda/bin/activate"' >> ~/.bashrc && \
    echo 'alias deactivate="/opt/anaconda/bin/deactivate"' >> ~/.bashrc && \
    echo 'alias python_keras="/opt/anaconda/envs/keras_kernel/bin/python3"' >> ~/.bashrc && \
    echo 'alias jupyterhub="/opt/anaconda/bin/jupyterhub"' >> ~/.bashrc 

#set ENV
ENV KERAS_BACKEND=theano
ENV PATH=/opt/anaconda/bin:$PATH

#spawn jupyter in /home/ dir for access shared directories.
RUN mkdir -p /srv/jupyterhub/ && \
    cd /srv/jupyterhub/ && \ 
    /opt/anaconda/bin/jupyterhub --generate-config && \
    echo "c.Spawner.notebook_dir = '/home/'" >> jupyterhub_config.py

WORKDIR /srv/jupyterhub/
EXPOSE 443

#start jupyterhub 
CMD ["jupyterhub","--port", "443", "--ssl-key", "/etc/ssl/my_key.pem", "--ssl-cert", "/etc/ssl/my_cert.pem", "-f", "jupyterhub_config.py"]
```
