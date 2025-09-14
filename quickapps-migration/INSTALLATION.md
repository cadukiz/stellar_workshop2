# QuickAppsCMS Docker Installation Process

This document details the complete installation process for QuickAppsCMS with Docker containers.

## Installation Overview

**Date**: September 14, 2025  
**QuickAppsCMS Version**: 2.0 (dev-master)  
**CakePHP Version**: 3.3.16  
**PHP Version**: 7.4  
**Platform**: macOS (Darwin, ARM64/Apple Silicon)

## Step-by-Step Installation Process

### 1. Project Structure Creation

Created the following directory structure:
```
quickapps-migration/
├── docker/
│   ├── Dockerfile
│   └── apache-config.conf
├── docker-compose.yml
├── src/                  # QuickAppsCMS installation directory
├── .env                  # Environment variables
└── README.md            # User documentation
```

**Command executed:**
```bash
mkdir -p quickapps-migration/docker quickapps-migration/src
```

### 2. Dockerfile Configuration

Created `docker/Dockerfile` with PHP 7.4 and all required extensions:

**Key components:**
- Base image: `php:7.4-apache`
- PHP extensions installed:
  - pdo, pdo_mysql (database connectivity)
  - mbstring (string handling)
  - intl (internationalization)
  - fileinfo (file operations)
  - gd (image processing)
  - zip (archive handling)
  - opcache (performance)
- Apache mod_rewrite enabled
- Composer 2 installed
- Custom PHP memory and upload limits configured

### 3. Apache Configuration

Created `docker/apache-config.conf` with:
- DocumentRoot set to `/var/www/html/webroot` (CakePHP standard)
- AllowOverride All for .htaccess support
- Rewrite rules for CakePHP routing
- Error and access logging configured

### 4. Docker Compose Setup

Created `docker-compose.yml` with three services:

#### Web Service (PHP/Apache)
- Container name: `quickapps-web`
- Port: 8080 → 80
- Volume mounts for source code and Apache config
- Depends on database service

#### Database Service (MySQL)
- Container name: `quickapps-db`
- Image: `mysql:5.7`
- **Platform specification added**: `platform: linux/amd64` (for Apple Silicon compatibility)
- Port: 3306 → 3306
- Persistent volume for data
- Database credentials configured via environment variables

#### phpMyAdmin Service
- Container name: `quickapps-phpmyadmin`
- Port: 8081 → 80
- Connected to database service
- Auto-configured with database credentials

### 5. Environment Configuration

Created `.env` file with:
```
MYSQL_ROOT_PASSWORD=rootpassword
MYSQL_DATABASE=quickapps
MYSQL_USER=quickapps
MYSQL_PASSWORD=quickapps123
APP_PORT=8080
PHPMYADMIN_PORT=8081
DB_PORT=3306
```

### 6. Container Build and Launch

**Build command:**
```bash
docker-compose build
```

**Issues encountered:**
- MySQL 5.7 doesn't have native ARM64 support
- **Solution**: Added `platform: linux/amd64` to force x86_64 emulation

**Start containers:**
```bash
docker-compose up -d
```

### 7. QuickAppsCMS Installation via Composer

**Initial attempt failed due to Composer plugin restrictions:**
```bash
docker exec quickapps-web bash -c "cd /var/www/html && composer create-project -s dev quickapps/website . --no-interaction"
```

Error: `aura/installer-default contains a Composer plugin which is blocked`

**Solution - Two-step installation:**

Step 1: Create project without installing dependencies:
```bash
docker exec quickapps-web bash -c "cd /var/www/html && composer create-project -s dev quickapps/website . --no-install --no-interaction"
```

Step 2: Allow plugins and install:
```bash
docker exec quickapps-web bash -c "cd /var/www/html && composer config allow-plugins.aura/installer-default true && composer config allow-plugins.cakephp/plugin-installer true && composer install"
```

### 8. File Permissions Configuration

Set proper ownership and permissions:
```bash
docker exec quickapps-web bash -c "chown -R www-data:www-data /var/www/html && chmod -R 755 /var/www/html && chmod -R 777 /var/www/html/tmp /var/www/html/logs"
```

### 9. Installation Verification

Verified installation by checking HTTP response:
```bash
curl -I http://localhost:8080
```

**Result:** HTTP 302 redirect to `/installer/startup` - confirming successful installation

## Installed Components

### QuickAppsCMS Plugins Installed:
- block (29c65b2)
- bootstrap (ec4c4f1)
- captcha (afcd671)
- cms (d4b909b)
- comment (6a55735)
- content (3f8a575)
- eav (b89afae)
- field (9525d5c)
- installer (6c21609)
- jquery (c78bde8)
- locale (437d486)
- media-manager (5201c60)
- menu (c60e591)
- search (81ca3ef)
- system (77040ee)
- taxonomy (c8ff833)
- user (66e8501)
- wysiwyg (53e8825)

### Themes Installed:
- backend-theme (6e6907e)
- frontend-theme (d4f6e7b)

### Core Dependencies:
- cakephp/cakephp: 3.3.16
- goaop/framework: 2.1.2
- aura/intl: 1.1.1
- doctrine/annotations: 1.14.4
- mobiledetect/mobiledetectlib: 2.8.45

## Platform-Specific Considerations

### Apple Silicon (M1/M2) Compatibility
- MySQL 5.7 requires x86_64 emulation via Rosetta
- Added `platform: linux/amd64` to MySQL service in docker-compose.yml
- Performance may be slightly reduced due to emulation layer
- phpMyAdmin also runs in emulation mode

## Common Commands for Management

### Container Management
```bash
# Start containers
docker-compose up -d

# Stop containers
docker-compose down

# View logs
docker-compose logs -f

# Restart services
docker-compose restart
```

### Application Access
```bash
# Access PHP container
docker exec -it quickapps-web bash

# Access MySQL
docker exec -it quickapps-db mysql -u quickapps -pquickapps123 quickapps

# Run Composer commands
docker exec quickapps-web composer [command]
```

### Troubleshooting Commands
```bash
# Check container status
docker ps

# Fix permissions
docker exec quickapps-web bash -c "chown -R www-data:www-data /var/www/html && chmod -R 777 /var/www/html/tmp /var/www/html/logs"

# Clear cache
docker exec quickapps-web bash -c "rm -rf /var/www/html/tmp/cache/*"
```

## Final Installation State

- ✅ Docker environment configured and running
- ✅ QuickAppsCMS source code installed via Composer
- ✅ Database service operational
- ✅ Web server configured with mod_rewrite
- ✅ File permissions properly set
- ✅ Application accessible at http://localhost:8080
- ✅ Web installer ready to complete setup

## Next Required Step

Access http://localhost:8080 in a web browser to complete the web-based installation wizard using the configured database credentials.