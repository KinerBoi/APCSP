---
toc: true
title: Flask/Python Deployment Guide
layout: post
description: Description of key methods process used to deploy a Flask/Python website; AWS EC2, Docker, docker-compose, and Nginx 
permalink: /techtalk/deploy
image: /images/aws.png
categories: [6.B]
tags: [aws, ec2. docker, nginx, certbot, dns]
type: pbl
week: 6
---

## Deployment Overview
Deploying a Web Application enables a Server and Web Application to be available to users on the Internet. Development Operations (DevOps) is understanding the tools and service used to enable the Web Application on the Internet.  This process can use many different cloud services, as well as choice of many different tools.  Here is an overview of what is covered in this procedure.

- EC2: Amazon Web Services is a cloud computing platform that the PUSD district has provided for their students to serve our Web Application.
- GitHub: The leading open platform to share a code across the Internet.
- Docker and docker-compose: Used to host a Web Application.  A Docker container prepares an environment that contains the Web Application code and all the dependencies (requirements.txt for Python)  Docker is an open platform for developing, shipping, and running applications.
- Nginx: In order to find a Web Application on a server, there needs to be a process that listens for the Web Application request and directs it to the Web Application service.  Nginx is an open source software for web serving, reverse proxy, caching, load balancing, media streaming, and more.
- Certbot: Web traffic on internet is reliably served over Secure Hyper Text Transfer Protocol (https).  Certbot is a free, open source software tool for automatically using Let's Encrypt certifications.
- DNS: Natively, the web works off of IP addresses.  Domain Name Services (DNS) allows the assignment of a friendly name to a Web Server.  This name is built into Nginx/Certbot configuration files.  Freenom is the cloud service described in this blog and has been used to register the nighthawkcodingsociety.com domain.

### Key/Values required as you go through these procedures
Listed are Keys, you need to obtain "values" specific to your Web Application setup.  It is important that you recognize the sample "values" as you work through these procedures, then replace them with the "values" that are specific to your Web Application use case.

- GitHub HTTPS link:
- IAM user:
- EC2 name:
- EC2 Public IPs:
- DNS Name:
- DNS Subdomain name(s)
- Docker Port:
- docker-compose, proxy pass Port:
- docker-compose, docker Image name:
- Nginx server file(s):

## Server Setup and Initial Deployment
> Development Operations (DevOps) begins with server setup.    
### Amazon Web Services (AWS): Electric Cloud Compute (EC2) Setup
Preparing and AWS EC2 instance is the process of creating a cloud computer.  This process starts by logging into your AWS IAM user, searching for EC2.
- To get started, launch a new AWS EC2 instance to learn process and understand how to work with Linux. here are some key considerations.
    - Choose an Amazon Machine Image (AMI), the class will be using Ubuntu you should check on last verified version with a Teacher before proceeding
    - When it comes to picking memory or disk it is VERY important to pick Free Tier.  As stated, this will only be used for testing and then it will be disposed for cost efficiency.
    - When presented with access dialog for http and https, make sure you check these boxes.  Remember you are making a Web application that will run over http and https.
    - Name the security group (.pem) file after your self or team.  It may be necessary to use SSH to access your EC2 instance.
    - The remainder of the steps you can use the defaults, refer to AWS documentation for guidance: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html
    - At the end of this process you need to "Connect to Instance". This will provide you a terminal like experience.


## Deployment Process 
> Deployment begins with terminal connections.   It is necessary to run commands to setup files to enable your Web Application to run on the Internet.

### Connect to EC2 through cloud
> Connect to an Ubuntu EC2 instance on AWS and then begin the system and software setup.
- Select your machine and press "Connect" at top middle.
![AWS Connect]({{site.baseurl}}/images/aws_connect.png)
- On next screen ensure things are lined "EC2 Instance Connect" and press "Connect" at bottom.
![AWS2 Connect]({{site.baseurl}}/images/aws_connect2.png)
- Only if this fails you will need to connect from our Anaconda terminal, we need pem file so place it in your home directory.
![AWS SSH]({{site.baseurl}}/images/aws_ssh.png)
- Either connection should work at home, school WIFI may be blocking, I have been trying to get this open.  A Hot spot or [VPN]({{site.baseurl}}/tutorial/vpn) will solve connection errors.


