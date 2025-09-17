# Custom Event Dispatcher System

## What problem was it solving?

Standard CakePHP 3 EventManager is global and doesn't provide debugging capabilities. QuickAppsCMS needed:
- Named event manager instances for different contexts
- Event logging and statistics for debugging
- Better event organization and isolation between plugins

## How does it work?

**Location**: `/vendor/quickapps-plugins/cms/src/Event/EventDispatcher.php`

```php
class EventDispatcher
{
    protected static $_instances = [];
    protected $_log = [];
    protected $_eventManager;
    
    // Named instances for different contexts
    public static function instance($name = 'default')
    {
        if (!isset(static::$_instances[$name])) {
            static::$_instances[$name] = new EventDispatcher();
        }
        return static::$_instances[$name];
    }
    
    // Enhanced event triggering with logging
    public function trigger($eventName)
    {
        $data = func_get_args();
        array_shift($data);
        $event = $this->_prepareEvent($eventName, $data);
        
        // Log event for debugging
        $this->_log($event->name());
        
        $this->_eventManager->dispatch($event);
        return $event;
    }
    
    // Event statistics and debugging
    public function triggered($eventName = null, $sort = true)
    {
        if ($eventName === null) {
            if ($sort) {
                arsort($this->_log, SORT_NATURAL);
            }
            return $this->_log; // All events with counts
        }
        return isset($this->_log[$eventName]) ? $this->_log[$eventName] : 0;
    }
    
    private function _log($eventName)
    {
        if (!isset($this->_log[$eventName])) {
            $this->_log[$eventName] = 0;
        }
        $this->_log[$eventName]++;
    }
}
```

## What would be the modern CakePHP 5 approach?

CakePHP 5 has improved event handling that could replace this pattern:

1. **Typed Events**: Use strongly typed event classes
2. **Event Middleware**: Implement event logging via middleware
3. **Performance Profiling**: Use the built-in profiler for event timing
4. **Dependency Injection**: Inject event managers where needed

```php
// Modern CakePHP 5 approach
class EnhancedEventManager implements EventManagerInterface
{
    private EventManagerInterface $eventManager;
    private LoggerInterface $logger;
    private array $statistics = [];
    
    public function dispatch(object $event): object
    {
        $eventName = $event::class;
        $startTime = microtime(true);
        
        // Log event start
        $this->logger->debug("Event triggered: {$eventName}");
        
        // Dispatch event
        $result = $this->eventManager->dispatch($event);
        
        // Record statistics
        $duration = microtime(true) - $startTime;
        $this->recordEventStats($eventName, $duration);
        
        return $result;
    }
    
    private function recordEventStats(string $eventName, float $duration): void
    {
        if (!isset($this->statistics[$eventName])) {
            $this->statistics[$eventName] = ['count' => 0, 'total_time' => 0.0];
        }
        
        $this->statistics[$eventName]['count']++;
        $this->statistics[$eventName]['total_time'] += $duration;
    }
    
    public function getStatistics(): array
    {
        return $this->statistics;
    }
}
```

## What is the migration strategy from now to a modern solution?

1. **Phase 1**: Wrap existing EventDispatcher to implement EventManagerInterface
2. **Phase 2**: Convert custom events to typed event classes  
3. **Phase 3**: Replace event logging with PSR-3 compatible logger
4. **Phase 4**: Integrate with CakePHP 5's profiling tools
5. **Phase 5**: Remove custom EventDispatcher in favor of enhanced EventManager