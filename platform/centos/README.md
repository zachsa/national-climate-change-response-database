# CentOS 7.6

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [(Optional) Configure SSH login](#optional-configure-ssh-login)
- [(Optional) Give your user passwordless sudo access](#optional-give-your-user-passwordless-sudo-access)
- [Install Nginx and Git](#install-nginx-and-git)
- [Install Docker](#install-docker)
  - [Configure SSL](#configure-ssl)
  - [Configure Nginx](#configure-nginx)
- [Install and configure firewalld](#install-and-configure-firewalld)
- [Setup continuous deployment via GitHub Actions](#setup-continuous-deployment-via-github-actions)
  - [self-hosted runner setup](#self-hosted-runner-setup)
- [Disable SELinux (or configure it correctly)](#disable-selinux-or-configure-it-correctly)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# (Optional) Configure SSH login

**Create an SSH key on your local PC**
```sh
ssh-keygen -t rsa
cat ~/.ssh/id_id_rsa.pub
```

**Add your key to the remote server**
```sh
mkdir ~/.ssh
chmod 700 ~/.ssh
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
vi ~/.ssh/authorized_keys # Copy your id_rsa.pub key into this file

# Configure public key authentication
sudo vi /etc/ssh/sshd_config
# set PubkeyAuthentication to 'yes'
# set PasswordAuthentication to 'yes' or 'no'

# Restart the SSH service
service sshd restart
```

# (Optional) Give your user passwordless sudo access

```sh
visudo

# Add this line to the bottom of the visudo file
<name> ALL=(ALL) NOPASSWD:ALL
```

# Install Nginx and Git

```sh
sudo yum install -y epel-release
sudo yum -y install https://packages.endpointdev.com/rhel/7/os/x86_64/endpoint-repo.x86_64.rpm
sudo yum -y install \
  nginx \
  git
sudo systemctl enable nginx
sudo systemctl start nginx
```

# Install Docker

```sh
sudo yum install -y yum-utils

sudo yum-config-manager \
  --add-repo \
  https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install -y \
  docker-ce \
  docker-ce-cli \
  containerd.io \
  docker-compose-plugin

sudo systemctl enable docker
sudo systemctl start docker
sudo docker swarm init
```

## Configure SSL
```sh
sudo su
mkdir /opt # Might already exist
mkdir /opt/ssl

# Generate dhparams
openssl dhparam -out /opt/ssl/dhparam.pem 2048
```

Obtain SSL certs. Or create self-signed certs for testing:

```sh
sudo openssl req -x509 -nodes -days 999 -newkey rsa:2048 -keyout /opt/ssl/<hostname>.key -out  /opt/ssl/<hostname>.cer
```

## Configure Nginx
Reference nginx files are defined in the `platform/centos/nginx` folder. The server blocks should be updated for the correct hostname, and for SSL cert paths

# Install and configure firewalld
Setup a firewall and explicitly allow:

- Public HTTP traffic
- Public HTTPS traffic
- Public SQL Server connections (depending on how SQL Server is setup)

```sh
sudo yum -y install firewalld \
  && systemctl unmask firewalld \
  && systemctl enable firewalld \
  && systemctl start firewalld \
  && sudo firewall-cmd --permanent --zone=public --add-service=http \
  && firewall-cmd --permanent --zone=public --add-service=https \
  && sudo firewall-cmd --permanent --add-port=1433/tcp \
  && firewall-cmd --reload
```

# Setup continuous deployment via GitHub Actions
Either use the Microsoft-provided shared runners, or setup a self-hosted runner to run GitHub Actions workflows. In either case a limited permissions user is required. Depending on the exact server setup, this user may/may not need access to be a member of the `docker` group.

```sh
sudo adduser runner
sudo passwd runner # this should be a strong password
sudo usermod -aG docker runner
```

## self-hosted runner setup
Follow GitHub's instructions to install a self-hosted runner, and then adjust the Actions workflows accordingly.

# Disable SELinux (or configure it correctly)

```sh
sudo vi /etc/selinux/config

# Make sure this line is uncommented
SELINUX=disabled

# Reboot the server
reboot
```
