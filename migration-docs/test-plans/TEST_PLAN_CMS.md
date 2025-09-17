# Test Plan: CMS Core Foundation Plugin

## Feature Overview
The CMS plugin provides the foundational infrastructure for QuickAppsCMS including base controller classes, extended helpers, event system integration, and core CMS functionality that all other plugins depend on.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/cms/src/Controller/Controller.php` - Base controller class
- Plugin AppControllers inherit from CMS base controller

### Models
- No specific tables (provides base classes and traits)
- `src/vendor/quickapps-plugins/cms/src/Model/Table/Table.php` - Base table class
- `src/vendor/quickapps-plugins/cms/src/Model/Entity/Entity.php` - Base entity class

### Views & Helpers
- `src/vendor/quickapps-plugins/cms/src/View/Helper.php` - Base helper class
- `src/vendor/quickapps-plugins/cms/src/View/Helper/FormHelper.php` - Extended form functionality
- `src/vendor/quickapps-plugins/cms/src/View/Helper/HtmlHelper.php` - Extended HTML functionality
- `src/vendor/quickapps-plugins/cms/src/Template/` - Base templates

### Core Classes
- `src/vendor/quickapps-plugins/cms/src/Core/Plugin.php` - Plugin management
- `src/vendor/quickapps-plugins/cms/src/Event/` - Event listeners
- `src/vendor/quickapps-plugins/cms/src/Utility/` - Core utilities

### Database Tables
- No plugin-specific tables
- Provides base behaviors and traits for other plugins

## Routes & URLs

### Core Routing
- CMS plugin handles base routing configuration
- Plugin discovery and route loading
- Dynamic route generation based on installed plugins

### Admin Routes (Inherited by all plugins)
| Route Pattern | Base Controller | Description |
|---------------|----------------|-------------|
| `/admin/{plugin}/{controller}` | `CMS\Controller\Controller` | Admin base routing |
| `/admin/{plugin}/{controller}/{action}` | `CMS\Controller\Controller` | Admin action routing |

### Frontend Routes (Inherited by all plugins)
| Route Pattern | Base Controller | Description |
|---------------|----------------|-------------|
| `/{plugin}/{controller}` | `CMS\Controller\Controller` | Frontend base routing |
| `/{plugin}/{controller}/{action}` | `CMS\Controller\Controller` | Frontend action routing |

## Test Scenarios

### 1. Base Controller Functionality
```gherkin
Feature: Base Controller
  As a CMS system
  I want base controller functionality
  To provide consistent behavior across plugins

  Scenario: Controller Initialization
    Given a plugin controller extends CMS base controller
    When controller is instantiated
    Then base initialization should occur:
      | Event system loaded |
      | Authentication checked |
      | Permissions verified |
      | View variables set |
      | Helper registration |

  Scenario: Request Lifecycle
    Given incoming HTTP request
    When controller processes request
    Then lifecycle should execute:
      | beforeFilter events |
      | Action execution |
      | View rendering |
      | afterFilter events |
      | Response generation |

  Scenario: Error Handling
    Given controller encounters error
    When exception is thrown
    Then error should be handled:
      | Log error message |
      | Render error page |
      | Maintain user session |
      | Provide fallback content |

  Scenario: Plugin Communication
    Given multiple plugins active
    When controller needs cross-plugin data
    Then communication should work:
      | Event dispatching |
      | Plugin registry access |
      | Shared service location |
      | Data exchange protocols |

  Scenario: Authentication Integration
    Given user authentication required
    When controller action is accessed
    Then authentication should be enforced:
      | Login state verified |
      | Permissions checked |
      | Redirects to login if needed |
      | Session management |
```

### 2. Extended Form Helper
```gherkin
Feature: CMS Form Helper
  As a developer
  I want enhanced form functionality
  To create rich admin interfaces

  Scenario: Enhanced Form Controls
    Given CMS form helper is loaded
    When creating form elements
    Then enhanced controls should be available:
      | WYSIWYG textarea |
      | Date/time pickers |
      | File upload widgets |
      | Multi-select dropdowns |
      | Dynamic field groups |

  Scenario: Field Type Integration
    Given custom field types exist
    When rendering field form elements
    Then appropriate widgets should render:
      | Text field -> input[type=text] |
      | Number field -> input[type=number] |
      | Date field -> datetime picker |
      | File field -> upload widget |
      | Reference field -> autocomplete |

  Scenario: Validation Display
    Given form has validation errors
    When form is rendered
    Then errors should be displayed:
      | Field-specific error messages |
      | Error styling and highlighting |
      | Accessible error descriptions |
      | Form summary of errors |

  Scenario: Dynamic Form Behavior
    Given form supports dynamic fields
    When user interacts with form
    Then dynamic behavior should work:
      | Add/remove field groups |
      | Conditional field display |
      | AJAX form submission |
      | Progress saving (drafts) |

  Scenario: Security Features
    Given form processing security
    When form is submitted
    Then security should be enforced:
      | CSRF token validation |
      | Input sanitization |
      | XSS prevention |
      | Data type validation |
