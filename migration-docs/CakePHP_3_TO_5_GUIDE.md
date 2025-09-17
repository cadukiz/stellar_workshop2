# CakePHP 3 to 5 Migration Guide for Junior Developers

## 🎯 Migration Strategy Decision

### Should we migrate 3→5 directly or 3→4→5?

**RECOMMENDATION: Direct 3→5 Migration**

#### Why Direct Migration is Better:
1. **Less Work**: Single migration effort instead of two separate migrations
2. **Cleaner Codebase**: Avoid temporary workarounds needed for CakePHP 4 compatibility
3. **Modern Standards**: Jump directly to PHP 8.2+ features and latest practices
4. **Time Efficient**: Estimated 40% less time than stepwise migration
5. **Testing Once**: Single comprehensive testing phase instead of two

#### When Stepwise (3→4→5) Might Be Better:
- If you have a massive codebase (>100K lines)
- If you need to maintain production during migration
- If you have limited testing resources
- If team is unfamiliar with modern PHP features

**For QuickAppsCMS: Direct 3→5 is recommended** due to the plugin-based architecture and extensive custom patterns.

---

## 📋 Pre-Migration Checklist

### Environment Requirements

#### ❌ CakePHP 3.3 (Current)
- PHP 5.6-7.4
- MySQL 5.5+
- ext-mcrypt (deprecated)
- ext-intl

#### ✅ CakePHP 5.0 (Target)
- PHP 8.1+ (8.2 recommended)
- MySQL 5.7+ (8.0 recommended)
- ext-mbstring
- ext-intl
- ext-simplexml

### Preparation Steps
```bash
# 1. Backup everything
cp -r quickapps-cakephp3/ quickapps-backup/
docker exec quickapps-db mysqldump -u quickapps -pquickapps123 quickapps > backup.sql

# 2. Create migration branch
git checkout -b migration-cake5
git add . && git commit -m "Pre-migration snapshot"

# 3. Update composer.json for CakePHP 5
cd quickapps-cakephp5/src
composer require cakephp/cakephp:"^5.0"
composer require cakephp/migrations:"^4.0"
composer require cakephp/debug_kit:"^5.0"
```

---

## 🔧 Core Framework Changes

### 1. Namespace and Autoloading

#### CakePHP 3
```php
// App namespace
namespace App\Controller;

// Plugin namespace
namespace QuickApps\Controller;

// Use statements
use Cake\Controller\Controller;
```

#### CakePHP 5
```php
// Same namespace structure but stricter PSR-4
namespace App\Controller;

// Plugins must declare in composer.json
"autoload": {
    "psr-4": {
        "QuickApps\\": "plugins/QuickApps/src/"
    }
}
```

### 2. Configuration System

#### CakePHP 3
```php
// config/app.php
return [
    'debug' => filter_var(env('DEBUG', true), FILTER_VALIDATE_BOOLEAN),
    'App' => [
        'namespace' => 'App',
        'encoding' => env('APP_ENCODING', 'UTF-8'),
    ]
];

// Reading config
Configure::read('App.namespace');
```

#### CakePHP 5
```php
// config/app.php remains similar but uses app_local.php for environment-specific
// .env file support is built-in
APP_NAME="QuickApps CMS"
DEBUG=true
APP_ENCODING=UTF-8

// Reading config (same)
Configure::read('App.namespace');

// New: Dependency Injection Container
// config/services.php
use Cake\Core\ServiceConfig;

return function (ServiceConfig $services) {
    $services->add(MyService::class);
};
```

### 3. Database Layer

#### CakePHP 3
```php
// Entity
class User extends Entity {
    protected $_accessible = [
        '*' => true,
        'id' => false
    ];
}

// Table
class UsersTable extends Table {
    public function initialize(array $config) {
        parent::initialize($config);
        $this->table('users');
        $this->primaryKey('id');
    }
}

// Query
$users = $this->Users->find()
    ->where(['active' => 1])
    ->contain(['Roles'])
    ->all();
```

#### CakePHP 5
```php
// Entity - Add type declarations
class User extends Entity {
    protected array $_accessible = [
        '*' => true,
        'id' => false
    ];
}

// Table - Method signature changes
class UsersTable extends Table {
    public function initialize(array $config): void {
        parent::initialize($config);
        $this->setTable('users');  // setTable instead of table
        $this->setPrimaryKey('id'); // setPrimaryKey instead of primaryKey
    }
}

// Query - Return type hints
public function findActive(Query $query, array $options): Query {
    return $query->where(['active' => 1]);
}

// New: Native enum support
enum UserStatus: string {
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}
```

