# Dynamic Plugin Loading System

## What problem was it solving?

Standard CakePHP 3 requires plugins to be statically loaded in `config/bootstrap.php`, making it impossible to enable/disable plugins without code changes. For a CMS, users need to install, activate, deactivate, and uninstall plugins through the admin interface without touching code or restarting the application.

## How does it work?

**Location**: `/vendor/quickapps-plugins/cms/config/bootstrap_site.php:199-248`

The system works in several phases:

1. **Database-Driven Discovery**: Plugins are stored in a `plugins` database table with status flags
2. **Dynamic PSR-4 Registration**: Active plugins get their namespaces registered at runtime
3. **Conditional Loading**: Only active plugins (and active themes) are loaded
4. **Automatic Route Generation**: Each plugin gets automatic routes created

```php
// Dynamic plugin loading with snapshot system
plugin()
    ->each(function ($plugin) use (&$pluginsPath, $classLoader) {
        if (strtoupper($plugin->name) === 'CMS') {
            return; // CMS is always loaded
        }
        
        // Filter by status and theme selection
        $filter = $plugin->status;
        if ($plugin->isTheme) {
            $filter = $filter && in_array($plugin->name, [option('front_theme'), option('back_theme')]);
        }
        
        if (!$filter) {
            return; // Skip inactive plugins
        }
        
        // Dynamic PSR-4 namespace registration
        if (!in_array("{$plugin->name}\\", array_keys($classLoader->getPrefixesPsr4()))) {
            $classLoader->addPsr4("{$plugin->name}\\", normalizePath("{$plugin->path}/src/"), true);
        }
        
        Plugin::load($plugin->name, $info);
    });
```

## What would be the modern CakePHP 5 approach?

CakePHP 5 introduces several improvements that could modernize this pattern:

1. **Plugin Objects**: Use the new Plugin classes with better lifecycle management
2. **Dependency Injection**: Leverage the DI container for plugin services
3. **Event-Driven Loading**: Use events for plugin activation/deactivation
4. **Console Commands**: Implement `bin/cake plugin install/uninstall` commands

```php
// Modern CakePHP 5 approach
class PluginManager
{
    private ContainerInterface $container;
    private EventManagerInterface $events;
    
    public function activate(string $pluginName): bool
    {
        $plugin = $this->container->get($pluginName . 'Plugin');
        
        // Trigger before activation event
        $event = new Event('Plugin.beforeActivate', $this, ['plugin' => $plugin]);
        $this->events->dispatch($event);
        
        if ($event->isStopped()) {
            return false;
        }
        
        // Load plugin using modern Plugin class
        $this->application->addPlugin($plugin);
        
        // Update database status
        $this->updatePluginStatus($pluginName, true);
        
        // Trigger after activation event
        $this->events->dispatch(new Event('Plugin.afterActivate', $this, ['plugin' => $plugin]));
        
        return true;
    }
}
```

## What is the migration strategy from now to a modern solution?

1. **Phase 1**: Create a PluginManager service class that wraps current functionality
2. **Phase 2**: Implement proper dependency injection for plugin services
3. **Phase 3**: Add console commands for plugin management
4. **Phase 4**: Replace database-driven loading with event-driven activation
5. **Phase 5**: Add plugin marketplace integration