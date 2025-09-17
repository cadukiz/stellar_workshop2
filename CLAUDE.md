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

### 📊 DEPENDENCY_MATRIX.md
**Objective**: Complete cross-reference matrix of all plugin dependencies
- **Contents**: Detailed dependency relationships, version constraints, loading order requirements
- **Key Sections**: Plugin-to-plugin dependencies, composer requirements, circular dependency analysis
- **Use Case**: Resolving complex plugin migration order and dependency conflicts
- **Critical Info**: Critical path dependencies, version compatibility matrix, migration blockers

### 🏗️ CakePHP_3_TO_5_GUIDE.md
**Objective**: Comprehensive migration guide from CakePHP 3.3.16 to CakePHP 5.x
- **Contents**: Breaking changes, upgrade strategies, code transformation patterns
- **Key Sections**: Authentication changes, routing updates, ORM modifications, deprecated features
- **Use Case**: Step-by-step migration execution and breaking change resolution
- **Critical Info**: Critical breaking changes, migration timeline, testing strategies

### 🗄️ DATABASE_SCHEMA.md
**Objective**: Complete database schema documentation and analysis
- **Contents**: All 50+ tables, relationships, constraints, indexes, EAV structure
- **Key Sections**: Core tables, plugin tables, EAV model, custom field storage
- **Use Case**: Database migration planning and schema compatibility validation
- **Critical Info**: Critical relationships, data integrity constraints, migration SQL scripts

### ⚠️ DATABASE_MIGRATION_ISSUES.md
**Objective**: Known database migration challenges and solutions
- **Contents**: MySQL 5.7→8.0 issues, data migration strategies, compatibility problems
- **Key Sections**: SQL mode changes, character set issues, constraint violations
- **Use Case**: Resolving database migration blockers and data integrity issues
- **Critical Info**: Breaking SQL changes, data migration scripts, rollback strategies

### 🔧 UNKNOWN_PATTERNS/
**Objective**: Document non-standard CakePHP patterns unique to QuickAppsCMS
- **Contents**: 10 custom architectural patterns with migration strategies
- **Key Sections**: Dynamic plugin loading, EAV implementation, custom authentication, AOP integration
- **Use Case**: Understanding QuickAppsCMS-specific patterns that require special migration consideration
- **Critical Info**: Security implications (php_eval), performance impacts, modern CakePHP 5 alternatives

### 🚀 API.md
**Objective**: Comprehensive API documentation for QuickAppsCMS routes and endpoints
- **Contents**: Complete route catalog (40+ endpoints), authentication system, request/response patterns
- **Key Sections**: User management, content management, file handling, admin operations, search & RSS
- **Use Case**: Understanding current API structure for migration planning and endpoint preservation
- **Critical Info**: Session-based auth, role-based access control, file upload security, error handling patterns

### 📊 openapi.yaml
**Objective**: Machine-readable API specification in OpenAPI 3.0.3 format
- **Contents**: Structured endpoint definitions, schemas, authentication, interactive documentation
- **Key Sections**: All 40+ endpoints with parameters, request/response examples, security schemes
- **Use Case**: API testing, client generation, migration validation, developer reference
- **Critical Info**: Swagger-compatible specification for API exploration and testing tools

### 🧪 TEST_PLAN.md
**Objective**: Master testing strategy for migration validation
- **Contents**: Comprehensive testing approach, validation criteria, acceptance tests
- **Key Sections**: Plugin testing matrix, integration tests, performance benchmarks
- **Use Case**: Ensuring migration quality and feature compatibility
- **Critical Info**: Test coverage requirements, validation criteria, acceptance thresholds

### 📁 test-plans/
**Objective**: Feature-specific testing plans for each plugin (16 detailed plans)
- **Contents**: Individual test plans for each plugin with specific test cases
- **Key Sections**: Unit tests, integration tests, user acceptance tests per plugin
- **Use Case**: Systematic plugin-by-plugin testing during migration
- **Critical Info**: Plugin-specific test scenarios, validation steps, rollback procedures