### Update, Upgrade, and Install packages
Terminal commands are shown, these commands will be run from Terminal after you connect to your EC2 instance.  It is advised that you try to comprehend and learn these commands as you go through these procedures.  Understanding steps is how you become a DevOps Engineer.  

> These commands update and then upgrade packages in your system.

```bash
$ sudo apt update; sudo apt upgrade
$ sudo apt install python3-pip nginx
$ sudo pip3 install virtualenv
```

### Clone and Change Directory to project location
> This command moves your Web Application code onto you EC2 cloud computer. The example GithUb HTTPs Link is demonstration is: https://github.com/nighthawkcoders/flask_portfolio.git.  

```bash
$ cd
$ git clone https://github.com/nighthawkcoders/flask_portfolio.git
$ cd flask_portfolio
```

* Clone needs to be repeated for each Web application on the EC2 instance (Team repository and each individual repository).  If repo is named the same use option parameter at the end of clone command ```git clone https://github.com/nighthawkcoders/flask_portfolio.git flask_portfolio_john```, then ```cd flask_portfolio_john```

### Test Web Service
> These steps will require you to understand a few new commands that will not be part of final deployment process.  However, these commands will help you understand machine dependencies and validate your requirements.txt for completeness.

```bash
$ cd ~/flask_portfolio
$ virtualenv -p /usr/bin/python3 webapp
$ source webapp/bin/activate
$ pip install -r requirements.txt
$ python main.py
```

* Failed result.  If you get a result that looks like the below, your requirements.txt needs work.  In this instance, I would need to add flask to the requirements.txt.

```bash
(webapp) ubuntu@ip-172-31-1-138:~/flask_portfolio$ python main.py
Traceback (most recent call last):
  File "main.py", line 2, in <module>
    from flask import Flask, render_template
ModuleNotFoundError: No module named 'flask'
(webapp) ubuntu@ip-172-31-1-138:~/flask_portfolio$ 
```

* Correct result. A successful result will look like the following.  At this point we will type "ctrl+c" and then at prompt "deactivate", as we will now build and run a Docker File to deploy in more automated fashion.

```bash
(webapp) ubuntu@ip-172-31-1-138:~/flask_portfolio$ python main.py
 * Serving Flask app 'main' (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
 * Running on http://127.0.0.1:5000 (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 710-199-540
```

### Create Dockerfile to run Web Service
> A Dockerfile is a configuration used to run the Web Service.  This is placed in a file called Dockerfile.  It is best to add this to VS Code and pull it, or you can update in place with nano, vi, or vim editor and use command line commands to push it into your repository.  The Dockerfile should be considered Code!

> Edit the Dockerfile

```bash
$ sudo nano Dockerfile
```

* Insert the Dockerfile commands, note that they are similar to Bash commands performed earlier.  Follow prompts on screen to save file when complete, look for Key/Value (GitHub HTTPS link) that requires change for your project.
```dockerfile
FROM docker.io/python:3.9
WORKDIR /app
# --- Update environment and install python and pip ---
RUN apt-get update && apt-get upgrade -y && \
    apt-get install -y python3 python3-pip git
# --- This should be the repository you created last week ---
RUN git clone https://github.com/nighthawkcoders/flask_portfolio /app
# --- Install project specific dependencies ---
RUN pip3 install --no-cache-dir -r requirements.txt
RUN pip3 install gunicorn
# --- Setup args to run 3 workers and run on port 8080 ---
ENV GUNICORN_CMD_ARGS="--workers=3 --bind=0.0.0.0:8080"
# --- Allow port 8080 to be accessed by system ---
EXPOSE 8080
# --- Run Web Application in production style ---
CMD [ "gunicorn", "main:app" ]
```


