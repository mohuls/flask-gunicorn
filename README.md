# Gunicorn-Nginx
This repository and README file describe basic python and flask setup and deploy from local to cloud using Ubuntu 20.04 LTS, Nginx Web Server, Sqlite Database and Let's Encrypt SSL.

**Usage Note:** This documentation is for both local and remote machine (Local & Cloud flagged). If the local machine is already configured then just setup the cloud according to the below guidelines.

## Python and Virtual Environment Intro
In this repo described the fully optimized way to install Python using **pyenv** so that we can install and manage different versions of Python in the both local and remote machine. Then described the optimized way to install python virtual environment for isolating individual python application.

**Important:** Using python virtual environment is very useful for developing any kind of python applications (ex: this flask application). It is because we can use different version and dependencies for each appliccation running in the same machine. Moreover, as the virtual environment is installed right inside the project folder so that the envrionment for the application remains same for local and remote machine when it is subject to deploy from local to remote server. So that, the dependencies never conflict.

## Flask, Gunicorn & Nginx Intro
Flask is a microframework written in python for developing web applications and web API with minimal effort.

Gunicorn is a python application server that helps to run python application (in this case flask application) also can be run django application as well.

Nginx is a high end web application server for serving static and dynamic content. In this repo nginx is used with gunicorn for running our flask aplication.

## Ubuntu 20.04 Setup (Local & Cloud)
We will not use root user alwyas. That's why we will create a new user with the sudo privilege.

1. On the local machine just create the user.
2. On remote machine first connect the machine using ssh.
3. Use a non-root sudo user for all the below operations.
**Hints:** To connect via SSH, add your local machine's public key (~/.ssh/id_rsa.pub) to the cloud SSH portion and then access the remote via $ ssh root@your_server_ip or use a .pem private key file for corresponding added public key on the machine (ex: EC2) by $ ssh -i 'key.pem' user@server-ip.

## Creating user account
1. `$ sudo adduser lina` Then enter password and info to create user.
2. `$ usermod -aG sudo lina` to add lina to sudo
3. `$ sudo ufw app list` to see all the firewall list
4. `$ sudo ufw allow OpenSSH` and then `$ sudo ufw enable` to allow OpenSSH
5. `$ sudo ufw status` to check status.
6. `$ su - lina` switch to lina from root or any other user.

## Installing pyenv (Local & Remote)
Pyenv is a python package manager that can be used to manage multiple versions of python on a single OS. Let's install pyenv latest version.

1. `$ sudo apt update -y` to update the system.
2. `$ sudo apt install -y build-essential` install the build dependencies.
3. `$ sudo apt install zlib1g-dev libssl-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libffi-dev liblzma-dev python-openssl`
3. `$ git clone https://github.com/pyenv/pyenv.git ~/.pyenv` to clone the latest version of git.
4. `$ sudo echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc`
5. `$ sudo echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc`
6. `$ sudo echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n eval "$(pyenv init -)"\nfi' >> ~/.bashrc`
7. `$ exec "$SHELL"`
8. `$ pyenv --version`. Now pyenv is installed successfully.

**Learn more about pyenv:** https://realpython.com/intro-to-pyenv/

##  Installing Python (Local & Remote)
**Note:** The python version to be installed depend on the version that require the application. We can install multiple version of python and switch back to specific verion for our specific python version.

0. `$ pyenv install --list` to see all the available python version.
1. `$ pyenv install x.x.x` to install the needed version. We can install multiple versions. We can activate specific version when we need to create a virtual environment for a specific application.
2. `$ pyenv versions` to see installed/available version to activate.
3. `$ pyenv global x.x.x` to activate a specific python version locally.
4. `$ pyenv global system` switch back to system version.

## Creating Virtual Environments (Local & Remote)

**Important:** Before creating any virtual environment, switch to the desired python version using `$ pyenv global x.x.x`.

1. `$ pip install --user virtualenv` to install virtual environment.
2. `$ mkdir ~/flask-gunicorn`  `$ cd ~/flask-gunicorn` to project directory. Now to create a virtual env goto your project directory and type `$ python -m virtualenv env` it will create a virtual environment in the env folder.
3. `$ source env/bin/activate` to active the virtual env. Now you are isolated in the project directory with your own python version and packages.
4. To check the python and pip location type `$ which python` and `$ which pip`. Check if the installation location indicating the env folder or not. If indicates the env folder then the environment working fine (Environment activation is needed before checking).

**Note:** Make sure the app that was developed locally used python version is same as the virtual env.

## Develop Flask Application (Locally)
Now develop the flask application locally. Make sure the version of virtual environment of development environment is same as the remote environment we just created.

1. After development `$ pip freeze > requirements.txt` to store required packages.

## Pushing to GitHub (Local)

