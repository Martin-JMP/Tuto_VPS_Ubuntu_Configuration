# VPS Ubuntu version 24.04 Configuration Guide

This guide outlines the steps required to configure a VPS Ubuntu 24.04 (Virtual Private Server) from the initial setup to complet security adjustments.

Instruction to read the guide :
- when you see word or text in this <> it's an example you need in real to delete the <> and add your real value.
- when you see word or text after µ it's more tricky instruction.

## 1. Initial Setup

### Update and upgrade the VPS to ensure the latest packages and security updates are installed :

```bash
sudo apt update
```
```bash
sudo apt upgrade
```

### Integrate inital user :

```bash
passwd
```
```bash
sudo adduser <username>
```
```bash
sudo usermod -aG sudo <username>
```
```bash
sudo -l -U <username>
```
```bash
su - <username>
```

### First Security check, change number of port :
```bash
sudo systemctl edit ssh.socket
```

#### in this file you need to add or change this value :
```bash
[Socket]
ListenStream =
ListenStream = <numberport>
```

##### execute this instruction with the touch countrol on your laptop keyboard and after Enter :
```bash
ctrl+O µthe letter O
enter
```
```bash
ctrl+X
```


```bash
sudo systemctl daemon-reload
```
```bash
sudo systemctl restart ssh
```

#### Check the tcp autorization for your new number port :
```bash
sudo afw allow <numberport>/tcp
```

## 2. Implementation of website by Nginx :

### Install Nginx :
```bash
sudo apt install nginx -y
```

### Start Nginx :
```bash
sudo systemctl start nginx
```
```bash
sudo systemctl enable nginx
```
```bash
sudo systemctl status nginx
```

### Configuration of the tcp port :
```bash
sudo ufw allow 80/tcp
```
```bash
sudo ufw allow 443/tcp
```
```bash
sudo ufw anable
```
```bash
sudo ufw status numbered
```

## 3. Configure the domain name by Nginx :

```bash
sudo mkdir /var/www/<domainname> µput the domain name without the terminaison like .com or .fr
```
```bash
sudo nano /etc/nginx/sites-available/<domainname>.conf
```

### Put this information in the file <domainname>.conf :
```bash
# Create new
server {
  listen  80;
  server_name  www.<domainname>.<terminaison like .com or .fr> <domainname>.<terminaison>;
  location / {
    root /var/www/<domainname>;
    index index.html index.htm;
  }
}
```

### Create the ressource :
```bash
cd /etc/nginx/sites-enabled
```
```bash
sudo ln -s /etc/nginx/sites-available/<domainname>.conf
```
```bash
sudo systemctl reload nginx
```
```bash
cd
```
```bash
sudo nano /var/www/<domainname>/index.html
```
```bash
cd /etc/nginx/sites-enabled
```
```bash
sudo systemctl reload nginx
```

## 4. Extra permission SSH for transfert local file to the serveur :
```bash
sudo chown <username>/var/www/<domainname>/
```
```bash
ls -l
```
```bash
sudo chmod 777 /var/www/<domainname>/
```
```bash
ls -l
```

## 5. Certificat SSL for the domain name