### 4. Controllers

#### CakePHP 3
```php
class UsersController extends AppController {

    public function initialize() {
        parent::initialize();
        $this->loadComponent('RequestHandler');
        $this->loadComponent('Flash');
    }

    public function index() {
        $users = $this->paginate($this->Users);
        $this->set(compact('users'));
    }

    public function beforeFilter(Event $event) {
        parent::beforeFilter($event);
        $this->Auth->allow(['view', 'index']);
    }
}
```

#### CakePHP 5
```php
use Cake\Http\Response;
use Cake\Http\ServerRequest;

class UsersController extends AppController {

    public function initialize(): void {
        parent::initialize();
        $this->loadComponent('RequestHandler');
        $this->loadComponent('Flash');
    }

    public function index(): ?Response {
        $users = $this->paginate($this->Users);
        $this->set(compact('users'));
        return null; // or return $this->render();
    }

    public function beforeFilter(EventInterface $event): ?Response {
        parent::beforeFilter($event);
        // Auth component replaced by Authentication/Authorization plugins
        $this->Authentication->allowUnauthenticated(['view', 'index']);
        return null;
    }
}
```

### 5. Authentication System (CRITICAL CHANGE)

#### CakePHP 3 - AuthComponent
```php
// AppController
public function initialize() {
    $this->loadComponent('Auth', [
        'authenticate' => [
            'Form' => [
                'fields' => ['username' => 'email', 'password' => 'password']
            ]
        ],
        'loginRedirect' => ['controller' => 'Dashboard', 'action' => 'index'],
        'logoutRedirect' => ['controller' => 'Users', 'action' => 'login']
    ]);
}

// Login action
public function login() {
    if ($this->request->is('post')) {
        $user = $this->Auth->identify();
        if ($user) {
            $this->Auth->setUser($user);
            return $this->redirect($this->Auth->redirectUrl());
        }
    }
}
```

#### CakePHP 5 - Authentication/Authorization Plugins
```bash
# Install new auth system
composer require cakephp/authentication:"^3.0"
composer require cakephp/authorization:"^3.0"
```

```php
// src/Application.php
use Authentication\AuthenticationService;
use Authentication\AuthenticationServiceInterface;
use Authentication\AuthenticationServiceProviderInterface;
use Authentication\Middleware\AuthenticationMiddleware;
use Authorization\AuthorizationService;
use Authorization\AuthorizationServiceInterface;
use Authorization\AuthorizationServiceProviderInterface;
use Authorization\Middleware\AuthorizationMiddleware;

class Application extends BaseApplication implements
    AuthenticationServiceProviderInterface,
    AuthorizationServiceProviderInterface {

    public function middleware(MiddlewareQueue $middlewareQueue): MiddlewareQueue {
        $middlewareQueue
            ->add(new ErrorHandlerMiddleware())
            ->add(new AssetMiddleware())
            ->add(new RoutingMiddleware($this))
            ->add(new AuthenticationMiddleware($this))
            ->add(new AuthorizationMiddleware($this));

        return $middlewareQueue;
    }

    public function getAuthenticationService(ServerRequestInterface $request): AuthenticationServiceInterface {
        $service = new AuthenticationService([
            'unauthenticatedRedirect' => '/users/login',
            'queryParam' => 'redirect',
        ]);

        $service->loadIdentifier('Authentication.Password', [
            'fields' => ['username' => 'email']
        ]);

        $service->loadAuthenticator('Authentication.Session');
        $service->loadAuthenticator('Authentication.Form', [
            'fields' => ['username' => 'email', 'password' => 'password'],
            'loginUrl' => '/users/login'
        ]);

        return $service;
    }

    public function getAuthorizationService(ServerRequestInterface $request): AuthorizationServiceInterface {
        return new AuthorizationService();
    }
}

// Controller login
public function login(): ?Response {
    $this->request->allowMethod(['get', 'post']);
    $result = $this->Authentication->getResult();

    if ($result->isValid()) {
        $redirect = $this->request->getQuery('redirect', [
            'controller' => 'Dashboard',
            'action' => 'index',
        ]);
        return $this->redirect($redirect);
    }

    if ($this->request->is('post') && !$result->isValid()) {
        $this->Flash->error(__('Invalid email or password'));
    }
    return null;
}
```

