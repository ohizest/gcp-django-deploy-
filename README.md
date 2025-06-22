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

### Step 1: Create a VM on Google Cloud Platform
Created a VM Instance on on Google Cloud Platform using the settings below:
```
 --zone=us-central1-a \
    --machine-type=e2-micro \
    --image-family=ubuntu-2204-lts \
    --image-project=ubuntu-os-cloud \
    --tags=http-server \
    --address=YOUR_STATIC_IP \
    --boot-disk-size=10GB
```
### Step 2: Install the Dependencies
After I successfully ssh into the repository, I installed the git dependency to enable me use git command.
```
sudo apt install git -y
```
Then I cloned my repository.
```
git clone https://github.com/ohizest/gcp-django-deploy-.git
```
Next we install the Python and Nginx dependencies
```
sudo apt install -y python3 python3-pip python3-venv
sudo apt install nginx -y
```
### Step 3: Create a Virtual Environment and Activate it
The below command creates a working environmnent (appenv) in the current directory
```
python3 -m venv appenv 
```
Since we are on Linux Ubuntu, we use this command to activate the virtual environment.
```
source appenv/bin/activate
```
### Step 4: Install Django and Gunicorn in the Virtual Environment
```
pip3 install django
pip3 install gunicorn
```
### Step 5: Create a Django Project folder that will contain the Django files
```
django-admin startproject project_name .
```
project_name: Replace this with the actual name you want for your Django project. In my case, it's "django_testapp"
.: The dot at the end tells Django to create the project files in the current directory.

### Step 6: Configure settings.py for Allowed Hosts
Django restricts incoming requests to a defined list of hosts for security reasons. To allow requests from your Google Cloud VM and local development environment, you need to update the ALLOWED_HOSTS setting in the Django project's settings.py file.
* Navigate to the Django project directory:
```
cd django_testapp
```
* Open the settings.py file in a text editor:
```
sudo nano settings.py
```
* I located the ALLOWED_HOSTS line and updated it to include: The external/static IP address of my VM Instance
```
ALLOWED HOSTS ['35.203.79.19', 'localhost']
```