### Create docker-compose file share Web Service
> A docker-compose file is a configuration used to share your Docker Web Service and resources with the Linux system.  This file enable Linux to have access to the container and the persistent data application via the /volumes location.  
- Once again it is best to add the docker-compose.yml in VS Code and pull it.  You can edit it on the machine itself using vi, vim, or nano.
- The ```image:``` needs to have unique name for each application on server (ie flask_john_v1)
- The ```ports:``` left value 8086 needs to be unique for each application on server (ie 8087:8080, 8088:8080, etc)

> Edit docker-compose.yml

```bash
$ sudo nano docker-compose.yml
```

* Insert the docker-compose.yml Keys/Values.
```yml
version: '3'
services:
        web:
                image: flask_port_v1
                build: .
                ports:
                        - "8086:8080"
                volumes:
                        - persistent_volume:/app/volumes
volumes:
  persistent_volume:
    driver: local
    driver_opts:
      o: bind
      type: none
      device: /home/ubuntu/flask_portfolio/volumes
      # replace just flask_portfolio
```

### Running Docker using docker-compose.yml
> At this point, it is best to review complete files on GitHub and for Docker and docker-compose: https://github.com/nighthawkcoders/flask_portfolio.   Review the Key/Values mentioned in this document.  Make sure your Docker and docker-compose files a personalized to your project.

* Make sure you are in project directory
```bash
$ cd ~/flask_portfolio/
```

> install docker-compose

```bash
$ sudo apt install docker-compose -y
```

> Run docker-compose

```bash
$ sudo docker-compose up -d
```

* Output from docker-compose.  When running this command, docker-compose will run all the Docker steps and build a Web Application running in a Docker container, a virtual environment.

```bash
Creating network "flask_portfolio_default" with the default driver
Building web
Step 1/9 : FROM docker.io/python:3.9
 ---> d0ce03c9330c
Step 2/9 : WORKDIR /app

.... LOTS of STEPs and OUTPUT ...

Successfully built 68d68ad9699b
Successfully tagged flask_port_v1:latest
WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Creating flask_portfolio_web_1 ... done
```

### Verifying Web Application via Docker commands
> Here is a look at some of the commands behind the scenes.  None of these are required to get things working, but show the results of the Docker and docker-compose.yml files and commands.
- docker-compose ps   The running Web process, "ps" is a linux command or option that provides information related to the processes on a system.  Look at headings in relation to outputs of the docker-compose process.

```bash
 ubuntu@ip-172-31-1-138:~/flask_portfolio$ sudo docker-compose ps
        Name                 Command        State                    Ports                  
--------------------------------------------------------------------------------------------
flask_portfolio_web_1   gunicorn main:app   Up      0.0.0.0:8086->8080/tcp,:::8086->8080/tcp
```

- docker ps   A more comprehensive list of all the docker processes on the system.  In this process reports, many of the alternate projects running on this AWS server are show.  The flask_portfolio_web_1 process is the items specific to this tutorial.
```bash
buntu@ip-172-31-1-138:~/flask_portfolio$ sudo docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                                       NAMES
749a93bc11ce   flask_port_v1   "gunicorn main:app"      45 minutes ago   Up 45 minutes   0.0.0.0:8086->8080/tcp, :::8086->8080/tcp   flask_portfolio_web_1
89155782b853   java_springv1   "java -jar target/sp…"   6 days ago       Up 6 days       0.0.0.0:8085->8080/tcp, :::8085->8080/tcp   spring_portfolio_web_1
9415d6397d2e   python_cspv1    "gunicorn main:app"      2 weeks ago      Up 2 weeks      0.0.0.0:8082->8080/tcp, :::8082->8080/tcp   nighthawk_csp_web_1
4bf324458bf6   python_laxv1    "gunicorn main:app"      5 weeks ago      Up 5 weeks      0.0.0.0:8084->8080/tcp, :::8084->8080/tcp   lax_web_1
7a6dff6425e9   python_ctev1    "gunicorn main:app"      5 weeks ago      Up 5 weeks      0.0.0.0:8083->8080/tcp, :::8083->8080/tcp   cte_web_1
abd77b8e77af   java_csav2      "java -jar target/cs…"   5 weeks ago      Up 5 weeks      0.0.0.0:8081->8080/tcp, :::8081->8080/tcp   nighthawk_csa_web_1
```

