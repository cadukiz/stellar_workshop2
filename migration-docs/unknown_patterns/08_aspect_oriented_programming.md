# Aspect-Oriented Programming (AOP)

## What problem was it solving?

Traditional OOP can't elegantly handle cross-cutting concerns like:
- Logging method calls across the entire application
- Performance monitoring and profiling
- Security checks and access control
- Caching method results
- Transaction management

These concerns would require modifying every class, violating DRY and separation of concerns.

## How does it work?

**Location**: `/vendor/quickapps-plugins/cms/src/Aspect/AppAspect.php` and bootstrap files

QuickAppsCMS integrates the GoAOP library to enable method interception:

```php
// Bootstrap AOP kernel
AppAspect::getInstance()->init([
    'debug' => Configure::read('debug'),
    'cacheDir' => TMP . 'aop',
    'includePaths' => $includePaths,
    'excludePaths' => $excludePaths,
    'features' => \Go\Aop\Features::INTERCEPT_FUNCTIONS,
]);

// Aspect registration
class AppAspect extends AspectKernel
{
    protected function configureAop(AspectContainer $container)
    {
        // Register all aspects from plugins
        foreach ((array)aspects() as $class) {
            $class = class_exists($class) ? new $class : null;
            if ($class instanceof Aspect) {
                $container->registerAspect($class);
            }
        }
    }
}

// Example aspect for logging
class LoggingAspect implements Aspect
{
    /**
     * @Around("execution(public|protected **->save(*))")
     */
    public function aroundSave(MethodInvocation $invocation)
    {
        $start = microtime(true);
        $className = get_class($invocation->getThis());
        $methodName = $invocation->getMethod()->name;
        
        Log::debug("Before {$className}::{$methodName}");
        
        try {
            $result = $invocation->proceed();
            $duration = microtime(true) - $start;
            Log::debug("After {$className}::{$methodName} ({$duration}s)");
            return $result;
        } catch (Exception $e) {
            Log::error("Exception in {$className}::{$methodName}: " . $e->getMessage());
            throw $e;
        }
    }
}
```

## What would be the modern CakePHP 5 approach?

Modern approaches avoid AOP complexity in favor of:

1. **Decorator Pattern**: Wrap services with logging/caching decorators
2. **Middleware**: Use middleware for cross-cutting concerns
3. **Event Listeners**: Use events for loose coupling
4. **Dependency Injection**: Inject concerns via DI container

```php
// Modern CakePHP 5 approach using decorators
class LoggingServiceDecorator
{
    private object $service;
    private LoggerInterface $logger;
    
    public function __construct(object $service, LoggerInterface $logger)
    {
        $this->service = $service;
        $this->logger = $logger;
    }
    
    public function __call(string $method, array $arguments)
    {
        $start = microtime(true);
        $className = get_class($this->service);
        
        $this->logger->debug("Calling {$className}::{$method}");
        
        try {
            $result = $this->service->$method(...$arguments);
            $duration = microtime(true) - $start;
            $this->logger->debug("Completed {$className}::{$method} in {$duration}s");
            return $result;
        } catch (Throwable $e) {
            $this->logger->error("Error in {$className}::{$method}: {$e->getMessage()}");
            throw $e;
        }
    }
}

// Service registration with decorators
class ServiceFactory
{
    private ContainerInterface $container;
    
    public function createUserService(): UserServiceInterface
    {
        $service = new UserService($this->container->get(UserRepository::class));
        
        // Wrap with logging
        $service = new LoggingServiceDecorator($service, $this->container->get(LoggerInterface::class));
        
        // Wrap with caching
        $service = new CachingServiceDecorator($service, $this->container->get(CacheInterface::class));
        
        return $service;
    }
}

// Event-driven approach
class UserService
{
    private EventManagerInterface $events;
    
    public function save(User $user): User
    {
        $event = new Event('User.beforeSave', $this, ['user' => $user]);
        $this->events->dispatch($event);
        
        if ($event->isStopped()) {
            throw new ValidationException('Save operation was cancelled');
        }
        
        $savedUser = $this->repository->save($user);
        
        $this->events->dispatch(new Event('User.afterSave', $this, ['user' => $savedUser]));
        
        return $savedUser;
    }
}
```

## What is the migration strategy from now to a modern solution?

1. **Phase 1**: Identify all aspects and their use cases
2. **Phase 2**: Implement decorator pattern for service-level concerns
3. **Phase 3**: Replace method interception with event-driven approach
4. **Phase 4**: Use middleware for request/response concerns
5. **Phase 5**: Remove GoAOP dependency entirely