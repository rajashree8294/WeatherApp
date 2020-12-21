# Jenkins

## Team Members:
1. Rajashree Joshi
1. Kinnar Kansara

## Run Playbooks

### JSON file for passing parameters

Name this file `extra_vars.json`
```
{
    "key_name":"jenkins_ec2_keypair",
    "ami":"ami-0817d428a6fb68645",
    "hosted_zone":"jenkins.example.me",
    "domain_name":"example.me",
    "sub_domain_name": "jenkins.example.me",
    "default_file_path":"/path/to/default",
    "email_id":"doe.j@northeastern.edu",
    "tag_value":"jenkins-instance",
    "aws_root": "root",
    "eip":"123.123.123.123"
}
```

### Build network and setup jenkins playbook

```
ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook jenkins_networking_playbook/jenkins_setup.yml --extra-vars "@extra_vars.json"
```

### Tear down network and jenkins playbook

```
ANSIBLE_STDOUT_CALLBACK=debug ansible-playbook teardown_playbook/teardown.yml --extra-vars "@extra_vars.json"
```


## Steps to setup Jenkins on EC2 instance with Nginx, Certbot and Let's encrypt

After launching EC2 instance with Ubuntu 18.04, create A record with public ip address of the instance in DNS of desired domain.

### JDK Setup 
SSH into the instance and execute following commands:

    sudo apt install default-jre

This will install OpenJRE 11. Validate the installation with command: `java -version`

Now install OpenJDK with below command

    sudo apt install default-jdk

Check installation with `javac -version` command

### Jenkins setup

    wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -

    sudo apt-add-repository "deb http://pkg.jenkins-ci.org/debian binary/"

    sudo apt install jenkins -y

### Install Certbot with Nginx

    sudo add-apt-repository ppa:certbot/certbot

    sudo apt install -y nginx python-certbot-nginx

### Configure Nginx

    sudo nano /etc/nginx/sites-available/default
With below content:
```
upstream jenkins {
  server 127.0.0.1:8080 fail_timeout=0;
}

server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;
        index index.html index.htm index.nginx-debian.html;

        server_name jenkins.example.me;

        location / {
                proxy_set_header        Host $host:$server_port;
                proxy_set_header        X-Real-IP $remote_addr;
                proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header        X-Forwarded-Proto $scheme; 
                proxy_set_header        Upgrade $http_upgrade;
                proxy_set_header        Connection "upgrade";
                proxy_pass              http://jenkins;
        }
}
```

### nginx config check

    sudo nginx -t

### restart nginx

    sudo systemctl restart nginx

### configure certbot to install certificate for your domain

    sudo certbot --nginx -d jenkins.example.me