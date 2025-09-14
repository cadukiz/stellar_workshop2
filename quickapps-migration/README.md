# QuickAppsCMS Docker Installation

This project sets up QuickAppsCMS (CakePHP 3.3.16) in a containerized environment for future migration to CakePHP 5.

## Current Setup

- **PHP**: 7.4 (with all required extensions)
- **Web Server**: Apache with mod_rewrite
- **Database**: MySQL 5.7
- **CakePHP**: 3.3.16
- **QuickAppsCMS**: 2.0 (dev-master)

## Services

- **Web Application**: http://localhost:8080
- **phpMyAdmin**: http://localhost:8081
- **MySQL**: localhost:3306

## Database Credentials

- **Database Name**: quickapps
- **Username**: quickapps
- **Password**: quickapps123
- **Root Password**: rootpassword

## Installation Steps Completed

1. ✅ Created Docker environment with PHP 7.4 and required extensions
2. ✅ Configured Apache with mod_rewrite for CakePHP
3. ✅ Set up MySQL 5.7 database container
4. ✅ Installed QuickAppsCMS using Composer
5. ✅ Set proper file permissions

## Getting Started

1. Ensure Docker is running on your machine
2. Navigate to the project directory
3. Start the containers:
   ```bash
   docker-compose up -d
   ```
4. Access the installer at: http://localhost:8080
5. Complete the web-based installation using the database credentials above

## Managing the Application

### Start containers
```bash
docker-compose up -d
```

### Stop containers
```bash
docker-compose down
```

### View logs
```bash
docker-compose logs -f
```

### Access PHP container
```bash
docker exec -it quickapps-web bash
```

### Access MySQL
```bash
docker exec -it quickapps-db mysql -u quickapps -pquickapps123 quickapps
```

## Next Steps for Migration

1. Complete the QuickAppsCMS installation through the web interface
2. Document the current application structure and dependencies
3. Create a migration plan from CakePHP 3.3.16 to CakePHP 5.x
4. Test the application thoroughly in the current state
5. Begin incremental migration to CakePHP 5

## Important Notes

- The MySQL container runs in x86_64 emulation mode on Apple Silicon (ARM64) for compatibility
- All application files are in the `src/` directory
- Database data is persisted in a Docker volume
- The `.env` file contains environment variables for the containers

## Troubleshooting

If you encounter permission issues:
```bash
docker exec quickapps-web bash -c "chown -R www-data:www-data /var/www/html && chmod -R 777 /var/www/html/tmp /var/www/html/logs"
```

If you need to reinstall QuickAppsCMS:
```bash
docker exec quickapps-web bash -c "rm -rf /var/www/html/* && cd /var/www/html && composer create-project -s dev quickapps/website . --no-install"
docker exec quickapps-web bash -c "cd /var/www/html && composer config allow-plugins.aura/installer-default true && composer config allow-plugins.cakephp/plugin-installer true && composer install"
```