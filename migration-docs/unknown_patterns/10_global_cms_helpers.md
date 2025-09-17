# Global CMS Helper Functions

## What problem was it solving?

Standard CakePHP doesn't provide CMS-specific shortcuts, requiring verbose code for common operations:
- Accessing configuration values stored in database
- Plugin and theme information retrieval
- String manipulation functions missing in older PHP versions
- Quick access to CMS-specific data structures

## How does it work?

**Location**: `/vendor/quickapps-plugins/cms/config/functions.php:304-753`

```php
// Configuration shortcuts
function quickapps($key = null) {
    if ($key !== null) {
        return Configure::read("QuickApps.{$key}");
    }
    return Configure::read('QuickApps');
}

function option($name, $default = false) {
    // Check cached configuration first
    if (Configure::check("QuickApps.options.{$name}")) {
        return Configure::read("QuickApps.options.{$name}");
    }
    
    // Fallback to database query
    $option = TableRegistry::get('Options')
        ->find()
        ->where(['Options.name' => $name])
        ->first();
        
    return $option ? $option->value : $default;
}

// Plugin shortcuts
function plugin($plugin = null) {
    if ($plugin === null) {
        return Collection::make(quickapps('plugins'));
    }
    return quickapps("plugins.{$plugin}");
}

function theme($name = null) {
    if ($name === null) {
        // Auto-detect theme based on admin context
        $option = Router::getRequest()->isAdmin() ? 'back_theme' : 'front_theme';
        $name = option($option);
    }
    
    return Collection::make(quickapps('plugins'))
        ->filter(function ($plugin) use ($name) {
            return $plugin['isTheme'] && $plugin['name'] == $name;
        })
        ->first();
}

// String helpers for PHP compatibility
function str_starts_with($haystack, $needle) {
    return strpos($haystack, $needle) === 0;
}

function str_ends_with($haystack, $needle) {
    return substr($haystack, -strlen($needle)) === $needle;
}

function str_replace_once($search, $replace, $subject) {
    $pos = strpos($subject, $search);
    if ($pos !== false) {
        return substr_replace($subject, $replace, $pos, strlen($search));
    }
    return $subject;
}

// Dynamic PHP evaluation (dangerous but needed for CMS flexibility)
function php_eval($code, $args = []) {
    ob_start();
    extract($args);
    print eval('?>' . $code);
    $output = ob_get_contents();
    ob_end_clean();
    return $output;
}

// Utility functions
function normalizePath($path) {
    return str_replace(['\\', '/'], DS, $path);
}

function trimPath($path) {
    return trim($path, DS);
}

// Collection helpers
function collect($items = []) {
    return new Collection($items);
}
```

## What would be the modern CakePHP 5 approach?

Modern PHP and CakePHP 5 provide better alternatives:

1. **Service Classes**: Replace global functions with injected services
2. **PHP 8+ Features**: Use modern PHP string functions and null coalescing
3. **Configuration Service**: Centralized configuration management
4. **Utility Classes**: Organized utility methods in classes

```php
// Modern CakePHP 5 approach using services
class ConfigurationService
{
    private array $cache = [];
    private ConnectionInterface $connection;
    
    public function get(string $key, mixed $default = null): mixed
    {
        // Use null coalescing assignment (PHP 7.4+)
        return $this->cache[$key] ??= $this->fetchFromDatabase($key, $default);
    }
    
    private function fetchFromDatabase(string $key, mixed $default): mixed
    {
        return $this->connection
            ->selectQuery()
            ->select(['value'])
            ->from('options')
            ->where(['name' => $key])
            ->execute()
            ->fetchColumn() ?? $default;
    }
}

class PluginService
{
    private array $plugins = [];
    
    public function get(?string $name = null): array|Plugin|null
    {
        if ($name === null) {
            return $this->plugins;
        }
        
        return $this->plugins[$name] ?? null;
    }
    
    public function getTheme(?string $name = null): ?Theme
    {
        if ($name === null) {
            $name = $this->getCurrentTheme();
        }
        
        return collect($this->plugins)
            ->filter(fn($plugin) => $plugin->isTheme && $plugin->name === $name)
            ->first();
    }
    
    private function getCurrentTheme(): string
    {
        return $this->isAdminContext() 
            ? $this->config->get('back_theme', 'AdminTheme')
            : $this->config->get('front_theme', 'FrontendTheme');
    }
}

// Utility class instead of global functions
class StringUtil
{
    // PHP 8+ now has these functions natively
    public static function startsWith(string $haystack, string $needle): bool
    {
        return str_starts_with($haystack, $needle); // Native PHP 8+
    }
    
    public static function endsWith(string $haystack, string $needle): bool
    {
        return str_ends_with($haystack, $needle); // Native PHP 8+
    }
    
    public static function replaceOnce(string $search, string $replace, string $subject): string
    {
        $pos = strpos($subject, $search);
        return $pos !== false 
            ? substr_replace($subject, $replace, $pos, strlen($search))
            : $subject;
    }
}

// Dependency injection instead of global access
class ContentController extends Controller
{
    private ConfigurationService $config;
    private PluginService $plugins;
    
    public function __construct(
        ConfigurationService $config,
        PluginService $plugins
    ) {
        parent::__construct();
        $this->config = $config;
        $this->plugins = $plugins;
    }
    
    public function index()
    {
        $siteName = $this->config->get('site_name', 'My Site');
        $currentTheme = $this->plugins->getTheme();
        
        // Use modern syntax
        $this->set(compact('siteName', 'currentTheme'));
    }
}
```

## What is the migration strategy from now to a modern solution?

1. **Phase 1**: Create service classes that wrap existing global functions
2. **Phase 2**: Replace global function calls with service injection in controllers
3. **Phase 3**: Remove unsafe functions like `php_eval()` and replace with safer alternatives
4. **Phase 4**: Leverage PHP 8+ native functions instead of custom implementations  
5. **Phase 5**: Remove all global functions and use proper service architecture