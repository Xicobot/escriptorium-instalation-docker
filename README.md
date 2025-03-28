# eScriptorium Docker Installation and Configuration Guide

This document provides comprehensive instructions for installing, configuring, and maintaining eScriptorium using Docker.

---
# Introduction to eScriptorium

eScriptorium is an open-source platform designed for the transcription, annotation, and analysis of historical manuscripts and documents. It combines powerful OCR (Optical Character Recognition) capabilities with collaborative editing features, making it an essential tool for digital humanities projects, archives, libraries, and scholarly research.

## What is eScriptorium?

eScriptorium provides a web-based environment where users can:

- Upload digital images of historical documents
- Apply advanced OCR using the Kraken engine, which is specially optimized for historical texts
- Manually correct and annotate transcriptions
- Train custom OCR models for specific document types or scripts
- Collaborate with team members on transcription projects
- Export results in various formats for further analysis

## Key Features

- **Specialized OCR**: Built on Kraken, which excels at recognizing historical scripts and layouts
- **Training capabilities**: Create custom OCR models for your specific manuscript collections
- **Annotation tools**: Mark up text with semantic tags and paleographic features
- **Multi-user support**: Collaborate with team members on transcription projects
- **Version control**: Track changes and maintain the history of transcriptions
- **Flexible export**: Export your data in various formats (TEI, ALTO, plain text, etc.)

## Why Use Docker?

Deploying eScriptorium with Docker provides several advantages:

1. **Simplified installation**: Avoid complex dependency management
2. **Consistent environment**: Ensure the application works the same way across different systems
3. **Isolation**: Keep the application and its dependencies separate from other system software
4. **Scalability**: Easily scale specific components as needed
5. **Easy updates**: Streamline the update process while preserving your data

