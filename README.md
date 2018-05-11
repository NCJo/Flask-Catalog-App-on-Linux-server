# Flask-Catalog-App-on-Linux-Server

#### This fullstack web application was developed using:
* Flask
* Javascript
* Python2
* Bootstrap
* HTML/CSS


#### How to run
1. clone or download "private key"
2. public IP address: 18.236.184.190
3. SSH post: 2200
4. Click [this link](http://18.236.184.190/) to get to the main page!

#### Note to Reviewer
Download private key from the repository and type the following in terminal:
```
ssh -i <path_to_key> grader@18.236.184.190 -p 2200
```

#### The configuration summary
1. Change port from 22 to 2200
2. Configuring Firewall to allow SSH, http, and ntp
3. Change local timezone to UTC
4. Install Apache
5. Install PostgreSQL
6. Configure database
7. add Grader user and grant the user sudo permission
8. Installed Python2 and all its dependencies

#### Useful resources!
1. [How to figure SSH key-based authentication on a linux server](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)
2. [How to deploy a flask application on an Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
