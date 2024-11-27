Here's a guide to set up your personal study project using **Django** and various **Google Cloud Platform (GCP)** services:

## Technologies:
- **Django**: Web framework.
- **GCP**:
  - **Dataflow**: For data processing pipelines.
  - **BigQuery**: For data analytics.
  - **Cloud Run**: For running serverless containers.
  - **Cloud SQL**: Managed database service.
  - **Cloud Composer**: Managed Apache Airflow for workflow orchestration.

### Project Setup Steps

#### 1. Set Up a Virtual Environment

Set up a virtual environment to manage your project dependencies:
```bash
python -m venv venv
source venv/bin/activate
```

#### 2. Install Django and Create a Project

Install Django and create a new project:
```bash
pip install django
```
Check that Django was installed correctly:
```bash
django-admin --version
```
Create a new Django project called `myproject`:
```bash
django-admin startproject myproject
```
Run the development server to verify it works:
```bash
python manage.py runserver
```
To create an app inside your Django project:
```bash
python manage.py startapp myapp
```
Run the server again (on a custom port):
```bash
python manage.py runserver 8080
```

#### 3. Install Requirements

Create a `requirements.txt` file to list your project dependencies:
```txt
# Django web framework
Django>=4.2

# Apache Beam for Google Cloud Dataflow
apache-beam[gcp]>=2.48.0  # Compatible with Python 3.8

# Google Cloud BigQuery client library
google-cloud-bigquery>=3.10.0  # Compatible with Python 3.8

# Google Cloud Storage client library
google-cloud-storage>=2.5.0  # Compatible with Python 3.8

# Google Cloud Secret Manager client library
google-cloud-secret-manager>=2.16.0  # Compatible with Python 3.8

# Google Auth library for authentication with GCP
google-auth>=2.24.0

# PostgreSQL adapter for Python, used for Cloud SQL
psycopg2-binary>=2.9.7

# Django integration for cloud storage (Google Cloud Storage in this case)
django-storages>=1.13.0

# Python Decouple for managing environment variables
python-decouple>=3.8

# Apache Airflow (Managed by Cloud Composer)
apache-airflow>=2.6.0  # Compatible with Python 3.8

# Django REST Framework
djangorestframework>=3.14.0

# Django-environ for environment variables management
django-environ>=0.10.0

# Requests library for making HTTP requests
requests>=2.31.0
```

Install the dependencies:
```bash
pip install -r requirements.txt
```

Add the following dependencies to your `requirements.txt`:
```txt
Django>=4.2
apache-beam[gcp]>=2.49.0
google-cloud-bigquery>=3.11.4
google-cloud-storage>=2.10.0
google-cloud-sqlcommenter>=2.1.0
google-cloud-secret-manager>=2.16.2
google-auth>=2.24.0
psycopg2-binary>=2.9.7
django-storages>=1.13.2
python-decouple>=3.8
apache-airflow>=2.6.3
djangorestframework>=3.14.0
django-environ>=0.10.0
requests>=2.31.0
```

#### 4. Set Up for Production

**Install Required Packages**
- Install **Gunicorn** for running Django in production:
  ```bash
  pip install gunicorn
  ```
- Install **Nginx** for reverse proxying:
  ```bash
  sudo apt update
  sudo apt install nginx
  ```

**Serve Your Application with Gunicorn**
- Run Gunicorn to serve the application:
  ```bash
  gunicorn myproject.wsgi:application
  ```
- Or run on a custom address/port:
  ```bash
  gunicorn myproject.wsgi:application --bind 0.0.0.0:8000
  ```

#### 5. Set Up Nginx as a Reverse Proxy

**Edit Nginx Configuration**
- Create a new Nginx configuration for your project:
  ```bash
  sudo nano /etc/nginx/sites-available/myproject
  ```
- Add the following configuration:
  ```nginx
  server {
      listen 80;
      server_name your_domain.com www.your_domain.com; # Replace with your domain or server IP

      location = /favicon.ico { access_log off; log_not_found off; }
      location /static/ {
          root /path/to/your/static/files;  # Update this path
      }

      location / {
          proxy_pass http://127.0.0.1:8000;  # Forward requests to Gunicorn
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Proto $scheme;
      }
  }
  ```

**Enable Nginx Configuration**
- Create a symbolic link to enable the site:
  ```bash
  sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
  ```
- Test the configuration and reload Nginx:
  ```bash
  sudo nginx -t
  sudo systemctl reload nginx
  ```

#### 6. Configure Django Settings for Production
- Set `DEBUG = False` in `settings.py`.
- Set `ALLOWED_HOSTS` to your domain name:
  ```python
  ALLOWED_HOSTS = ['your_domain.com', 'www.your_domain.com']
  ```
- Configure **security settings** for production:
  ```python
  SECURE_SSL_REDIRECT = True
  SESSION_COOKIE_SECURE = True
  CSRF_COOKIE_SECURE = True
  ```

#### 7. Running Gunicorn as a Daemon

Create a **systemd** service file for Gunicorn:
```bash
sudo nano /etc/systemd/system/gunicorn.service
```
Add the following configuration:
```ini
[Unit]
Description=gunicorn daemon for myproject
After=network.target

[Service]
User=your_user
Group=www-data
WorkingDirectory=/path/to/your/project
ExecStart=/path/to/your/venv/bin/gunicorn --workers 3 --bind unix:/path/to/your/project/myproject.sock myproject.wsgi:application

[Install]
WantedBy=multi-user.target
```
Start and enable the service:
```bash
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

#### 8. Summary
- **Gunicorn** serves the Django application.
- **Nginx** acts as a reverse proxy to handle client requests and serve static files.
- Ensure your `settings.py` is configured properly for production.
- Run `collectstatic` to gather static files for Nginx.
- Secure your server using **HTTPS** and proper security configurations.
- Integrate with **GCP** services like **Dataflow**, **BigQuery**, **Cloud Run**, **Cloud SQL**, and **Cloud Composer** as needed.

### Summary of Dependencies:
- **Django** is the core web framework used for the project.
- **Apache Beam** is required to write data processing pipelines, which are executed by **Google Cloud Dataflow**.
- **BigQuery** and **Cloud Storage** client libraries allow integration with GCP services for data analytics and storage.
- **Cloud SQL** is accessed using **psycopg2-binary** for PostgreSQL, which is commonly used in production.
- **Airflow** is included to support workflow automation through **Cloud Composer**.
- **Django-Storages** integrates with **Google Cloud Storage** for handling file uploads in the app.
- **Requests** and **REST Framework** make it easier to create APIs and interact with external services.
- Integrate with **GCP** services like **Dataflow**, **BigQuery**, **Cloud Run**, **Cloud SQL**, and **Cloud Composer** as needed.

