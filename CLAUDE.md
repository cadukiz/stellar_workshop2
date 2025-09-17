# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a CakePHP 3 to CakePHP 5 migration project containing QuickAppsCMS (a plugin-based CMS) that needs to be migrated from CakePHP 3.3.16 to CakePHP 5.x. Both applications run simultaneously on different ports to enable comparison and gradual migration.

## Development Environment

### QuickAppsCMS (CakePHP 3) - Source Application
- **Location**: `quickapps-cakephp3/`
- **Web**: http://localhost:8080
- **Database**: localhost:3306 (MySQL 5.7)
- **phpMyAdmin**: http://localhost:8081
- **PHP**: 7.4 with mcrypt extension

### CakePHP 5 - Target Application  
- **Location**: `quickapps-cakephp5/`
- **Web**: http://localhost:8090
- **Database**: localhost:3307 (MySQL 8.0)
- **phpMyAdmin**: http://localhost:8091
- **PHP**: 8.2

## Essential Commands

### Docker Management
```bash
# Start both environments
docker-compose -f quickapps-cakephp3/docker-compose.yml up -d
docker-compose -f quickapps-cakephp5/docker-compose.yml up -d

# Access containers
docker exec -it quickapps-web bash      # CakePHP 3 container
docker exec -it quickapps5-web bash     # CakePHP 5 container

# Database access
docker exec quickapps-db mysql -u quickapps -pquickapps123 quickapps
docker exec quickapps5-db mysql -u quickapps5 -pquickapps123 quickapps5

# Reset databases (for clean testing)
docker exec quickapps-db mysql -u root -prootpassword -e "DROP DATABASE quickapps; CREATE DATABASE quickapps CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
```

### CakePHP 5 Development
```bash
cd quickapps-cakephp5/src
composer check          # Run all checks (tests + standards)
composer test           # Run PHPUnit tests
composer cs-check       # Check coding standards  
composer cs-fix         # Fix coding standards
composer stan           # Run PHPStan static analysis
bin/cake server -p 8765 # Built-in development server
```

### Database Management
```bash
# Export QuickAppsCMS schema for analysis
docker exec quickapps-db mysqldump -u quickapps -pquickapps123 --no-data quickapps > schema_cakephp3.sql

# Compare database schemas
docker exec quickapps-db mysql -u quickapps -pquickapps123 -e "SHOW TABLES;" quickapps
docker exec quickapps5-db mysql -u quickapps5 -pquickapps123 -e "SHOW TABLES;" quickapps5
```

## Architecture Overview

### QuickAppsCMS Plugin Architecture

QuickAppsCMS is built on a modular plugin system with 17 core plugins, each providing specific CMS functionality:

**Core Plugins** (located in `/vendor/quickapps-plugins/`):
- `cms` - Core CMS functionality and base classes
- `user` - Authentication and user management
- `content` - Content management with custom content types
- `field` - Custom field system for extending content
- `eav` - Entity-Attribute-Value model for flexible data
- `taxonomy` - Content categorization system
- `menu` - Navigation management
- `block` - Content block system
- `media-manager` - File and media handling
- `search` - Full-text search functionality
- `comment` - Commenting system
- `locale` - Internationalization support

**Template Structure**:
```
templates/
├── Back/     # Admin interface (management UI)
├── Common/   # Shared components and layouts
└── Front/    # Public-facing templates
```

**Key Architectural Patterns**:
- Plugin-based modularity with cross-plugin dependencies
- Three-tier template system (Back/Common/Front)
- EAV model for flexible content attributes
- Event-driven plugin communication
- Custom authentication system integrated with CakePHP auth

### Migration Strategy

The migration follows a three-phase approach:

1. **Documentation Phase**: Map all plugins, custom fields, content types, and configurations
2. **Analysis Phase**: Identify CakePHP 3→5 breaking changes and compatibility issues  
3. **Implementation Phase**: Gradual migration starting with core functionality

### Configuration Differences

**CakePHP 3 (QuickAppsCMS)**:
- Array-based configuration in `/config/settings.php`
- Plugin loading through custom bootstrap
- SQL mode: `NO_ENGINE_SUBSTITUTION` (non-strict)

**CakePHP 5 (Target)**:
- Environment-based configuration with `.env` files
- Modern plugin loading system
- Enhanced middleware and dependency injection

## Database Schema

### Critical Database Considerations
- QuickAppsCMS uses EAV model extensively (`eav_attributes`, `eav_values` tables)
- Content types are dynamic and stored in database
- Custom fields create runtime database relationships
- User roles and permissions have custom implementation
- MySQL 5.7→8.0 migration requires SQL mode adjustments