### 6. Routing

#### CakePHP 3
```php
// config/routes.php
use Cake\Core\Plugin;
use Cake\Routing\RouteBuilder;
use Cake\Routing\Router;

Router::defaultRouteClass('DashedRoute');

Router::scope('/', function (RouteBuilder $routes) {
    $routes->connect('/', ['controller' => 'Pages', 'action' => 'display', 'home']);
    $routes->connect('/pages/*', ['controller' => 'Pages', 'action' => 'display']);

    $routes->fallbacks('DashedRoute');
});

Plugin::routes();
```

#### CakePHP 5
```php
// config/routes.php
use Cake\Routing\RouteBuilder;

return static function (RouteBuilder $routes) {
    $routes->setRouteClass(DashedRoute::class);

    $routes->scope('/', function (RouteBuilder $builder) {
        $builder->connect('/', ['controller' => 'Pages', 'action' => 'display', 'home']);
        $builder->connect('/pages/*', ['controller' => 'Pages', 'action' => 'display']);

        $builder->fallbacks();
    });

    // Plugins loaded differently
    $routes->scope('/admin', function (RouteBuilder $builder) {
        $builder->prefix('Admin', function (RouteBuilder $routes) {
            $routes->fallbacks(DashedRoute::class);
        });
    });
};
```

### 7. Forms and Validation

#### CakePHP 3
```php
// Template
<?= $this->Form->create($user) ?>
<?= $this->Form->control('email') ?>
<?= $this->Form->control('password') ?>
<?= $this->Form->button(__('Submit')) ?>
<?= $this->Form->end() ?>

// Validation
public function validationDefault(Validator $validator) {
    $validator
        ->integer('id')
        ->allowEmpty('id', 'create');

    $validator
        ->email('email')
        ->requirePresence('email', 'create')
        ->notEmpty('email');

    return $validator;
}
```

#### CakePHP 5
```php
// Template (similar but stricter types)
<?= $this->Form->create($user) ?>
<?= $this->Form->control('email', ['type' => 'email']) ?>
<?= $this->Form->control('password', ['type' => 'password']) ?>
<?= $this->Form->button(__('Submit')) ?>
<?= $this->Form->end() ?>

// Validation with type hints
public function validationDefault(Validator $validator): Validator {
    $validator
        ->integer('id')
        ->allowEmptyString('id', null, 'create'); // allowEmptyString instead of allowEmpty

    $validator
        ->email('email')
        ->requirePresence('email', 'create')
        ->notEmptyString('email'); // notEmptyString instead of notEmpty

    return $validator;
}
```

### 8. Components

#### CakePHP 3
```php
namespace App\Controller\Component;

use Cake\Controller\Component;

class CustomComponent extends Component {

    public $components = ['Flash'];

    public function initialize(array $config) {
        parent::initialize($config);
    }

    public function doSomething() {
        $this->Flash->success('Done!');
    }
}
```

#### CakePHP 5
```php
namespace App\Controller\Component;

use Cake\Controller\Component;

class CustomComponent extends Component {

    protected array $components = ['Flash']; // property visibility

    public function initialize(array $config): void {
        parent::initialize($config);
    }

    public function doSomething(): void {
        $this->Flash->success('Done!');
    }
}
```

### 9. Helpers

#### CakePHP 3
```php
namespace App\View\Helper;

use Cake\View\Helper;

class CustomHelper extends Helper {

    public $helpers = ['Html', 'Form'];

    public function makeLink($title, $url) {
        return $this->Html->link($title, $url, ['class' => 'custom-link']);
    }
}
```

#### CakePHP 5
```php
namespace App\View\Helper;

use Cake\View\Helper;
use Cake\View\StringTemplateTrait;

class CustomHelper extends Helper {

    protected array $helpers = ['Html', 'Form'];

    public function makeLink(string $title, array|string $url): string {
        return $this->Html->link($title, $url, ['class' => 'custom-link']);
    }
}
```

### 10. Shell to Command Migration