1. Upload the source file to a GitHub repo. Keep env folder in the .gitignore file. So that env folder will not be uploaded. Because it is created in the remote server by following above instructions.
2. If the application uses databases rather than sqlite3. Then backup the database and put in source file (better keep that db file in db folder).
3. `$ git init` then `$ git add .` the `$ git commit -m 'test deploy'`
4. `$ git remote add origin git@github.com:mohuls/flask-gunicorn.git` the GitHub repo we just created on GitHub.
5. `$ git push -u origin master` to push the full project to GitHub repo.

## Deploying to Live Server (Remote)

1. Clone the github repo `$ git clone https://github.com/mohuls/flask-gunicorn` in a directory at server.
2. Copy all the cloned files to project dir where you created the virtual environment `$ cp -a flask-gunicorn/. ~/flask-gunicorn`.
2. cd to the project dir.
3. `$ source env/bin/activate` to activate the virtual env (important).
4. `$ pip install -r requirements.txt` to install all the required package.
5. `$ nano app.py` and add `host='0.0.0.0'` to the app.run. The line will be like `app.run(debug=True, host='0.0.0.0')`.
6. `$ sudo ufw allow 5000` to allow the development port.
7. `$ python app.py` to run the development server.
8. Now Browse the server IP with port 5000 (http://server-ip:5000)to see if the app is running. (It should be running actually).

## Creating the WSGI Entry Point (Remote)

1. cd to project dir then `$ nano wsgi.py` to create a wsgi file. paste the following:
```python
from app import app

if __name__ == "__main__":
    app.run()
```
2. Here first `app` is the main flask entry python file that resides in the project's root folder. and the second `app` is the flask instance inside the app file.

## Configuring Gunicorn

1. `$ cd project-dir` activate virtual environment then `$ pip install gunicorn` to install gunicorn.
2. `$ gunicorn --bind 0.0.0.0:5000 wsgi:app` to start the app in gunicorn server. Again browse the server IP the app is running using gunicorn application server.
3. `$ deactivate` to deactivate the virtual environment.
4. `$ sudo nano /etc/systemd/system/app.service` here app is the gunicorn service name. Change according to your choice. Conventionally use the name of the app. Paste the following:

```conf
[Unit]
Description=Gunicorn instance to serve a flask app
After=network.target

[Service]
User=lina 
Group=www-data
WorkingDirectory=/home/lina/flask-gunicorn
Environment="PATH=/home/lina/flask-gunicorn/env/bin"
ExecStart=/home/lina/flask-gunicorn/env/bin/gunicorn --workers 3 --bind unix:flask-gunicorn.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target
```

**Note:** Here define the current username, path of virtual env bin and gunicorn service path and here `flask-gunicorn` is the app name.

5. `$ sudo systemctl start flask-gunicorn` to start the gunicorn server.
6. `$ sudo systemctl enable flask-gunicorn` to enable the gunicorn server on system startup.
7. `$ sudo systemctl status flask-gunicorn` to check the status.
8. `$ sudo systemctl stop flask-gunicorn` to stop the service.

## Configuring Nginx
1. `$ sudo apt install nginx` to install nginx server.
2. `$ sudo nano /etc/nginx/sites-available/flask-gunicorn` create a nginx server configaration.
```conf
server {
    listen 80;
    server_name ServerIP or domain name;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/lina/flask-gunicorn/flask-gunicorn.sock;
    }
}
``` 
3. `$ sudo ln -s /etc/nginx/sites-available/flask-gunicorn /etc/nginx/sites-enabled` to enable the conf file.
4. `$ sudo unlink /etc/nginx/sites-enabled/default` to disable the default site.

5. `$ sudo nginx -t` to check the configaration file.
6. `$ sudo systemctl restart nginx` to restart the server.
7. `$ sudo ufw delete allow 5000` to delete the development port.
8. `$ sudo ufw allow 'Nginx Full'` to allow nginx traffic.
9. Now browse the server IP again. Now it should serve the application using Nginx web server by piping to gunicorn server.

## Configuring Domain

1. `$ sudo nano /etc/nginx/sites-available/flask-gunicorn` edit the server block file. Replace the domain name you want to configure with the server IP.
2. Make sure you have added the IP address in the domain's DNS.
3. `$ sudo systemctl restart nginx` to restart the server.
4. Wait for propagrating the domain wordwide (can take upto 48 hours)
5. Now the app sould be accessed by the domain name.

## Configuring Let's Encrypt SSL
1. `$ sudo apt install python3-certbot-nginx` to install certbot.
2. `$ sudo ufw allow 'Nginx Full'` allow Nginx in http and htpps.
3 `$ sudo nano /etc/nginx/sites-available/flask-gunicorn` and add the host name as the domain name that is configured with the server ip:
server_name mohuls.com www.mohuls.com;
4. `$ sudo systemctl restart nginx` restarts nginx.
5. `$ sudo certbot --nginx -d mohuls.com -d www.mohuls.com` starts the installation process. Answer appropriate to the asked questions to complete.
6. `$ sudo systemctl status certbot.timer` to check status.
7. `$ sudo certbot renew --dry-run` tests renewal process.
Now the site should serve in https.

## Setting up multiple applications