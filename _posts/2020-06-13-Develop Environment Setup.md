---
title: "Development Environment Setup"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
---

## Install OS

1. Download image
2. Make startup disk
3. Plug USB Stick and Turn on
4. If computer doesn't boot with the startup disk, Check Boot Sequence at BIOS setting

## Adding Certificate

### Chrome

1. Go to settings
2. Search `Manage Certificate` under `Security`
3. Click `Authority` and import certificate

### Firefox

1. Go to preferences
2. Search `Certificates` and click `View Certificates`
3. Click `Authorities` and import certificate

### Terminal

1. Copy certificate at /usr/share/ca-certificates

    ```bash
    sudo cp my.crt /usr/share/ca-certificates/
    ```

2. Make the file's path relative to /etc/ca-certificates.conf by using command below

    - For interactively

      ```bash
      sudo dpkg-reconfigure ca-certificates
      ```

    - For non-interactively

        ```bash
        sudo update-ca-certificates
        ```

## Download code

1. Python Install

    ```bash
    sudo apt install python3 python-is-python3
    ```

2. Git install and config

    ```bash
    sudo apt install git
    ```

3. Repo install

    [Reference](https://source.android.com/setup/build/downloading#installing-repo)

    ```bash
    cd /usr/bin
    sudo curl https://storage.googleapis.com/git-repo-downloads/repo --output repo
    sudo chmod a+x repo
    ```

4. Repo init and sync

## Install Docker

[Reference](https://docs.docker.com/engine/install/ubuntu/)

0. Uninstall old versions

    ```bash
    sudo apt-get remove docker docker-engine docker.io containerd runc
    ```

1. Update apt package index and install a few things

    ```bash
    sudo apt update
    ```

    ```bash
    sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
    ```

2. Add Docker's GPG key

    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```

    Verify fingerprint

    ```bash
    sudo apt-key fingerprint 0EBFCD88
    ```

3. Add repository

    ```bash
    sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    ```

4. Install Docker

    ```bash
    sudo apt update
    ```

    ```bash
    sudo install docker-ce docker-ce-cli containerd.io
    ```

5. Verify install

    ```bash
    sudo docker run hello-world
    ```

### Managing Docker as a non-root user

[Reference](https://docs.docker.com/engine/install/linux-postinstall/)

1. Create docker group (It might be exist already)

    ```bash
    sudo groupadd docker
    ```

2. Add you to the docker group

    ```bash
    sudo usermod -aG docker $USER
    ```

3. Make it activate or re-login

    ```bash
    newgrp docker
    ```

4. Verify you can run docker without command sudo

    ```bash
    docker run hello-world
    ```

### Install docker-compose

1. Install docker-compose

    ```bash
    sudo apt install docker-compose
    ```

### Bring up container

1. Script docker-compose.yml like below

    ```yml
    version: "3"

    services:
        aosp:
            image: kylemanna/aosp:4.4-kitkat
            tty: true 
            volumes:
                - ./ccache:/tmp/ccache
                - ./aosp:/aosp
    ```

2.  Pull Docker image and Bring up container

    `up` command checks the compose file whether it is changed or not. if it is changed, It recreate container again.

    ```bash
    docker-composer up -d
    ```