### Schema Analysis Commands
```bash
# Analyze EAV structure
docker exec quickapps-db mysql -u quickapps -pquickapps123 quickapps -e "DESCRIBE eav_attributes;"

# Check content types
docker exec quickapps-db mysql -u quickapps -pquickapps123 quickapps -e "SELECT * FROM content_types;"

# Plugin-specific tables
docker exec quickapps-db mysql -u quickapps -pquickapps123 quickapps -e "SHOW TABLES LIKE '%plugin%';"
```

## Development Workflow

### Plugin Analysis Workflow
1. Identify plugin dependencies: Check `composer.json` and plugin loading order
2. Map database tables: Each plugin may create custom tables
3. Document template overrides: Plugins can override core templates
4. Catalog custom fields: EAV system creates runtime relationships
5. Note admin interfaces: Back templates provide management UI

### Migration Testing
```bash
# Test both environments are functional
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080  # Should return 200
curl -s -o /dev/null -w "%{http_code}" http://localhost:8090  # Should return 200

# Database connectivity tests
docker exec quickapps-web php -r "echo (new PDO('mysql:host=db;dbname=quickapps', 'quickapps', 'quickapps123')) ? 'Connected' : 'Failed';"
```

### Common Issues & Solutions

**mcrypt Extension**: CakePHP 3 requires mcrypt - already configured in Dockerfile
**SQL Strict Mode**: Disabled via `--sql-mode="NO_ENGINE_SUBSTITUTION"` in docker-compose
**Port Conflicts**: Applications use different port ranges (8080-8081 vs 8090-8091)
**Container Naming**: Uses `quickapps-*` vs `quickapps5-*` prefixes to avoid conflicts

## Migration Documentation

The `/migration-docs/` folder contains comprehensive analysis documents for the CakePHP 3→5 migration:

### 📋 PROJECT_AUDIT.MD
**Objective**: Complete x-ray analysis of QuickAppsCMS architecture
- **Contents**: Folder structure, 17 core plugins, dependencies, configuration files, components
- **Key Sections**: Plugin ecosystem, EAV model, template system (Back/Common/Front), security features
- **Use Case**: Understanding the overall project complexity and migration scope
- **Critical Info**: Plugin dependency matrix, migration critical points, recommended strategy

### 🗺️ PLUGINS_DEPENDENCY_MAP.md  
**Objective**: Visual dependency relationships between all QuickAppsCMS plugins
- **Contents**: Mermaid diagram showing plugin dependencies, migration risks, resolution order
- **Key Sections**: Dependency statistics, critical migration paths, risk assessment
- **Use Case**: Planning plugin migration sequence and identifying circular dependencies
- **Critical Info**: Foundation plugins (cms, eav), high-risk dependencies (GoAOP), migration timeline

### 🧩 CONTROLLERS_DEPENDENCY_MAP.md
**Objective**: Controller architecture analysis and inheritance mapping
- **Contents**: Controller hierarchy, traits/components usage, admin vs public routes
- **Key Sections**: 42 controllers, inheritance patterns, trait analysis, API endpoints
- **Use Case**: Understanding controller migration complexity and authentication system
- **Critical Info**: CMS foundation controller, custom auth system, admin interface architecture

### Migration Documentation Usage
These documents should be consulted in order:
1. **PROJECT_AUDIT.MD** - Get overall understanding
2. **PLUGINS_DEPENDENCY_MAP.md** - Plan plugin migration order  
3. **CONTROLLERS_DEPENDENCY_MAP.md** - Understand controller migration complexity

## File Locations

### Key Files to Monitor
- `quickapps-cakephp3/src/config/settings.php` - Core CMS configuration
- `quickapps-cakephp3/src/plugins/` - Custom plugins (if any)
- `quickapps-cakephp5/src/config/app_local.php` - Target app configuration
- `migration-docs/` - Migration analysis documentation

### Plugin Locations
- Core plugins: `/vendor/quickapps-plugins/`
- Themes: `/vendor/quickapps-themes/`
- Custom plugins: `/src/plugins/` (check for local customizations)

## Migration Checklist

When migrating functionality, always:
1. Document the original plugin's database schema
2. Identify template dependencies (Back/Common/Front structure)
3. Map custom fields and EAV relationships  
4. Check for plugin interdependencies
5. Validate admin interface functionality
6. Test public-facing features
7. Verify permission systems work correctly

The QuickAppsCMS plugin ecosystem is tightly integrated, so changes to one plugin often affect others.