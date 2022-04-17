# Lab2-ST0263

<details>
  <summary>Table of Contents</summary>
  <ol>
    <li><a href="#setup">Setup</a></li>
        <ol>
            <li><a href="#ec2">EC2</a></li>
            <li><a href="#install-docker">Install Docker</a></li>
            <li><a href="#nginx">Nginx</a></li>
        </ol>
    <li><a href="#flow">Flow</a></li>
    <li><a href="#testing">Testing</a></li>
  </ol>
</details>

## Project Objetive

The objetive of this project is to explore docker and nginx to deploy an CMS in this case Wordpress and encript the CMS using SSL Certificated provided by Let's Encript. 

## Setup

### EC2

For this project please install any virtual machine or instance on DigitalOcean Droplet, AWS EC2, Google Cloud Compute Engine, Ibm Cloud.

In this case the machine will be created on AWS EC2.

#### Machine setup

For the EC2 instance for demostrative reasons, create T2 MICRO (Free tier) option.

#### VPC

For the domain you will need to create an elastic IP address 

[How to create Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)

### Install Docker

For this lab you need to install 
* Docker 
* Docker-Compose

#### Docker

To install docker please follow the steps from the oficial docker webpage

[How to install docker](https://docs.docker.com/engine/install/ubuntu/)

#### Docker-COmpose

To install docker-compose please follow the steps from the oficial docker webpage

[How to install docker-compose](https://docs.docker.com/compose/install/)

#### NOTE

Please follow the steps always on linux.

#### RUN DOCKER

When everthing is done please run this command where is located your docker-compose.yml 

```bash
sudo docker-compose up -d
```
if you wanna see the logs run on the same location

```bash
sudo docker-compose logs
```

### Nginx

To install NGINX you need to put this commands on your machine

```bash
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update
sudo apt install certbot python3-certbot-nginx
```

The last command is fundamental to get the SSL certificate on your machine.

#### Setup NGINX

When you install CERTBOT let's generate the cerfiticate but first you need to adquire a domain on this case we are going to use [Freenom](https://www.freenom.com/) 

My domain is daves.tk so let's generate the certificate

```bash
sudo certbot --nginx certonly -d daves.tk -d www.daves.tk
```
Certbot will request you to add your personal email.

when the certificate is done it will be saved on 

```bash
/etc/letsencrypt/live/daves.tk/fullchain.pem
/etc/letsencrypt/live/daves.tk/privkey.pem
```

To renew your cert run this command

```bash
sudo certbot renew --dry-run
```

Finally run this command and edit Nginx conf file

```bash
sudo nano /etc/nginx/nginx.conf
```

On the part of HTTP next to 

```basb
include /etc/nginx/conf.d/*.conf;
include /etc/nginx/sites-enabled/*;
```
add this lines

```bash
server {
            listen 80;
            server_name daves.tk;
            rewrite ^(.*) https://$host$1 permanent;
}

server {
    listen 443;
    server_name daves.tk;
    ssl on;
    ssl_certificate /etc/letsencrypt/live/daves.tk/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/daves.tk/privkey.pem;
    proxy_redirect off;
    location / {
        proxy_set_header        Host $host:$server_port;
        proxy_set_header        X-Real-IP $remote_addr;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_>
        proxy_set_header        X-Forwarded-Proto $scheme;
        proxy_pass http://172.16.238.201/;
    }
}
```

We are using NGINX Reverse proxy to redirect any request form https://daves.tk/ or http://daves.tk/ to the private ip address from the docker-compose Wordpress that shows on this line.
```bash
proxy_pass http://172.16.238.201/;
```

When nginx.conf is done run this commands

```bash
sudo nginx -t
sudo service nginx restart
```

##### Demo
```bash
ubuntu@ip-x-x-x-x:~$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
ubuntu@ip-x-x-x-x:~$ sudo service nginx restart
 * Restarting nginx                                                           [ OK ]
ashley@pluto:~$
```

## Flow

![Untitled Diagram drawio](https://user-images.githubusercontent.com/53051438/163523068-5f1e418d-22fb-4065-9a99-81a5b1207e8c.png)

## Testing

visit the website https://daves.tk/ and check is your browser indicates that your connection is secure

![image](https://user-images.githubusercontent.com/53051438/163518590-0f270ac5-0998-4e0c-80af-6332b8fd5f90.png)