- docker images   This lists all of the docker images, or containers, that are used to serve the process shown above.  The flask_port_v1 is the image created from the Docker file created in this tutorial.  The image contains the running Web application.
```bash
ubuntu@ip-172-31-1-138:~/flask_portfolio$ sudo docker images
REPOSITORY      TAG             IMAGE ID       CREATED          SIZE
flask_port_v1   latest          68d68ad9699b   51 minutes ago   1.01GB
java_springv1   latest          e85a584b1836   6 days ago       523MB
python_laxv1    latest          713c84a30d3b   5 weeks ago      1.16GB
python_ctev1    latest          1608eaee06c7   5 weeks ago      1.18GB
python_cspv1    latest          b9bb27be863b   5 weeks ago      1.12GB
java_csav2      latest          4055a9fd5ea7   5 weeks ago      570MB
python          3.9             d0ce03c9330c   7 weeks ago      915MB
alpine          latest          e66264b98777   8 weeks ago      5.53MB
openjdk         16-alpine3.13   2aa8569968b8   17 months ago    324MB
```

### Testing localhost endpoint
Verify everything is working with your docker configurations.  

* Local Test of Web Application Endpoint.  This should return HTML related to the home page of your Web site.  If this fails, you need to review Docker and docker-compose configurations.  ```Failed to connect``` means review your port on docker-compose.

```bash
$ curl http://localhost:8086
```

## Preparing the Docker Web Application for Internet Access
There are a couple of steps to this preparation. We need to direct the internet to the Server running the Web Application, this is done using Domain Name Service (DNS).   After being directed to the Web Server, the server needs to respond to the Hyper Text Transfer Protocol (HTTP), this will be manged by Nginx.   Additionally, we will be required to support Secure HTTP (HTTPS), a utility called Certbot will augment our Nginx configuration with a certificate.

### DNS provider and setup
Each student scrum team is required to learn how to obtain a DNS provider and setup an independent domain.  However, the final set up will be using a Subdomain under nighthawkcodingsociety.com.

A picture is included to show key elements in setting up a domain with a DNS provider.  The nighthawkcodingsociety.com is using Freenom as its service provider.  As you build your own DNS server you will need to obtain your own IP address and domain.

This illustration is dependent on...
- EC2 Public IPs: 3.233.212.71
- DNS Name: nighthawkcodingsociety.com
- DNS Subdomain name(s): battleship.nighthawkcodingsociety.com. cowboys.nighthawkcodingsociety.com

A minimum configuration will have the two "A" type definitions using you Public IP address.  These two are resolved with a single Web Application.  The "CNAME" type is used for subdomains, these will resolve to a different Web Application.

<img alt="Setup a Domain" src="{{site.baseurl}}/images/freenom.png" title="DNS Provider">


### Nginx install, configuration, and services
Each student scrum team will perform Nginx installation and setup on an AWS EC2 test server.  The final configuration will be on AWS server managed by Teachers or Student DevOps Engineers.

Enable Nginx to retrieve Python Web Application on internet request (Reverse Proxy)! Make a server file located at /etc/nginx/sites-available/nighthawk.

* Install Nginx on Ubuntu servers
```bash
$ sudo apt install nginx
```

* Go to location of Nginx server configuration files
```bash
$ cd /etc/nginx/sites-available
```

* Open editor to Create your own "Nginx server configuration".  For clarity, the name ```nighthawk``` should reflect your application name, domain or subdomain.  The name needs to be unique for each web application (ie john_nighthawk).
```bash
$ sudo nano nighthawk
```

