# Django Web App Deployment on Google Cloud Platform (GCP)
Deploying a Django Web App on Google Cloud Platform (GCP)  

A step-by-step implementation of a Django web application hosted on a Google Cloud VM using Gunicorn, Nginx, and systemd. This repository also highlights common deployment challenges, such as managing external IP addresses and firewall rules, and how they were resolved.

 ### Overview

* Backend Framework: Django
* Cloud Provider: Google Cloud Platform (GCP)
* Web Server: Nginx (configured as a reverse proxy)
* App Server: Gunicorn (managed via systemd for service control)
* OS: Ubuntu 22.04 LTS
* Process Manager: systemd
* Custom firewall rules and static IP setup

### Step 1: Provisioned a Virtual Machine on Google Cloud
I created a Compute Engine VM instance on Google Cloud Platform using the following configuration:
```
 --zone=us-central1-a \
    --machine-type=e2-micro \
    --image-family=ubuntu-2204-lts \
    --image-project=ubuntu-os-cloud \
    --tags=http-server \
    --address=YOUR_STATIC_IP \
    --boot-disk-size=10GB
```
### Step 2: Installed Required Dependencies
After SSH-ing into the VM, I installed system dependencies to support version control, Python, and web server operations.
```
sudo apt update && sudo apt install -y git python3 python3-pip python3-venv nginx
```
Once Git was installed, I cloned this repository:
```
git clone https://github.com/ohizest/gcp-django-deploy-.git
cd gcp-django-deploy-/django_app
```
### Step 3: Set Up a Python Virtual Environment
To isolate project dependencies, I created a virtual environment within the project directory:
```
python3 -m venv appenv
source appenv/bin/activate 
```
All Python packages were installed within this virtual environment to avoid polluting the system Python installation.
### Step 4: Installed Django and Gunicorn
With the virtual environment activated, I installed the Django framework and Gunicorn — the WSGI server that will serve the application:
```
pip3 install django gunicorn
```
This setup ensures the app is ready to be served in a production-grade environment.
### Step 5: Created the Django Project Structure
Inside the django_app directory, I initialized a new Django project(django_testapp). I chose to place the project files directly in the current directory by appending a dot (.) to the command:
```
django-admin startproject django_testapp .
```
This created the standard Django project structure (with settings, URLs, and WSGI configuration) right at the root of my app folder. It simplifies deployment by aligning the project layout with the web server and service configs that follow.

### Step 6: Configure ```settings.py``` for Allowed Hosts
Django restricts incoming requests to a defined list of hosts for security reasons. To allow my Django app to respond to external requests from the GCP VM, I updated the ALLOWED_HOSTS list in the ```settings.py``` file.
First, I navigated into the Django project directory:
```
cd django_testapp
```
Then I opened the ```settings.py``` file using nano:
```
sudo nano settings.py
```
* I located the ALLOWED_HOSTS line and updated it to include: The external/static IP address of my VM Instance and 'localhost' for local development
```
ALLOWED HOSTS ['35.203.79.19', 'localhost']
```
This change ensures that Django accepts and serves requests coming from those specified hosts — which is essential when moving from local development to a production environment.
### Step 7: Created a Systemd Service for Gunicorn
To ensure that my Django application runs reliably and starts automatically on boot, I created a systemd service to manage Gunicorn, the WSGI server serving my app.

I created a new service file:
```
sudo nano /etc/systemd/system/gunicorn.service
```
Here’s the content of the service definition:
```
[Unit]
Description=gunicorn daemon for Django app
After=network.target

[Service]
User=ohiremen58
Group=www-data
WorkingDirectory=/home/ohiremen58/gcp-django-deploy-/django_app
Environment="PATH=/home/ohiremen58/gcp-django-deploy-/django_app/appenv/bin"
ExecStart=/home/ohiremen58/gcp-django-deploy-/django_app/appenv/bin/gunicorn \
    --access-logfile - \
    --workers 3 \
    --bind unix:/run/gunicorn.sock \
    django_testapp.wsgi:application
Restart=always

[Install]
WantedBy=multi-user.target

```
After saving the file, I enabled and started the service:
```
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable gunicorn
sudo systemctl start gunicorn
```
To confirm it was running successfully, I checked the status:
```
sudo systemctl status gunicorn
```
### Step 8: Set Up Nginx as a Reverse Proxy for Gunicorn
To expose the Django app to the web securely and efficiently, I configured Nginx to act as a reverse proxy. This allows Nginx to handle client requests and forward them to Gunicorn, which runs the Django application.

I created a new Nginx server block:
```
sudo nano /etc/nginx/sites-available/django_project
```
And added the following configuration:
```
server {
    listen 80;
    server_name 35.203.79.19;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/ohiremen58/gcp-django-deploy-/django_app;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }
}
```
* Replace YOUR_SERVER_IP_OR_DOMAIN with your actual VM external IP, in my case I used 35.203.79.19
Then I enabled the config:
```
sudo ln -s /etc/nginx/sites-available/django_project /etc/nginx/sites-enabled
```
I tested the Nginx config:
```
sudo nginx -t
```
The output was "syntax is ok" and "test is successful", then I reloaded Nginx:
```
sudo systemctl restart nginx
```
* Final Result
At this point, visiting http://35.203.79.19 in a browser loaded the Django app served through Gunicorn and proxied by Nginx
