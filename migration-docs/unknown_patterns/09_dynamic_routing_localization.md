# Dynamic Routing with Localization

## What problem was it solving?

Standard CakePHP routing is static and defined at application start. CMSs need:
- Automatic routes for all installed plugins
- Optional language prefixes (example.com/en/page vs example.com/page)
- Route generation without manually defining each plugin's routes
- SEO-friendly URLs in multiple languages

## How does it work?

**Location**: `/vendor/quickapps-plugins/cms/config/routes_site.php:68-99`

```php
// Dynamic plugin route generation
foreach ((array)Plugin::loaded() as $plugin) {
    Router::plugin($plugin, function ($routes) {
        // Default plugin routes
        $routes->connect('', ['controller' => 'main', 'action' => 'index'], ['routeClass' => 'Cake\Routing\Route\DashedRoute']);
        $routes->connect('/:controller', ['action' => 'index'], ['routeClass' => 'Cake\Routing\Route\DashedRoute']);
        $routes->connect('/:controller/:action/*', [], ['routeClass' => 'Cake\Routing\Route\DashedRoute']);
    });
}

// Language prefix injection
if (option('url_locale_prefix')) {
    $locales = array_keys(quickapps('languages'));
    $localesPattern = '(' . implode('|', array_map('preg_quote', $locales)) . ')';
    
    // Clone existing routes with language prefixes
    $existingRoutes = Router::routes();
    foreach ($existingRoutes as $route) {
        foreach ($locales as $code) {
            $template = "/{$code}" . $route->template;
            $defaults = $route->defaults + ['language' => $code];
            Router::connect($template, $defaults, $route->options);
        }
    }
    
    // URL filter to add language prefix to generated URLs
    Router::addUrlFilter(function ($params, $request) use ($localesPattern) {
        if (empty($params['language'])) {
            $params['language'] = I18n::locale();
        }
        
        if (empty($params['_base']) || !preg_match("/\/{$localesPattern}\//", $params['_base'])) {
            $params['_base'] = $request->base . '/' . $params['language'] . '/';
        }
        
        return $params;
    });
}

// Content-based routing (for CMS pages)
Router::connect('/*', ['controller' => 'contents', 'action' => 'details'], ['routeClass' => ContentRoute::class]);

// Custom route class for content
class ContentRoute extends Route
{
    public function match($url, $context = [])
    {
        // Check if URL matches a content page
        $content = TableRegistry::get('Contents')
            ->find()
            ->where(['slug' => $url['pass'][0] ?? ''])
            ->first();
            
        if ($content) {
            return [
                'controller' => 'Contents',
                'action' => 'view',
                'content_id' => $content->id,
            ];
        }
        
        return false;
    }
}
```

## What would be the modern CakePHP 5 approach?

CakePHP 5 provides better routing capabilities:

1. **Route Builders**: Use route builder classes for complex routing logic
2. **Scoped Routing**: Better organization with route scoping
3. **Route Caching**: Improved route caching mechanisms
4. **Middleware Integration**: Route-specific middleware

```php
// Modern CakePHP 5 approach
class MultilingualRouteBuilder
{
    private RouteBuilder $routes;
    private array $locales;
    
    public function __construct(RouteBuilder $routes, array $locales)
    {
        $this->routes = $routes;
        $this->locales = $locales;
    }
    
    public function buildRoutes(): void
    {
        // Create scoped routes for each locale
        foreach ($this->locales as $locale) {
            $this->routes->scope("/{$locale}", function (RouteBuilder $routes) use ($locale) {
                $routes->setDefaults(['language' => $locale]);
                $routes->applyMiddleware(LocaleMiddleware::class);
                
                $this->buildPluginRoutes($routes);
                $this->buildContentRoutes($routes);
            });
        }
        
        // Default locale without prefix
        $defaultLocale = $this->locales[0];
        $this->routes->scope('/', function (RouteBuilder $routes) use ($defaultLocale) {
            $routes->setDefaults(['language' => $defaultLocale]);
            $this->buildPluginRoutes($routes);
            $this->buildContentRoutes($routes);
        });
    }
    
    private function buildPluginRoutes(RouteBuilder $routes): void
    {
        foreach (Plugin::loaded() as $plugin) {
            $routes->plugin($plugin, function (RouteBuilder $routes) {
                $routes->setExtensions(['json', 'xml']);
                $routes->resources('Contents');
                $routes->fallbacks();
            });
        }
    }
    
    private function buildContentRoutes(RouteBuilder $routes): void
    {
        // Use route class for dynamic content routing
        $routes->connect(
            '/*',
            ['controller' => 'Contents', 'action' => 'view'],
            ['routeClass' => ContentRoute::class]
        );
    }
}

// Enhanced content route with caching
class ContentRoute extends Route
{
    private CacheInterface $cache;
    
    public function match(array $url, array $context = []): array|false
    {
        $slug = $url['pass'][0] ?? '';
        $language = $context['language'] ?? 'en';
        
        $cacheKey = "route_content_{$language}_{$slug}";
        
        return $this->cache->remember($cacheKey, function () use ($slug, $language) {
            $content = $this->findContent($slug, $language);
            
            return $content ? [
                'controller' => 'Contents',
                'action' => 'view',
                'id' => $content->id,
                'pass' => [$content->slug],
            ] : false;
        }, '1 hour');
    }
}
```

## What is the migration strategy from now to a modern solution?

1. **Phase 1**: Create RouteBuilder classes to organize current routing logic
2. **Phase 2**: Implement route caching for better performance
3. **Phase 3**: Replace URL filters with proper middleware
4. **Phase 4**: Add support for route parameters and constraints
5. **Phase 5**: Integrate with CakePHP 5's improved routing system