### Migration Documentation Usage
These documents should be consulted in this recommended order:

**Phase 1: Understanding & Analysis**
1. **PROJECT_AUDIT.MD** - Get overall understanding of the system architecture
2. **DEPENDENCY_MATRIX.md** - Understand complete dependency relationships
3. **PLUGINS_DEPENDENCY_MAP.md** - Plan plugin migration sequence
4. **CONTROLLERS_DEPENDENCY_MAP.md** - Understand controller migration complexity
5. **DATABASE_SCHEMA.md** - Analyze database structure and relationships

**Phase 2: Migration Planning**
6. **CakePHP_3_TO_5_GUIDE.md** - Review breaking changes and migration strategies
7. **DATABASE_MIGRATION_ISSUES.md** - Understand database migration challenges
8. **API.md** - Document current API structure for preservation
9. **UNKNOWN_PATTERNS/** - Identify custom patterns requiring special handling

**Phase 3: Implementation & Testing**
10. **openapi.yaml** - Use for API testing and validation during migration
11. **TEST_PLAN.md** - Follow comprehensive testing strategy
12. **test-plans/** - Execute plugin-specific test plans during implementation

## File Locations

### Key Files to Monitor
- `quickapps-cakephp3/src/config/settings.php` - Core CMS configuration
- `quickapps-cakephp3/src/plugins/` - Custom plugins (if any)
- `quickapps-cakephp5/src/config/app_local.php` - Target app configuration

**Migration Analysis Documentation:**
- `migration-docs/PROJECT_AUDIT.md` - Complete system architecture analysis
- `migration-docs/DEPENDENCY_MATRIX.md` - Cross-reference matrix of all dependencies
- `migration-docs/PLUGINS_DEPENDENCY_MAP.md` - Visual plugin dependency relationships
- `migration-docs/CONTROLLERS_DEPENDENCY_MAP.md` - Controller architecture and inheritance
- `migration-docs/DATABASE_SCHEMA.md` - Complete database schema documentation
- `migration-docs/DATABASE_MIGRATION_ISSUES.md` - Known database migration challenges
- `migration-docs/CakePHP_3_TO_5_GUIDE.md` - Comprehensive CakePHP 3→5 migration guide

**API Documentation:**
- `migration-docs/API.md` - Complete API documentation with routes and endpoints
- `migration-docs/openapi.yaml` - OpenAPI 3.0.3 specification for API testing and validation

**Custom Patterns & Testing:**
- `migration-docs/unknown_patterns/` - Custom QuickAppsCMS architectural patterns (10 patterns)
- `migration-docs/TEST_PLAN.md` - Master testing plan for migration validation
- `migration-docs/test-plans/` - Feature-specific test plans (16 detailed plugin plans)

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
5. **Review API endpoints** in `/migration-docs/API.md` that may be affected by the component
6. **Validate route compatibility** using `/migration-docs/openapi.yaml` for endpoint testing
7. **Review unknown patterns** in `/migration-docs/unknown_patterns/` that may affect the component
8. **Execute feature test plan** from `/migration-docs/test-plans/TEST_PLAN_[FEATURE].md`
9. Validate admin interface functionality
10. Test public-facing features
11. Verify permission systems work correctly
12. **Test API endpoints** to ensure request/response compatibility is maintained
13. **Confirm all test scenarios pass** before marking migration complete

The QuickAppsCMS plugin ecosystem is tightly integrated, so changes to one plugin often affect others. **Pay special attention to the 10 documented unknown patterns and 40+ API endpoints** as they represent critical architectural decisions that require careful migration planning.

## API Testing & Validation

When working with API-related migration tasks:
- **Use `/migration-docs/API.md`** for understanding current endpoint behavior and authentication
- **Use `/migration-docs/openapi.yaml`** with Swagger UI or Postman for interactive API testing
- **Validate session-based authentication** works correctly after migration
- **Test file upload endpoints** with proper multipart form data
- **Verify admin routes** maintain role-based access control
- **Check RSS and search endpoints** for proper response formatting
- Update the migration-docs files