#### CakePHP 3 - Shell
```php
namespace App\Shell;

use Cake\Console\Shell;

class CustomShell extends Shell {

    public function main() {
        $this->out('Hello world.');
    }

    public function import() {
        $this->out('Importing...');
        // Import logic
    }

    public function getOptionParser() {
        $parser = parent::getOptionParser();
        $parser->addSubcommand('import', [
            'help' => 'Import data'
        ]);
        return $parser;
    }
}

// Run: bin/cake custom
// Run: bin/cake custom import
```

#### CakePHP 5 - Command
```php
namespace App\Command;

use Cake\Command\Command;
use Cake\Console\Arguments;
use Cake\Console\ConsoleIo;
use Cake\Console\ConsoleOptionParser;

class CustomCommand extends Command {

    public function execute(Arguments $args, ConsoleIo $io): int {
        $io->out('Hello world.');
        return static::CODE_SUCCESS;
    }

    protected function buildOptionParser(ConsoleOptionParser $parser): ConsoleOptionParser {
        $parser
            ->setDescription('Custom command')
            ->addArgument('name', [
                'help' => 'Name to display',
                'required' => false
            ]);
        return $parser;
    }
}

// Run: bin/cake custom
```

---

## 🔌 Plugin System Changes

### Plugin Loading

#### CakePHP 3
```php
// config/bootstrap.php
Plugin::load('QuickApps', ['bootstrap' => true, 'routes' => true]);
Plugin::load('DebugKit', ['bootstrap' => true]);

// Or load all plugins
Plugin::loadAll([
    'DebugKit' => ['bootstrap' => true],
    'Migrations' => ['bootstrap' => false],
]);
```

#### CakePHP 5
```php
// src/Application.php
public function bootstrap(): void {
    parent::bootstrap();

    $this->addPlugin('QuickApps');
    $this->addPlugin('DebugKit');

    // Or with configuration
    $this->addPlugin('QuickApps', [
        'bootstrap' => true,
        'routes' => true,
        'ignoreMissing' => true
    ]);
}

// Plugin class required: plugins/QuickApps/src/Plugin.php
namespace QuickApps;

use Cake\Core\BasePlugin;
use Cake\Core\PluginApplicationInterface;

class Plugin extends BasePlugin {

    public function bootstrap(PluginApplicationInterface $app): void {
        parent::bootstrap($app);
        // Plugin initialization
    }

    public function routes(RouteBuilder $routes): void {
        parent::routes($routes);
        // Plugin routes
    }
}
```

---

## 🎨 View Layer Changes

### Templates File Extensions

#### CakePHP 3
```
src/Template/Users/index.ctp
src/Template/Layout/default.ctp
src/Template/Element/header.ctp
```

#### CakePHP 5
```
templates/Users/index.php
templates/layout/default.php
templates/element/header.php
```

### Template Syntax

#### CakePHP 3
```php
// Layout
<?= $this->fetch('content') ?>
<?= $this->fetch('script') ?>

// View blocks
<?php $this->start('sidebar'); ?>
    <div>Sidebar content</div>
<?php $this->end(); ?>

// Extending
<?php $this->extend('/Common/view'); ?>
```

#### CakePHP 5
```php
// Same syntax but type safety
<?= $this->fetch('content') ?>
<?= $this->fetch('script') ?>

// View blocks (same)
<?php $this->start('sidebar'); ?>
    <div>Sidebar content</div>
<?php $this->end(); ?>

// Extending (same)
<?php $this->extend('/Common/view'); ?>

// New: Components support
<?= $this->element('MyComponent', [
    'data' => $data,
    'cache' => ['key' => 'unique_key']
]) ?>
```

---

## 🔒 Security Changes

### CSRF Protection

#### CakePHP 3
```php
// AppController
public function initialize() {
    $this->loadComponent('Csrf');
}

// Disable for specific actions
public function beforeFilter(Event $event) {
    $this->eventManager()->off($this->Csrf);
}
```

#### CakePHP 5
```php
// Application.php - Middleware based
public function middleware(MiddlewareQueue $middlewareQueue): MiddlewareQueue {
    $middlewareQueue
        ->add(new CsrfProtectionMiddleware([
            'httponly' => true,
        ]));

    return $middlewareQueue;
}

// Disable for specific routes
$routes->registerMiddleware('csrf', new CsrfProtectionMiddleware());
$routes->applyMiddleware('csrf');

// Skip CSRF for API routes
$routes->scope('/api', function (RouteBuilder $builder) {
    // No CSRF middleware applied here
    $builder->fallbacks();
});
```