```

### 3. Extended HTML Helper
```gherkin
Feature: CMS HTML Helper
  As a developer
  I want enhanced HTML generation
  To create consistent markup

  Scenario: Enhanced Link Generation
    Given CMS HTML helper loaded
    When generating links
    Then enhanced features should work:
      | Language-aware URLs |
      | Permission-based links |
      | SEO-friendly URLs |
      | Cache-busting parameters |

  Scenario: Asset Management
    Given assets need to be included
    When rendering page
    Then asset management should work:
      | CSS aggregation |
      | JavaScript bundling |
      | Version-based cache busting |
      | CDN URL generation |
      | Minification support |

  Scenario: Responsive Elements
    Given mobile-responsive requirements
    When generating HTML elements
    Then responsive features should work:
      | Responsive images |
      | Mobile-optimized menus |
      | Touch-friendly buttons |
      | Breakpoint-aware content |

  Scenario: Accessibility Features
    Given accessibility requirements
    When generating markup
    Then accessibility should be enforced:
      | ARIA labels and roles |
      | Keyboard navigation |
      | Screen reader support |
      | Color contrast compliance |

  Scenario: SEO Optimization
    Given SEO requirements
    When generating page markup
    Then SEO features should work:
      | Meta tag generation |
      | Structured data markup |
      | Canonical URLs |
      | Open Graph tags |
```

### 4. Plugin Management System
```gherkin
Feature: Plugin Management
  As a CMS administrator
  I want to manage plugins
  To control system functionality

  Scenario: Plugin Discovery
    Given plugins are installed
    When system starts up
    Then plugin discovery should work:
      | Scan plugin directories |
      | Read plugin configurations |
      | Check dependencies |
      | Load plugin metadata |

  Scenario: Plugin Loading
    Given enabled plugins exist
    When application initializes
    Then plugins should load:
      | In dependency order |
      | Register routes |
      | Initialize services |
      | Load translations |
      | Register event listeners |

  Scenario: Plugin Dependencies
    Given plugin has dependencies
    When dependency is missing
    Then system should handle:
      | Prevent plugin loading |
      | Display error message |
      | Suggest dependency installation |
      | Maintain system stability |

  Scenario: Plugin Communication
    Given multiple plugins active
    When plugins need to communicate
    Then communication should work:
      | Event dispatching |
      | Service sharing |
      | Data exchange |
      | Hook system |

  Scenario: Plugin Hot-swapping
    Given plugin needs update
    When plugin is disabled/enabled
    Then hot-swapping should work:
      | Graceful shutdown |
      | Route re-registration |
      | Cache invalidation |
      | Database migrations |
```

### 5. Event System Integration
```gherkin
Feature: Event System
  As a plugin developer
  I want event-driven architecture
  To extend functionality

  Scenario: Event Registration
    Given plugin needs to listen for events
    When plugin initializes
    Then event listeners should register:
      | Core system events |
      | Plugin-specific events |
      | Custom event handlers |
      | Priority-based ordering |

  Scenario: Event Dispatching
    Given action occurs in system
    When event is triggered
    Then event should be dispatched:
      | To all registered listeners |
      | In priority order |
      | With event data payload |
      | Allow event modification |

  Scenario: Event Cancellation
    Given event listener needs to cancel action
    When event is processed
    Then cancellation should work:
      | Stop event propagation |
      | Prevent default action |
      | Return error status |
      | Log cancellation reason |

  Scenario: Event Performance
    Given many event listeners active
    When events are frequently triggered
    Then performance should be maintained:
      | Efficient listener lookup |
      | Minimal overhead |
      | Event caching |
      | Lazy loading of handlers |

  Scenario: Event Debugging
    Given event system debugging needed
    When debug mode is enabled
    Then debugging should be available:
      | Event trace logging |
      | Listener registration info |
      | Event timing metrics |
      | Event data inspection |
```

## API Testing

### Plugin Management API
```javascript
// GET /api/cms/plugins
// Response: List of installed plugins with status

{
  "plugins": [
    {
      "name": "Content",
      "version": "2.0.0",
      "status": "enabled",
      "dependencies": ["CMS", "Field"],
      "routes": 15,
      "tables": ["contents", "content_types"]
    }
  ]
}

// GET /api/cms/system/info
// Response: System information

{
  "cms_version": "2.0.0",
  "php_version": "8.2.0",
  "database": "MySQL 8.0",
  "cache_status": "enabled",
  "debug_mode": false
}