* Edit your own Nginx server configuration making modifications to <mark>primary server</mark> file, (1 is required):
    * DNS Name(s): nighthawkcodingsociety.com www.nighthawkcodingsociety.com
    * docker-compose, proxy pass Port: 8086

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name nighthawkcodingsociety.com www.nighthawkcodingsociety.com;

    location / {
        proxy_pass http://localhost:8086;
        # Simple requests
        if ($request_method ~* "(GET|POST)") {
                add_header "Access-Control-Allow-Origin"  *;
        }

        # Preflight requests
        if ($request_method = OPTIONS ) {
                add_header "Access-Control-Allow-Origin"  *;
                add_header "Access-Control-Allow-Methods" "GET, POST, OPTIONS, HEAD";
                add_header "Access-Control-Allow-Headers" "Authorization, Origin, X-Requested-With, Content-Type, Accept";
                return 200;
        }
    }
}
```

* Or, or in addition, edit your own Nginx server configuration making modifications to <mark>subdomain</mark> file (0 to many):
    * DNS Name(s): flask.nighthawkcodingsociety.com
    * docker-compose, proxy pass Port: 8086

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name flask.nighthawkcodingsociety.com;

    location / {
        proxy_pass http://localhost:8086;
        # Simple requests
        if ($request_method ~* "(GET|POST)") {
                add_header "Access-Control-Allow-Origin"  *;
        }

        # Preflight requests
        if ($request_method = OPTIONS ) {
                add_header "Access-Control-Allow-Origin"  *;
                add_header "Access-Control-Allow-Methods" "GET, POST, OPTIONS, HEAD";
                add_header "Access-Control-Allow-Headers" "Authorization, Origin, X-Requested-With, Content-Type, Accept";
                return 200;
        }
    }
}
```

* Activate/enabled Nginx server configuration:
  * nginx configuration file: nighthawk
```bash
$ sudo ln -s /etc/nginx/sites-available/nighthawk /etc/nginx/sites-enabled
$ sudo nginx -t
```

* If there are no errors, restart NGINX so the server is an endpoint to the internet:
```bash
$ sudo systemctl restart nginx
```

### Testing HTTP endpoint
Before finishing, this is a good opportunity to review everything you have done.  

* Make sure curl is still working on local machine.

```bash
$ curl http://localhost:8086
```

* Now test unsecure HTTP endpoint on the internet.  Go to a browser anywhere and type your DNS domain: ```http://nighthawkcodingsociety.com```. 
   * Timeout.  This means something is wrong with EC2 Public IP.
   * Nginx Default page.  This means DNS is working, but something is wrong with you Nginx configuration.
   * Broken Gateway.  This means Nginx is working, but something is wrong with Web Application endpoint on machine, if this fails something is wrong with Web Application.  This requires you to look at Docker and docker-compose configuration.
   

### Certbot install and configuration
Each student scrum team will learn Certbot on on AWS EC2 test server, establish working https web application.  The final configuration will be on AWS server managed by Teachers or Student DevOps Engineers.

```bash
$ sudo snap install core; sudo snap refresh core
$ sudo snap install --classic certbot
$ sudo ln -s /snap/bin/certbot /usr/bin/certbot
$ sudo certbot --nginx
```

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx

Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: nighthawkcodingsociety.com
2: csa.nighthawkcodingsociety.com
3: csp.nighthawkcodingsociety.com
4: flm.nighthawkcodingsociety.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel):    
Cert not yet due for renewal

You have an existing certificate that has exactly the same domains or certificate name you requested and isn't close to expiry.
(ref: /etc/letsencrypt/renewal/nighthawkcodingsociety.com-0001.conf)

What would you like to do?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: Attempt to reinstall this existing certificate
2: Renew & replace the cert (limit ~5 per 7 days)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for nighthawkcodingsociety.com
http-01 challenge for csa.nighthawkcodingsociety.com
http-01 challenge for cso.nighthawkcodingsociety.com
http-01 challenge for flm.nighthawkcodingsociety.com
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/nighthawk_society
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/nighthawk_csa
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/nighthawk_csp
Deploying Certificate to VirtualHost /etc/nginx/sites-enabled/nighthawk_flm

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Traffic on port 80 already redirecting to ssl in /etc/nginx/sites-enabled/nighthawk_society
Traffic on port 80 already redirecting to ssl in /etc/nginx/sites-enabled/nighthawk_csa
Traffic on port 80 already redirecting to ssl in /etc/nginx/sites-enabled/nighthawk_csp
Traffic on port 80 already redirecting to ssl in /etc/nginx/sites-enabled/nighthawk_flm

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Your existing certificate has been successfully renewed, and the new certificate
has been installed.