### Security Component

#### CakePHP 3
```php
$this->loadComponent('Security');
$this->Security->config('unlockedFields', ['price']);
```

#### CakePHP 5
```php
// FormProtection component replaces Security component
$this->loadComponent('FormProtection');

// In view
$this->Form->unlockField('price');
```

---

## 📦 QuickAppsCMS Specific Migration

### 1. EAV Model Pattern
```php
// CakePHP 3 - QuickApps EAV
$this->Content->addBehavior('Eav.Eav');
$entity->set('custom_field', $value); // Magic setter

// CakePHP 5 - Needs refactoring
// Option 1: Use JSON columns
$entity->set('custom_fields', json_encode(['field' => $value]));

// Option 2: Implement custom EAV behavior with type hints
class EavBehavior extends Behavior {
    public function beforeSave(EventInterface $event, EntityInterface $entity, ArrayObject $options): void {
        // Handle EAV fields
    }
}
```

### 2. Plugin Dependencies
```php
// Update all plugin dependencies in composer.json
{
    "require": {
        "quickapps-plugins/cms": "dev-cake5",
        "quickapps-plugins/user": "dev-cake5",
        "quickapps-plugins/content": "dev-cake5"
    }
}
```

### 3. Three-Tier Template System
```php
// CakePHP 3 structure
src/Template/Back/Users/index.ctp
src/Template/Front/Users/view.ctp

// CakePHP 5 structure
templates/Back/Users/index.php
templates/Front/Users/view.php

// Update ViewBuilder
$this->viewBuilder()
    ->setTheme('QuickApps')
    ->setLayout('back')
    ->setTemplate('Back/Users/index');
```

### 4. Dynamic Routing
```php
// CakePHP 3 - Dynamic routes
Router::scope('/', function ($routes) {
    $routes->connect('/:slug',
        ['controller' => 'Content', 'action' => 'view'],
        ['pass' => ['slug'], 'routeClass' => 'QuickApps\Routing\Route\SlugRoute']
    );
});

// CakePHP 5 - Update route class
use Cake\Routing\Route\Route;

class SlugRoute extends Route {
    public function match(array $url, array $context = []): ?string {
        // Update method signature
        return parent::match($url, $context);
    }
}
```

---

## 🧪 Testing Migration

### PHPUnit Version

#### CakePHP 3
```json
"require-dev": {
    "phpunit/phpunit": "^5.7|^6.0"
}
```

#### CakePHP 5
```json
"require-dev": {
    "phpunit/phpunit": "^10.0"
}
```

### Test Case Changes

#### CakePHP 3
```php
use Cake\TestSuite\TestCase;

class UserTest extends TestCase {

    public $fixtures = ['app.users'];

    public function setUp() {
        parent::setUp();
        $this->Users = TableRegistry::get('Users');
    }

    public function testFindActive() {
        $query = $this->Users->find('active');
        $this->assertInstanceOf('Cake\ORM\Query', $query);
    }
}
```

#### CakePHP 5
```php
use Cake\TestSuite\TestCase;

class UserTest extends TestCase {

    protected array $fixtures = ['app.Users']; // Capital letter

    public function setUp(): void {
        parent::setUp();
        $this->Users = $this->fetchTable('Users');
    }

    public function testFindActive(): void {
        $query = $this->Users->find('active');
        $this->assertInstanceOf(Query::class, $query);
    }
}
```

---

## 📝 Migration Checklist Summary

### Phase 1: Environment Setup
- [ ] Install PHP 8.2
- [ ] Update Docker containers
- [ ] Backup database and code
- [ ] Create new CakePHP 5 project structure
- [ ] Update composer.json dependencies

### Phase 2: Core Migration
- [ ] Update namespace declarations
- [ ] Add type hints to all methods
- [ ] Convert .ctp templates to .php
- [ ] Move Template/ to templates/
- [ ] Update Table class methods (setTable, setPrimaryKey)
- [ ] Replace AuthComponent with Authentication plugin

### Phase 3: Plugin Migration
- [ ] Create Plugin.php for each plugin
- [ ] Update plugin loading in Application.php
- [ ] Migrate plugin routes
- [ ] Update plugin templates
- [ ] Fix plugin dependencies