// POST /api/cms/events/dispatch
{
  "event_name": "content.afterSave",
  "data": {"content_id": 123}
}
// Response: Event dispatch result

// GET /api/cms/routes
// Response: All registered routes in system
```

### Helper API Testing
```javascript
// Form Helper Methods
formHelper.input('title', {
  'type': 'text',
  'required': true,
  'class': 'form-control'
});

// HTML Helper Methods  
htmlHelper.link('Home', '/', {
  'class': 'nav-link',
  'lang': 'en'
});

// Asset Helper Methods
htmlHelper.css(['bootstrap.css', 'custom.css'], {
  'block': 'css',
  'minify': true
});
```

## Performance Requirements

### System Performance
- Plugin loading: < 100ms for all plugins
- Event dispatching: < 10ms per event
- Helper rendering: < 5ms per helper call
- Route resolution: < 20ms
- Base controller initialization: < 50ms

### Memory Usage
- Base controller overhead: < 5MB
- Plugin registry: < 10MB
- Event system: < 2MB
- Helper system: < 3MB

### Caching Strategy
- Plugin metadata: Permanent cache until change
- Route definitions: Bootstrap cache
- Event listeners: Memory cache
- Helper output: Context-dependent cache

## Edge Cases & Validation

### Plugin System
- [ ] Circular plugin dependencies
- [ ] Missing plugin files
- [ ] Corrupted plugin metadata
- [ ] Version compatibility conflicts
- [ ] Plugin loading failures
- [ ] Database migration errors

### Controller System
- [ ] Multiple inheritance conflicts
- [ ] Memory leaks in long-running processes
- [ ] Session management failures
- [ ] Authentication state corruption
- [ ] Request processing timeouts

### Helper System
- [ ] Helper method conflicts
- [ ] Invalid helper configurations
- [ ] HTML injection vulnerabilities
- [ ] Performance with large datasets
- [ ] Memory usage with complex forms

## Security Testing

### Core Security
- [ ] Base controller access control
- [ ] Event system injection prevention
- [ ] Helper output sanitization
- [ ] Plugin isolation
- [ ] File system protection

### Authentication & Authorization
- [ ] Session hijacking prevention
- [ ] CSRF token integration
- [ ] Permission inheritance
- [ ] Role-based access control
- [ ] API authentication

### Input Validation
- [ ] Form helper validation
- [ ] Data sanitization
- [ ] SQL injection prevention
- [ ] XSS protection
- [ ] File upload security

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] Controller structure changes
- [ ] Helper method compatibility
- [ ] Event system migration
- [ ] Plugin loading mechanism
- [ ] Route configuration updates
- [ ] Namespace changes
- [ ] Dependency injection updates

### Breaking Changes Validation
- [ ] Base class method signatures
- [ ] Helper API changes
- [ ] Event naming conventions
- [ ] Plugin hook system
- [ ] Configuration format changes

## Test Data Requirements

### Plugin Configuration
```json
{
  "plugins": {
    "CMS": {
      "version": "2.0.0",
      "enabled": true,
      "dependencies": []
    },
    "Content": {
      "version": "2.0.0", 
      "enabled": true,
      "dependencies": ["CMS", "Field"]
    }
  }
}
```

### Event Listeners
```php
// Sample event configuration
$events = [
    'Model.afterSave' => ['Plugin.Listener.afterSave'],
    'Controller.beforeRender' => ['Plugin.Listener.beforeRender'],
    'Custom.event' => ['Plugin.Listener.customHandler']
];
```

### Helper Test Data
```php
// Form test data
$formData = [
    'title' => 'Test Title',
    'body' => 'Test content',
    'status' => 'published'
];

// HTML test data
$linkData = [
    'url' => '/test/page',
    'title' => 'Test Link',
    'options' => ['class' => 'btn btn-primary']
];
```

## Success Metrics

### Functional Success
- All base controller methods working
- Plugin loading system operational
- Event system functioning correctly
- Helper extensions working properly
- Route registration successful
- Authentication integration working

### Performance Success
- Base system overhead < 10% total
- Plugin loading within time limits
- Event dispatching efficient
- Helper rendering fast
- Memory usage controlled

### Security Success
- No security vulnerabilities
- Access control enforced
- Input validation working
- Output sanitization active
- CSRF protection enabled

### Migration Success
- All CakePHP 3 functionality preserved
- No breaking changes introduced
- Plugin compatibility maintained
- Performance equal or better
- Zero data loss
- Full backward compatibility

## Integration Testing

### Cross-Plugin Testing
- CMS + Content plugin integration
- CMS + User authentication integration
- CMS + Field system integration
- Event communication between plugins

### Full System Testing
- Complete request/response cycle
- Multi-plugin workflow testing
- Performance under load
- Security penetration testing
- Browser compatibility testing