The new certificate covers the following domains:
https://nighthawkcodingsociety.com, 
https://csa.nighthawkcodingsociety.com, 
https://csp.nighthawkcodingsociety.com, and
https://flm.nighthawkcodingsociety.com,

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=nighthawkcodingsociety.com
https://www.ssllabs.com/ssltest/analyze.html?d=csa.nighthawkcodingsociety.com
https://www.ssllabs.com/ssltest/analyze.html?d=csp.nighthawkcodingsociety.com
https://www.ssllabs.com/ssltest/analyze.html?d=flm.nighthawkcodingsociety.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/nighthawkcodingsociety.com-0001/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/nighthawkcodingsociety.com-0001/privkey.pem
   Your cert will expire on 2022-03-06. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

## Update Deployment, this process is after you change code
> This procedure is a very short, as much of the deployment performed is persistent on your EC2/Ubuntu.  

### Goto Project directory 

> Check your docker processes.  Make sure git has nothing to commit.

```bash
$ cd ~/flask_portfolio
$ sudo docker-compose ps
       Name                 Command        State                    Ports                  
--------------------------------------------------------------------------------------------
flask_portfolio_web_1   gunicorn main:app   Up      0.0.0.0:8086->8080/tcp,:::8086->8080/tcp

$ sudo docker ps
CONTAINER ID   IMAGE           COMMAND                  CREATED        STATUS        PORTS                                       NAMES
ee3fd0edfdcd   flask_port_v1   "gunicorn main:app"      6 weeks ago    Up 6 weeks    0.0.0.0:8086->8080/tcp, :::8086->8080/tcp   flask_portfolio_web_1

$ git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
```

### Shutdown process and update source

> Stop docker processes and git pull
```bash
$ sudo docker-compose kill  # run ps on docker-compose and docker and you should observe changes
Killing flask_portfolio_web_1 ... done
```

> At this point your server is down, look at it from browser
```
502 Bad Gateway
```

> Update code

```bash
$ git pull    # dialog will look as follows
remote: Enumerating objects: 42, done.
remote: Counting objects: 100% (42/42), done.
remote: Compressing objects: 100% (23/23), done.
remote: Total 33 (delta 17), reused 24 (delta 9), pack-reused 0
Unpacking objects: 100% (33/33), 7.42 KiB | 474.00 KiB/s, done.
From https://github.com/nighthawkcoders/flask_portfolio
   f0b6ba1..def0da9  main       -> origin/main

... These are the files you changed ...

model_jokes.py                                 |   4 ++--
requirements.txt                               |   8 ++++----
```

### Rebuild and Restart Web Application

> Force rebuild of docker container.  This can take a couple of minutes.

```bash
$ sudo docker-compose build --no-cache
Building web
Step 1/9 : FROM docker.io/python:3.9
 ---> d0ce03c9330c
Step 2/9 : WORKDIR /app
 ---> Running in 3f705fefc681
Removing intermediate container 3f705fefc681
 ---> 9c6ebd23dcb1

... Build steps 1 through 9 ...

Step 9/9 : CMD [ "gunicorn", "main:app" ]
 ---> Running in 9ca595a79669
Removing intermediate container 9ca595a79669
 ---> 54ca7ab064b4
Successfully built 54ca7ab064b4
Successfully tagged flask_port_v1:latest
```

> Run docker-compose
```bash
$ sudo docker-compose up -d
Recreating flask_portfolio_web_1 ... done
```

> Now server is up, test in Browser for recent change
```
Python Home Page
```