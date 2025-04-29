### What is this?
This repo shows how to set up a minimal FastAPI webserver on Amazon EC2

### Overview
1. Set up the EC2 machine
2. Perform initial set up of the machine
3. Set up an ssh keypair
4. Copy webserver application files
5. Set up webserver locally on EC2
6. Set up gunicorn service
7. Set up nginx service

### Setting up the EC2 Machine

This guide assumes we are running on an Ubuntu EC2 machine.

To set up the machine we will set up python, git, openssh-server, and nginx

### Performing initial set up of the machine

```bash
sudo apt-get update
sudo apt install python3-pip python3-virtualenv nginx curl
sudo apt install git-all
sudo apt install openssh-server
sudo apt install nginx
```

### Setting up an ssh keypair

To set up an ssh keypair: 

1. Create the keypair as a .pem first in the AWS account as outlined here https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/describe-keys.html
2. Then retrieve the public key from the generated pem file as described
3. As explained on the link, store the generated public key on the EC2 machine in the .ssh folder, in the authorized_keys file

### Copy webserver application files

You can do this either with git or with an ssh copy of the zipped files

```
scp -i ~/.ssh/<mykeypairfile>.pem  webserver-minimal.zip ubuntu@XYZ:/home/ubuntu/
```



