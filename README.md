# VPS Ubuntu version 24.04 Configuration Guide

This guide outlines the steps required to configure a VPS Ubuntu 24.04 (Virtual Private Server) from the initial setup to complet security adjustments.

Instruction to read the guide :
- when you see word or text in this <> it's an example you need in real to delete the <> and add your real value.
- when you see word or text after µ it's more tricky instruction.

More Informations :

https://www.server-world.info/en/note?os=Ubuntu_24.04&p=download

https://docs.vultr.com/how-to-install-nginx-mysql-php-lemp-stack-on-ubuntu-24-04

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
sudo nginx -t
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

## 5. Certificat SSL for the domain name :

### Installation of Certbot :
```bash
sudo apt -y install certbot
```
```bash
sudo certbot certonly --webroot -w /var/www/<domainname> -d <domainname>.<terminaison> -d www.<domainname>.<terminaison>
```

### Modification information in the file <domaine>.conf for nginx :
```bash
sudo nano /etc/nginx/sites-available/<domainname>.conf
```
#### In this file you need to modify partly information with your domain name or like that :
```bash
# create new
server {
    listen       80;
    listen       [::]:80;
    server_name  www.<domainname>.<terminaison like .com or .fr> <domainname>.<terminaison>;

    # Rediriger tout le trafic HTTP vers HTTPS
    return 301 https://$host$request_uri;
}

server {
       listen       443 ssl http2 default_server;
       listen       [::]:443 ssl http2 default_server; #default_server for the primary website
       server_name  www.<domainname>.<terminaison> <domainname>.<terminaison>;
       root         /var/www/domainname>;
       index index.html index.htm index.nginx-debian.html;

       ssl_certificate "/etc/letsencrypt/live/<domainname>.com/fullchain.pem";
       ssl_certificate_key "/etc/letsencrypt/live/<domainname>.com/privkey.pem";
       ssl_session_cache shared:SSL:1m;
       ssl_session_timeout  10m;

       location / {
              try_files $uri $uri/ =404;
       }

       location ~ \.php$ {
              include snippets/fastcgi-php.conf;
              fastcgi_pass unix:/run/php/php8.3-fpm.sock;
       }

}
```

### To add the HTTPS you nedd to reload nginx for apply the modifications :
```bash
sudo systemctl reload nginx
```

### If your https is out verify the certificates and reload the serveur :
```bash
sudo certbot certificates
```

```bash
sudo systemctl reload nginx
```

## 6. Installation of PHP 8.3 :

```bash
sudo apt -y install php8.3 php8.3-mbstring php.pear
```
```bash
php -v
```

### Verfication of the installation :
```bash
sudo chown <username> /var/www/html/
```
```bash
ls -l
```
```bash
sudo chmod 777 /var/www/html/
```
```bash
ls -l
```
```bash
cd /var/www/html/
```
```bash
sudo echo '<?php echo 'php -i'."\n" ?>' > php_test.php     µIn this instruction you nedd to add the quote <>
```
```bash
ls
```
```bash
php php_test.php | head
```
```bash
cd
```

### Second verification is not optionnal :
```bash
sudo apt -y install php8.3-fpm
```
```bash
cd /etc/nginx/sites-available/
```
```bash
sudo nano <domainname>.conf
```
```bash
cd
```
```bash
sudo systemctl reload php8.3-fpm nginx
```
```bash
sudo echo '<?php phpinfo(); ?>' > /var/www/<domainname>/info.php
```

## 7. Installation of mySQL :

```bash
sudo apt -y install mysql-server-8.0
```
```bash
sudo mysql
```
```bash
select user,host,plugin from mysql.user;
```
```bash
exit
```

### Security installation of mySQL :
```bash
sudo mysql_secure_installation
```

#### Just accept that exact procedure :
```bash
y
1
y
y
y
```

### Acces to mySQL :
```bash
sudo mysql
```

### Create database :
```bash
create database <databasename>
```

### Create user :
```bash
create User '<username>'@'localhost' Identified by '<Password>';   µput a new password to have the acces of the database
```

### Add admin privilege of the user to acces the database :
```bash
Grant all privileges on <domainname>.* to '<databasename>'@'localhost';
```

### Acces to the database :
```bash
use <databasename>;
```

### Part 2 add admin privilege of the user to acces the database :
```bash
flush privileges;
```

## 8. Create the link betwwen github repository and VPS with devops CI/CD :

### Go to the repository GitHub and add the actions secrets :

```bash
HOST = IP of the server
USERNAME = the name of the user with all the authorizations on the server
PORT = The name of the port used to connect on the server
```

### Install the ssh client and server :
```bash
sudo apt install openssh-client openssh-server
```
```bash
mkdir ~/.ssh/
```
```bash
cd .ssh/
```
```bash
ls
```

### Create the public key and the private key :
```bash
ssh-keygen -t rsa -b 4096 -C "<personalemail>"
```
Enter
```bash
<repositoryname>
```
Enter
Enter

```bash
ls
```
### Save the Public key :
```bash
cat <repositoryname>.pub
```
Go to the GitHub account -> Settings -> SSH and GPG keys -> New SSH Key 
```bash
Title = Write a name with the name of the server and the repository name (.e.g: Ubuntu Website)
```
```bash
Key Type = Authentification Key
```
```bash
Key = copy and past the result of the "cat <repositoryname>.pub"
```

### Save the Private key :
```bash
cat <repositoryname>
```
Go to the repository project GitHub and add the action secret 
```bash
SSH_PRIVATE_KEY = put the private key like copy past the result of "cat <repositoryname>"
```

