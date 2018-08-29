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

## Preparation
Requirements are `git`, `slurm`/`munge`, `sudo` and `sshd`.

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

Modify the file `credentials` and source it to export variables.

```
source galaxy-admin/credentials
```

Update the server configuration. This prints the configuration values for a check
and creates the folder `galaxy-admin/configured` which contains all updated files
with the `%%GA_*%%%` variables replaced by your credentials.

```
galaxy-admin/configure
```

Adjust the genome reference indices for the aligner `tophat2` and `bowtie` at
`galaxy-admin/configured/server/tool-data/bowtie2_indices.loc`. Register more
tables (e.g. for `bwa`) at `galaxy-admin/configured/server/config/tool_data_table_conf.xml`.

#### Sudo

Enable `sudo` for the galaxy linux user.

```
sudo visudo
```

Insert the lines, replace `%%GA_USER%%` with the user id.

```
%%GA_USER%% ALL=(ALL) ALL
%%GA_USER%% ALL=(ALL) NOPASSWD: /usr/bin/systemctl
%%GA_USER%% ALL=(ALL) NOPASSWD: /usr/bin/journalctl
```

#### CentOS packages

```
sudo yum install \
  zlib-devel.x86_64 libxml2-devel.x86_64 libstdc++-devel.x86_64 \
  tmux tree most \
  httpd mod_ldap mod_xsendfile mod_ssl \
  postgresql-server \
  proftpd proftpd-postgresql proftpd-utils proftpd-ldap proftpd-devel
```

#### Slurm drmaa

Documentation: http://apps.man.poznan.pl/trac/slurm-drmaa

```
curl -# http://apps.man.poznan.pl/trac/slurm-drmaa/downloads/9 | tar xz
SLURMLIB=$(which srun)
SLURMLIB=${SLURMLIB%/bin/srun}
export LD_LIBRARY_PATH="$SLURMLIB:$LD_LIBRARY_PATH"
cd slurm-drmaa-1.0.7
./configure #CFLAGS="-g -O0"
make
sudo make install
cd
```

Now, test the configuration:

```
export DRMAA_LIBRARY_PATH=/usr/local/lib/libdrmaa.so
echo 'echo "Test executed on host $(hostname) by user $USER"' > test.drmaa
drmaa-run bash test.drmaa
```

#### PostgreSQL

Initialize and create the database and galaxy sql user.

```
sudo postgresql-setup initdb
sudo -u postgres bash -c "cd; psql << HERE
CREATE USER $GA_SQLUSER PASSWORD '$GA_SQLKEY';
ALTER ROLE $GA_SQLUSER with password '$GA_SQLKEY';
CREATE DATABASE $GA_SQLDB WITH OWNER = $GA_SQLUSER;
HERE
"
```

