# Installation Guide for a Scalable Galaxy Server on a Workstation with Ubuntu 18.04
## About
This documentation contains steps and scripts to be used for the installation of a galaxy webserver instance using the following specifications:

- Ubuntu 18.04.1 LTS (GNU/Linux 4.15.0-33-generic x86_64)
- 2 CPUs, 30 Cores, 384 GB RAM, 4TB storage
- Linux user to run servers (galaxy, ftp, http), and submit jobs
- galaxy home
- Module environment for software path management
  - slurm
  - python 2.7.15rc1
- PostgreSQL server with user and database for galaxy

For installation, requirements are `git`, `slurm`/`munge`, `sudo` and `sshd`. In the documentation, if you see the "#" sign besides any of the commands, a brief comment is to follow. So, you may ignore it if you understand the command.

## Preparation
Create a "galaxy" user that does not belong to the group of sudoers and will hold ownership
over galaxy server files.

```
$ sudo adduser galaxy
$ sudo su galaxy # to become the galaxy user
```

Run the installation commands from home folder of "galaxy" user.

```
$ cd
```

## Installation

Clone the repository from github. Please visit [here](https://galaxyproject.org/admin/get-galaxy/), for an updated version.

```
$ git clone -b release_18.05 https://github.com/galaxyproject/galaxy.git
```

--------------------------------------------------------------
## Some more preparations

Now, before we continue, let's go back to our sudo user and finish some more preparation. Basically, we want to configure the job scheduler and database for history and tasks tracking. We will configure slurm job schedule and PostgreSQL database for our local instance of Galaxy server.

```
$ exit # to exit from the galaxy user to a sudo user
```

### Installing slurm Scheduler
An excellent and detailed documentation for this step can be found [here](http://galaxyproject.github.io/training-material/topics/admin/tutorials/connect-to-compute-cluster/tutorial.html). Therefore, the steps needed will be listed here but without a full explanation. Galaxy server comes with a limited job scheduler that struggles to scale with workflow implementations. Thus, we will install a very scalable scheduler and configure Galaxy server to work with it.

#### Section 1 - Install and configure Slurm
```
$ sudo apt-get install -y slurm-wlm
```

Create the user slurm on the system:
```
$ sudo deluser slurm # please note that this step might not be necessary. However, when experienced some debugging issue, I had to make sure that a slurm home directory exist. If one does not exist, just delete the slurm user and create it again.
$ sudo adduser slurm
$ sudo chown slurm:slurm /var/lib/slurm-llnl/slurmctld
$ sudo chown slurm:slurm /var/log/slurm-llnl
```
One probably, can add the user "slurm" and then, install slurm-wlm which will help in avoiding to run chown commands in this case.

Under Ubuntu, Slurm configs are stored in /etc/slurm-llnl. No config is created by default.
Before starting slurm scheduler, we will need to create a slurm.conf configuration file. You can use this [site](https://slurm.schedmd.com/configurator.easy.html) to create the file or by opening /usr/share/doc/slurmctld/slurm-wlm-configurator.html. Here, we will use [this file](slurm.conf) and upload it to /etc/slurm-llnl on the workstation. Several variables are necessary to be defined to have slurm scheduler working. If you are going to user our uploaded file, PLEASE replace $(localhost) everywhere in the file by its actual system value. Also, replace $(nproc). Otherwise, slurm may fail to start properly.

For example, the following options including others helped in successfully enabling slurm to run several jobs in one workstation.

- SelectType=select/cons_res
- SelectTypeParameters=CR_Core

Remember, slurm is originally meant for a cluster computing environment but here, we are setting it up for a single workstation.

So, after copying the slurm.conf file or creating it in /etc/slurm-llnl, we can run the following commands to bring slurm up:
```
$ sudo /etc/init.d/slurmctld start
$ sudo /etc/init.d/slurmd start
$ sinfo # If SLURM runs successfully, sinfo should give you a non-error, and display the column headers for SLURM jobs
```

#### Section 2 - Get Slurm ready for Galaxy
```
$ sudo apt-get install slurm-drmaa1 # For Ubuntu 18.04, if the repository is not yet added, you can sudo add-apt-repository "deb http://ca.archive.ubuntu.com/ubuntu/ xenial universe", sudo add-apt-repository "deb http://ca.archive.ubuntu.com/ubuntu/ xenial-updates main restricted", etc, of Ubuntu 16.04. Then, install slurm-drmaa1
```

##### Configure galaxy.conf
...


### Installing PostgreSQL database

## Installing Docker and building PostgreSQL image
The steps for installing Docker on Ubuntu is fully documented [here](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-18-04).
```
$ sudo apt-get update
$ sudo apt install apt-transport-https ca-certificates curl software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - # Then add the GPG key for the official Docker repository to your system
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
$ sudo apt update
$ sudo apt install docker-ce
```
After installing Docker, you can use [this Dockerfile](Dockerfile) to build a fully configured image with PostgreSQL. The image is setup to work with the Galaxy server.
```
$ sudo docker build -t postgresondocker:9.3 . # build image
$ sudo docker volume create datavol_inst1 # for Data persistence
$ sudo docker run --name psql_inst1 -v datavol_inst1:/var/lib/postgresql/9.3/main -p 5632:5432 -d postgresondocker:9.3 # 5632:5432 refers to port forwarding from host machine port 5632 to container port 5432. 5432 is the default PostgreSQL port but we use 5632 on any other value on the host just in case PostgreSQL is running over the hosting server.
```
Before, we link to the container, we just need to install a PostgreSQL client on the host machine:
```
$ sudo apt install postgresql-client-common
$ sudo apt install postgresql-client-*
```
Now to run the PostgreSQL configured on the container image from the host machine, you can use the following command:
```
$ psql -h localhost -p 5632 -d galaxy -U galaxy --password # 5632 is the forwarding host machine port initiated by the previous command
```