### Some verification :
```bash
cat <repositoryname>.pub >> authorized_keys
```
```bash
verfication :
```
```bash
cat authorized_keys
```
```bash
eval "$(ssh-agent -s)"
```
```bash
ssh-add ~/.ssh/<repositoryname>
```
```bash
ssh-add -l
```
```bash
cd
```
```bash
ssh -T git@github.com
```
yes

Now you can see this message :
"Hi usergithubname! You've successfully authenticated, but GitHub does not provide shell access."


### Clone the GitHub repository on the VPS :
#### Go to the WWW folder :
```bash
cd
```
```bash
cd /var/www
```
```bash
ls
```
If you see already the name of the repository name, please delete the folder before the cloning : 
```bash
sudo rm -r <repositoryname>
```
```bash
ls
```
```bash
cd
```
```bash
sudo chown -R <username>: /var/www
```
```bash
cd /var/www/
```

Now you can clone the repository :
```bash
sudo git clone git@github.com:<usergithubname>/<repositoryname>.git
```
yes

Now you can see this message :
"Hi usergithubname! You've successfully authenticated, but GitHub does not provide shell access."

### Verification if the clone has been well uploaded 
```bash
cd /var/www/<repositoryname>
```
```bash
ls
```

### Create the Action for the repository :
Go to GitHub -> <repositoryname> -> Actions -> New Workflow -> set up a workflow yourself

#### Implement this code in the main.yml :
```yml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:       
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Set up SSH
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          port: ${{ secrets.PORT }}
          script: |
            # Print current SSH keys
            echo "Checking SSH keys..."
            ls -l ~/.ssh
            
            # Check if the SSH key exists
            if [ -f ~/.ssh/<repositoryname> ]; then
              eval "$(ssh-agent -s)"
              ssh-add ~/.ssh/<repositoryname>
      
              # Verify if the key was added
              echo "Loaded SSH keys:"
              ssh-add -l
            else
              echo "SSH key not found!"
              exit 1
            fi
            
            # Test SSH connection with verbose output
            ssh -vT git@github.com
      
            # Navigate to the project directory
            cd /var/www/<repositoryname>/ || exit 1
            
            # Pull the latest code
            echo "Pulling latest code from repository..."
            git pull origin main || exit 1
            
            # Show the current branch and status
            echo "Current branch and status:"
            git branch
            git status
```


### Install the package you needed to improve you project :
```bash
sudo apt update
```
```bash
sudo apt upgrade
```

### Change some user authorization :
```bash
cd
```
```bash
sudo chown <username> /var/www/<repositoryname>/
```
```bash
ls -l
```
```bash
cd /var/www/
```
```bash
ls -l
```
```bash
sudo chmod 777 /var/www/<repositoryname>/
```
```bash
ls -l
```

## 9. Creation of proxy stystem :

```bash
sudo ufw status
```
```bash
sudo ufw allow <portnumber>   µUse a different port number than the security port to acces of the vps
```
```bash
sudo ufw status
```
```bash
cd <your path with the file proxy.js create for your project with your parameters>
```
#### Put in this file lot of parameters and informations like HTTPS, SSL certificat, API link, Port number, localisation of file :
```bash
sudo nano proxy.js
```
#### Modify the links between the web site page and the server for exemple 'https://<domainname>.<terminaison>:<portnumber>/' :
```bash
sudo nano Main.html
```
#### Modify the file <domainname>.conf with the information of the proxy :
```bash
sudo nano /etc/nginx/sites-available/<domainname>.conf
```
```bash
server {
  listen  80;
  listen [::]:80;
  server_name www.<domainname>.<terminaison> <domainname>.<terminaison>
  # Redirected the trafic HTTP by HTTPS
  return 301 https://$host$request_uri;
}

# Part for reverse proxy for Node.js application
Location /api {
  proxy_pass http://127.0.0.1:<portnumber>;
  proxy_http_version 1.1;
  proxy_set_header Upgrade $http_Upgrade;
  proxy_set_header Connection 'upgrade';
  proxy_set_header Host $host;
  proxy_cache_bypass $http_upgrade;
}
}
```

## 9. Run any file, code in background of your server :

```bash
cd <your path with the file proxy.js>
```
```bash
nohup node proxy.js &
```
### You can check if your file or code run perfectly :
```bash
ps aux | grep node
```

### If you want to stop the process in background :
```bash
ps aux | grep node
```
```bash
kill -9 µOk change the PID by the number or the code who you can find with the instruction above, for exemple kill -9 <2323>
```


### If your proxy serveur is out verify the certificates in your file, kill the nohup and reload the proxy serveur :
```bash
const privateKey = fs.readFileSync('/etc/letsencrypt/archive/<domainname>.<terminaison>/privkey<changenumber>.pem', 'utf8');
const certificate = fs.readFileSync('/etc/letsencrypt/archive/<domainname>.<terminaison>/fullchain<changenumber>.pem', 'utf8');
```



Cooked by Martin JONCOURT

      JJJ  EEEEE   AAA    NNNN 
       JJ  E      A   A  NN  NN
       JJ  EEEE   AAAAA  NN  NN
    J  JJ  E      A   A  NN  NN
     JJJ   EEEEE  A   A  NN  NN

    MM   MM   AAA   RRRR   TTTTTT  IIII   NNNN
    MMM MMM  A   A  RR  R    TT     II   NN  NN
    MM M MM  AAAAA  RRRR     TT     II   NN  NN
    MM   MM  A   A  RR R     TT     II   NN  NN
    MM   MM  A   A  RR  R    TT    IIII  NN  NN

    PPPPP   IIII  EEEEE  RRRR   RRRR   EEEEE
    PP   PP  II   E      RR  R  RR  R  E
    PPPPP    II   EEEE   RRRR   RRRR   EEEE
    PP       II   E      RR R   RR R   E
    PP      IIII  EEEEE  RR  R  RR  R  EEEEE
