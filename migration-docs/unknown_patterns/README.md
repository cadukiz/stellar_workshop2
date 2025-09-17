# QuickAppsCMS Unknown Patterns

This directory contains documentation for patterns used in QuickAppsCMS that are NOT standard CakePHP 3 framework patterns. These patterns represent unique architectural decisions made by the QuickAppsCMS team to solve CMS-specific challenges.

## Pattern Overview

Each pattern is documented with:
1. **What problem was it solving?** - The motivation behind the pattern
2. **How does it work?** - Technical implementation details with code examples
3. **What would be the modern CakePHP 5 approach?** - How to implement this using modern CakePHP 5 patterns
4. **What is the migration strategy?** - Step-by-step migration plan from current to modern implementation

## Patterns Documented

### 1. [Dynamic Plugin Loading System](01_dynamic_plugin_loading.md)
**Problem**: Enable/disable plugins through admin interface without code changes
**Solution**: Database-driven plugin discovery with runtime PSR-4 registration
**Location**: `/vendor/quickapps-plugins/cms/config/bootstrap_site.php:199-248`

### 2. [Snapshot Configuration System](02_snapshot_configuration.md)
**Problem**: Database-driven configuration with performance optimization
**Solution**: Cached configuration files generated from database state
**Location**: `/vendor/quickapps-plugins/cms/config/functions.php:50-254`

### 3. [EAV (Entity-Attribute-Value) Implementation](03_eav_implementation.md)
**Problem**: Add custom fields to content types without database migrations
**Solution**: Three-table EAV system with behavior-based query modification
**Location**: `/vendor/quickapps-plugins/eav/src/Model/Behavior/EavBehavior.php`

### 4. [Custom Event Dispatcher System](04_custom_event_dispatcher.md)
**Problem**: Event logging, statistics, and isolation between plugins
**Solution**: Named event manager instances with debugging capabilities
**Location**: `/vendor/quickapps-plugins/cms/src/Event/EventDispatcher.php`

### 5. [Three-Tier Template System](05_three_tier_template_system.md)
**Problem**: Separate admin and public templates while sharing common components
**Solution**: Back/Common/Front template organization with context detection
**Location**: Template directory structure

### 6. [Pluggable Field Handler System](06_field_handler_system.md)
**Problem**: Extensible field types with validation, rendering, and storage logic
**Solution**: Abstract Handler class with lifecycle hooks and plugin integration
**Location**: `/vendor/quickapps-plugins/field/src/Handler.php`

### 7. [Multi-Method Authentication System](07_multi_method_authentication.md)
**Problem**: "Remember me", token auth, anonymous permissions, and role-based access
**Solution**: Authentication cascade with cookie, token, and ACL fallbacks
**Location**: `/vendor/quickapps-plugins/user/src/Auth/AnonymousAuthenticate.php`

### 8. [Aspect-Oriented Programming (AOP)](08_aspect_oriented_programming.md)
**Problem**: Cross-cutting concerns like logging, security, and performance monitoring
**Solution**: GoAOP library integration for method interception
**Location**: `/vendor/quickapps-plugins/cms/src/Aspect/AppAspect.php`

### 9. [Dynamic Routing with Localization](09_dynamic_routing_localization.md)
**Problem**: Automatic plugin routes with optional language prefixes
**Solution**: Runtime route generation with language URL filters
**Location**: `/vendor/quickapps-plugins/cms/config/routes_site.php:68-99`

### 10. [Global CMS Helper Functions](10_global_cms_helpers.md)
**Problem**: Verbose code for common CMS operations and missing PHP functions
**Solution**: Global utility functions for configuration, plugins, and string manipulation
**Location**: `/vendor/quickapps-plugins/cms/config/functions.php:304-753`

## Migration Priority

These patterns should be migrated in order of criticality:

### High Priority (Core Architecture)
1. **Plugin Loading System** - Foundation of the CMS architecture
2. **EAV Implementation** - Core content flexibility
3. **Authentication System** - Security and user management
4. **Configuration System** - Application behavior control

### Medium Priority (Development Experience)
5. **Field Handler System** - Content type extensibility
6. **Template System** - Theme and admin interface separation
7. **Event Dispatcher** - Plugin communication

### Lower Priority (Convenience Features)
8. **Dynamic Routing** - URL management and localization
9. **AOP System** - Cross-cutting concerns (can be replaced with simpler patterns)
10. **Global Functions** - Developer convenience (can be gradually replaced)

## Security Considerations

⚠️ **Critical Security Pattern**: The `php_eval()` function in Global CMS Helpers (#10) poses a significant security risk and should be prioritized for removal or sandboxing during migration.

## Performance Impact

Several patterns have performance implications:
- **EAV queries** can be slow for large datasets
- **Dynamic routing** adds overhead to request processing
- **AOP method interception** impacts method call performance
- **Configuration caching** is essential for database-driven configuration

## Migration Strategy Overview

1. **Analysis Phase**: Understand current usage and dependencies
2. **Wrapper Phase**: Create modern interfaces wrapping existing functionality
3. **Service Phase**: Implement proper dependency injection and services
4. **Migration Phase**: Gradually replace old patterns with new implementations
5. **Cleanup Phase**: Remove deprecated patterns and optimize performance

Each pattern document provides detailed migration steps specific to that pattern.

## Additional Resources

- [Project Audit](../PROJECT_AUDIT.MD) - Complete project analysis
- [Plugin Dependency Map](../PLUGINS_DEPENDENCY_MAP.md) - Plugin relationships
- [Controller Dependency Map](../CONTROLLERS_DEPENDENCY_MAP.md) - Controller architecture

## Usage Notes

These patterns enable QuickAppsCMS to function as a full CMS while maintaining CakePHP 3 compatibility. The migration to CakePHP 5 will require careful architectural decisions to maintain this functionality while adopting modern patterns like dependency injection, middleware, typed events, and enhanced service layers.