The Docker-based deployment allows you to get eScriptorium up and running quickly, even without extensive system administration experience.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Installation Steps](#installation-steps)
- [Configuration](#configuration)
  - [Environment Variables](#environment-variables)
  - [Docker Compose Files](#docker-compose-files)
- [Architecture Overview](#architecture-overview)
- [Usage Guide](#usage-guide)
- [Maintenance](#maintenance)
- [Troubleshooting](#troubleshooting)
- [Advanced Configuration](#advanced-configuration)
- [Sample Configuration Files](#sample-configuration-files)

## Prerequisites

Before installing eScriptorium, ensure you have:

- A Linux server (Ubuntu 20.04+ or Debian 11+ recommended)
- Docker Engine (version 20.10+)
- Docker Compose (version 2.0+)
- Git
- At least 4GB RAM
- At least 20GB disk space
- Ports 8000 available (or configured for your needs)

Install Docker and Docker Compose if not already installed:

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/download/v2.18.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

## Installation Steps

### 1. Clone the Repository

```bash
git clone https://gitlab.com/scripta/escriptorium.git
cd escriptorium
```

### 2. Configure Environment Variables

Create and edit the environment file:

```bash
cp variables.env_example variables.env  
```

### 3. Build and Start the Containers

For production:

```bash
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

For development:

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up -d
```

### 4. Create Superuser (if not using environment variables)

If you didn't configure superuser credentials in the .env file:

```bash
docker-compose exec app python manage.py createsuperuser
```

### 5. Access eScriptorium

Once the installation is complete, access eScriptorium at:

- http://localhost:8000 
- or http://your-domain.com (if configured)

## Configuration

### Environment Variables

Key environment variables to configure in your `.env` file:

| Variable | Description | Example |
|----------|-------------|---------|
| POSTGRES_PASSWORD | Database password | `securepassword` |
| SECRET_KEY | Django secret key | `a-very-long-random-string` |
| DEBUG | Debug mode (False for production) | `False` |
| ALLOWED_HOSTS | List of allowed hosts | `localhost,example.com,127.0.0.1` |
| DEFAULT_FROM_EMAIL | From email for notifications | `no-reply@example.com` |
| EMAIL_HOST | SMTP server | `smtp.gmail.com` |
| EMAIL_HOST_USER | SMTP username | `your-email@gmail.com` |
| EMAIL_HOST_PASSWORD | SMTP password | `your-app-password` |
| TIME_ZONE | Your timezone | `Europe/Paris` |
| DJANGO_SUPERUSER_* | Admin credentials | See example file |

### Docker Compose Files

eScriptorium uses several Docker Compose files for different environments:

- `docker-compose.yml`: Base configuration
- `docker-compose.dev.yml`: Development overrides
- `docker-compose.prod.yml`: Production overrides

Combine these files as needed with the `-f` flag.

## Architecture Overview

The eScriptorium Docker setup consists of the following services:

- **app**: Main Django application
- **db**: PostgreSQL database for data storage
- **redis**: Redis for caching and message broker
- **celery**: Celery worker for asynchronous task processing
- **celery-beat**: Celery beat for scheduled tasks
- **kraken**: Kraken OCR engine container

Data persistence is handled through Docker volumes:
- `postgres-data`: Database files
- `media`: User uploaded files and generated documents
- `static`: Static web assets

## Usage Guide

### Initial Setup

After installation:

1. Log in with your admin credentials
2. Create a project
3. Upload documents
4. Train models or use pre-trained ones
5. Process documents with OCR

### Backup and Restore

To backup your eScriptorium instance:

```bash
# Backup database
docker-compose exec db pg_dump -U postgres postgres > escriptorium_backup.sql

# Backup media files
tar -czvf media_backup.tar.gz ./media
```

To restore:

```bash
# Restore database
cat escriptorium_backup.sql | docker-compose exec -T db psql -U postgres postgres

# Restore media files
tar -xzvf media_backup.tar.gz
```

## Maintenance

### Updating eScriptorium

To update to the latest version:

```bash
# Pull latest changes
git pull

# Rebuild and restart containers
docker-compose -f docker-compose.yml -f docker-compose.prod.yml down
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d --build
```

### Database Migrations

After updates, you may need to run migrations:

```bash
docker-compose exec app python manage.py migrate
```

### Collecting Static Files

If static files have changed:

```bash
docker-compose exec app python manage.py collectstatic --noinput
```

## Troubleshooting

### Viewing Logs

```bash
# View logs for all services
docker-compose logs

# View logs for a specific service
docker-compose logs app
docker-compose logs kraken
```

### Common Issues

1. **Database connection errors**
   - Check PostgreSQL is running: `docker-compose ps db`
   - Verify database credentials in .env file

2. **Permission issues with mounted volumes**
   - Fix permissions: 
     ```bash
     sudo chown -R 1000:1000 ./media
     sudo chown -R 1000:1000 ./static
     ```

3. **Kraken OCR not working**
   - Check Kraken container logs: `docker-compose logs kraken`
   - Verify connections between app and kraken services

4. **Application errors**
   - Check app logs: `docker-compose logs app`
   - Enable DEBUG=True temporarily for detailed error pages

## Advanced Configuration

### Using a Reverse Proxy

For production, use Nginx as a reverse proxy:

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /static/ {
        alias /path/to/escriptorium/static/;
    }

    location /media/ {
        alias /path/to/escriptorium/media/;
    }
}
```

### SSL/TLS Configuration

With Let's Encrypt:

```bash
# Install certbot
sudo apt-get install certbot python3-certbot-nginx

# Get certificate
sudo certbot --nginx -d your-domain.com

# Automatic renewal
sudo certbot renew --dry-run
```

### Scaling Workers

For larger deployments, you can scale the Celery workers:

```bash
docker-compose up -d --scale celery=3
```

## Sample Configuration Files

### Docker Compose File

```yaml
version: '3'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    depends_on:
      - db
      - redis
    volumes:
      - ./media:/usr/src/app/media
      - ./static:/usr/src/app/static
    environment:
      - DJANGO_SETTINGS_MODULE=escriptorium.settings
      - POSTGRES_HOST=db
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - SECRET_KEY=${SECRET_KEY}
      - DEBUG=${DEBUG:-False}
      - ALLOWED_HOSTS=${ALLOWED_HOSTS:-localhost,127.0.0.1}
      - DEFAULT_FROM_EMAIL=${DEFAULT_FROM_EMAIL}
      - EMAIL_HOST=${EMAIL_HOST}
      - EMAIL_HOST_USER=${EMAIL_HOST_USER}
      - EMAIL_HOST_PASSWORD=${EMAIL_HOST_PASSWORD}
      - TIME_ZONE=${TIME_ZONE:-UTC}
    ports:
      - "8000:8000"

  db:
    image: postgres:13
    restart: unless-stopped
    volumes:
      - postgres-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}

  redis:
    image: redis:6
    restart: unless-stopped

  celery:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    depends_on:
      - app
      - db
      - redis
    volumes:
      - ./media:/usr/src/app/media
    environment:
      - DJANGO_SETTINGS_MODULE=escriptorium.settings
      - POSTGRES_HOST=db
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - SECRET_KEY=${SECRET_KEY}
      - DEBUG=${DEBUG:-False}
    command: celery -A escriptorium worker -l INFO

  celery-beat:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped
    depends_on:
      - app
      - db
      - redis
    environment:
      - DJANGO_SETTINGS_MODULE=escriptorium.settings
      - POSTGRES_HOST=db
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - SECRET_KEY=${SECRET_KEY}
      - DEBUG=${DEBUG:-False}
    command: celery -A escriptorium beat -l INFO

  kraken:
    image: registry.gitlab.com/scripta/escriptorium/krakenworker:latest
    restart: unless-stopped
    depends_on:
      - app
      - redis
    volumes:
      - ./media:/usr/src/app/media
    environment:
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - KRAKEN_QUEUE=kraken

volumes:
  postgres-data:
  media:
  static:
```

### .env File

```
# Database configuration
POSTGRES_PASSWORD=ChangeThisToASecurePassword

# Django configuration
SECRET_KEY=ChangeThisToAVeryLongRandomString
DEBUG=False
ALLOWED_HOSTS=localhost,127.0.0.1

# Email configuration
DEFAULT_FROM_EMAIL=no-reply@example.com
EMAIL_HOST=smtp.example.com
EMAIL_HOST_USER=user
EMAIL_HOST_PASSWORD=password
EMAIL_PORT=587
EMAIL_USE_TLS=True

# Timezone
TIME_ZONE=UTC

# Superuser (optional - will be created on first run)
DJANGO_SUPERUSER_USERNAME=admin
DJANGO_SUPERUSER_PASSWORD=ChangeThisToASecurePassword
DJANGO_SUPERUSER_EMAIL=admin@example.com
```

## Additional Resources

- [Official eScriptorium Documentation](https://gitlab.com/scripta/escriptorium/-/wikis/home)
- [Kraken OCR Documentation](https://kraken.re/master/index.html)
- [Django Documentation](https://docs.djangoproject.com/)
- [Docker Documentation](https://docs.docker.com/)