### Phase 4: Custom Pattern Migration
- [ ] Refactor EAV implementation
- [ ] Update dynamic routing
- [ ] Migrate three-tier template system
- [ ] Update field handler system
- [ ] Replace AOP implementation

### Phase 5: Testing & Validation
- [ ] Update test fixtures
- [ ] Fix test method signatures
- [ ] Run PHPStan analysis
- [ ] Check coding standards
- [ ] Performance testing
- [ ] Security audit

---

## 🚨 Common Pitfalls & Solutions

### 1. "Call to undefined method"
**Cause**: Method names changed in CakePHP 5
**Solution**: Check migration guide for new method names
```php
// Old
$this->primaryKey('id');
// New
$this->setPrimaryKey('id');
```

### 2. "Return type declaration must be compatible"
**Cause**: Missing or wrong return types
**Solution**: Add proper type hints
```php
// Add return types
public function index(): ?Response {
    return null;
}
```

### 3. "Class not found"
**Cause**: Namespace or autoloading issues
**Solution**: Check PSR-4 compliance and update composer autoload
```bash
composer dump-autoload
```

### 4. "Authentication component not found"
**Cause**: AuthComponent removed in CakePHP 5
**Solution**: Install and configure new Authentication plugin

### 5. "Template not found"
**Cause**: Wrong file extension or location
**Solution**: Rename .ctp to .php and move to templates/

---

## 📚 Resources

### Official Documentation
- [CakePHP 5 Migration Guide](https://book.cakephp.org/5/en/appendices/migration-guide.html)
- [CakePHP 5 Cookbook](https://book.cakephp.org/5/en/index.html)
- [Authentication Plugin](https://book.cakephp.org/authentication/3/en/index.html)
- [Authorization Plugin](https://book.cakephp.org/authorization/3/en/index.html)

### Tools
- [Rector for CakePHP](https://github.com/cakephp/upgrade) - Automated code upgrades
- [PHP CodeSniffer](https://github.com/cakephp/cakephp-codesniffer) - Check coding standards
- [PHPStan](https://github.com/phpstan/phpstan) - Static analysis

### Commands for Migration
```bash
# Use upgrade tool for automated fixes
composer require --dev cakephp/upgrade
vendor/bin/upgrade rector --rules cakephp50 /path/to/app

# Check code standards
vendor/bin/phpcs --standard=CakePHP src/

# Run static analysis
vendor/bin/phpstan analyse src/

# Run tests
vendor/bin/phpunit
```

---

## 🎓 Tips for Junior Developers

1. **Start Small**: Migrate one controller at a time
2. **Test Often**: Run tests after each component migration
3. **Use IDE Support**: Configure PHPStorm/VSCode with CakePHP 5 support
4. **Read Errors Carefully**: PHP 8 has better error messages
5. **Ask for Help**: CakePHP community is helpful
6. **Keep Old Code**: Reference CakePHP 3 code during migration
7. **Document Changes**: Keep notes of what you changed and why
8. **Use Version Control**: Commit working changes frequently
9. **Pair Program**: Work with senior developers on complex parts
10. **Learn PHP 8 Features**: Understand union types, attributes, enums

---

## 🔄 Migration Timeline Estimate

### Small Project (< 10K lines)
- Environment Setup: 1 day
- Core Migration: 3-5 days
- Plugin Migration: 2-3 days
- Testing & Fixes: 2-3 days
**Total: 8-12 days**

### Medium Project like QuickAppsCMS (10K-50K lines)
- Environment Setup: 2 days
- Core Migration: 1-2 weeks
- Plugin Migration: 2-3 weeks
- Custom Pattern Migration: 1-2 weeks
- Testing & Fixes: 1 week
**Total: 5-8 weeks**

### Large Project (> 50K lines)
- Environment Setup: 3-5 days
- Core Migration: 3-4 weeks
- Plugin Migration: 4-6 weeks
- Testing & Fixes: 2-3 weeks
**Total: 10-14 weeks**

---

## 🎯 Next Steps

1. **Read this guide completely** before starting
2. **Set up CakePHP 5 environment** using Docker
3. **Create migration branch** in git
4. **Start with simplest controller** (e.g., PagesController)
5. **Migrate one component at a time**
6. **Test each migration step**
7. **Document issues and solutions**
8. **Celebrate small wins!** 🎉

Remember: Migration is a marathon, not a sprint. Take breaks, ask questions, and learn as you go!