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
For installation, requirements are `git`, `slurm`/`munge`, `sudo` and `sshd`.

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
$git clone -b release_18.05 https://github.com/galaxyproject/galaxy.git
```

--------------------------------------------------------------
## Some more preparations

Now, before we continue, let's go back to our sudo user and finish some more preparation. Basically, we want to configure the job scheduler and database for history and tasks tracking. We will configure slurm job schedule and PostgreSQL database for our local instance of Galaxy server.

### Installing slurm Scheduler

