---
layout: post
title: 'OpenSSH: Introduction to ssh server setup'
date: 2024-09-22 12:46:00 -0500
categories: posts Security SSH
tags: Tools, OpenSSH, SSH, DevOps, Security
---

In todays world, security is paramount, especially when it comes to communication over networks. In order to protect sensitive data and unauthorized access, it is important that all interactions over internet occur through secure channels. Especially when comes to running commands remotely on a server, we need secure methods to avoid risks such as `man-in-the-middle` (MITM) attacks. Just as `HTTPS` secures communication in web application, `Secure Shell` (SSH) provides a secure channel to access terminal of a remote server. ssh ensures that your commands and encrypted and safely transmitted over the network.

In this block, we'll explore on setting up a ssh server, creating users and authentication based on passwords and ssh keys. In this blog we will leverage docker and OpenSSH (open source implementation of `ssh` protocol).

### Creating SSH Server

Here we will use `sshd` which is a OpenSSH server program.

```dockerfile
FROM ubuntu:20.04
RUN apt update \
    && apt install openssh-server -y \
    && apt clean \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/* \
    && service ssh start
EXPOSE 22
CMD ["/usr/sbin/sshd","-D"]
```

### Creating SSH Client

In practice, this would be the machine of any one of the user.
Here we will use OpenSSH client program `ssh`.

```dockerfile
FROM ubuntu:20.04
RUN apt update \
    && apt install openssh-client -y \
    && apt clean \
    && rm -rf /var/lib/apt/lists/* /var/cache/apt/archives/*
```

### Combine Using Docker compose file

```compose
networks:
  ssh_network:

services:
  ssh-server:
    container_name: ssh-server-container
    image: ssh-server
    build:
      context: .
      dockerfile: Dockerfile
    networks:
      - ssh_network
    ports:
      - 2022:22
  ssh-client:
    container_name: ssh-client-container
    image: ssh-client
    stdin_open: true #Keep stdin open, so that container doesn't exit
    build:
      context: .
      dockerfile: ClientDockerfile
    networks:
      - ssh_network
```

### Deployment

Run the docker compose file from your terminal and you can see the containers running in your machine.

![list of docker containers](/assets/images/ssh-server-setup/ssh-deployment.png)

### Creating a User

- sh into the ssh server container

  ```bash
  $ docker exec -it ssh-server-container sh
  ```

- create a ssh user, once the user is created check the home directory, it should contain a directory with the name of the user on it.

  ```bash
  # adduser ssh-user1
  ```

  ![ssh server home directory](/assets/images/ssh-server-setup/ssh-user1-dir-created-ss.png)

- Run tail commands of `/etc/passwd` and `/etc/shadow` to see if the user and password is created for that user.

  ![tail command on /etc/passwd](/assets/images/ssh-server-setup/ssh-user1-passwd-ss.png)

  ![tail command on /etc/shadow](/assets/images/ssh-server-setup/ssh-user1-shadow-ss.png)

- Now that we have a user, let sh into client container. In real scenario this would actually be the machine of the user.

  ```bash
  $ docker exec -it ssh-client-container sh
  ```

- Now lets login to ssh server.

  ```bash
  # ssh ssh-user1@ssh-server-container
  ```

  > Note: if this is your first time connecting to this server it will prompt you with either a `ECDSA` or `ED25519` fingerprint of the ssh server. Before accepting, verify the fingerprint. if you have access to the server you can run the following command on the server and compare it with the one on the ssh prompt.

  ```bash
  # ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
  # ssh-keygen -lf /etc/ssh/ssh_host_ecdsa_key.pub
  ```

Once you accept, the fingerprint will be added to the `known_hosts` file, and it will ask for the password for the user.

since you logged in as `ssh-user1`, you will be inside the `home/ssh-user1` directory.

![user1-home-directory](/assets/images/ssh-server-setup/ssh-user1-home-dir-ss.png)

### Set up public key authentication

1. Turn off Password Authentication (Optional)

   - install your choice of editor inside the ssh server

   ```bash
   # apt update
   # apt install vim
   # vi /etc/ssh/sshd_config
   ```

   - Make sure to read comments at the top of the file.
   - Find the below lines in the file

   ```text
   # To disable tunneled clear text passwords, change to no here!
   # PasswordAuthentication yes
   #PermitEmptyPasswords no
   ```

   - uncomment PasswordAuthentication and change it to no

   ```text
   # To disable tunneled clear text passwords, change to no here!
   PasswordAuthentication no
   #PermitEmptyPasswords no
   ```

   - save the file and restart your ssh service, or in this case restarting container will also work.

   ![error on password login](/assets/images/ssh-server-setup/ssh-user1-passwd-denied-ss.png)

2. Make sure the default for `PubkeyAuthentication` is `yes` in `etc/ssh/sshd_config` file, else override it to yes.

3. generate ssh keys inside ssh client container

   ```bash
   $ ssh-keygen -t rsa
   ```

   ![ssh key generation](/assets/images/ssh-server-setup/ssh-key-gen-user1-ss.png)

4. Copy the public key to `home/ssh-user1/.ssh/authorized_keys` of ssh server. since copying from one container to another is not supported in docker. i'll use the combination of `cat` and `tee` to copy the file.

   ```bash
   $ docker exec -i ssh-client-container cat /root/.ssh/id_rsa.pub | docker exec -i ssh-server-container tee /home/ssh-user1/.ssh/authorized_keys
   ```

   > Make sure to create `.ssh` folder inside the `home/ssh-user1` before running above command.

5. Make sure that the `ssh-user1` is the owner of the `.ssh` and `authorized_keys` inside `home/ssh-user1` of the ssh server.
   you can change the ownership by logging in ssh server as a root and then run the following. The Recursive flag `-R` makes sure everything inside `ssh-user1` directory is also owned by `ssh-user1`.

```bash
$ chown -R ssh-user1 .ssh
```

6. Now from the client container you can run `ssh ssh-user1@ssh-server-container` to login using public key.

#### Instead of creating a seperate client container, you could as easily use your own host machine to connect to the ssh server with the same user or create a different one for fun.

In ssh server:

```bash
$ adduser newuser
$ cd /home/newuser
$ mkdir .ssh
```

In your host machine:

```bash
$ ssh-keygen -t rsa
$ docker cp ~/.ssh/id_rsa_newuser.pub ssh-server-container:/home/newuser/.ssh/authorized_keys
$ ssh new_user@localhost -p 2022
```

Permissions in ssh server:

```bash
$ chown -R newuser .ssh
```

The ssh server port 22 is mapped to the port 2022 of the host machine, so you can login from the host machine with the following command `ssh -i ~/.ssh/id_rsa_newuser newuser@localhost -p 2022`.

In case the ssh connection fails you can add `-v` or `-vvv` in the command for more detailed information.

[source code](https://github.com/bad-logic/devops/tree/main/ssh)
