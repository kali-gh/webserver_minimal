### Overview
This repo shows how to set up a minimal FastAPI (async) webserver on Amazon EC2. We show all the steps needed to get a "Hello World" async response from the server, including the following components:

- A simple "Hello World" FastAPI application in python
- A gunicorn service running on the server
- An nginx reverse proxy that routes requests to the gunicorn service.

Combined these elements allow the server to respond asynchronously to requests, following best practices to manage load. The serve can optionally sit behind a load balancer (this is not shown here).

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
sudo apt install gunicorn3
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

Directory structure on the EC2
```
/home/ubuntu/
/home/ubuntu/app < webserver files located here
```


### Set up local webserver on EC2

To test the server works locally you can do from within the app directory:

Set up the python enviornment with uvicorn and fastapi (see requirements.txt file)
```
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```


Start the server. This just calls the app which deployed on /api/test.  Note: this runs : uvicorn app:app --reload
```
./run.sh
```

In another terminal check you get the response
```
 curl http://127.0.0.1:8000/api/test 
```

You should see
```
"Hello World!"
```

### Serve the application from gunicorn


We need a gunicorn service to serve our application. To do this, run this command

First create the socket

``` 
sudo vim /etc/systemd/system/gunicorn.socket
```

Paste the following
``` 
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

Start the socket

```
sudo systemctl start gunicorn.socket
```

Next create the service: 
```
sudo vim /etc/systemd/system/gunicorn.service
```

and paste the following

```
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/app/
ExecStart=/home/ubuntu/app/venv/bin/gunicorn \
          --access-logfile - \
          --workers 5 \
          --bind unix:/run/gunicorn.sock \
          --worker-class uvicorn.workers.UvicornWorker \
          app:app

[Install]
WantedBy=multi-user.target
```

Now run these commands to serve the application
```
sudo systemctl restart gunicorn
```

You should be able to run the status and see the service up
```
sudo systemctl status gunicorn
```

### Set up nginx
```
sudo vim /etc/nginx/sites-available/default
```

Add the following in the server below location /

```
location /api {
        proxy_pass http://unix:/run/gunicorn.sock;
}
```

you should now be able to run
```
curl http://localhost/api/test
```

You can verify the logs located at 
```
/var/log/nginx/access.log
```
