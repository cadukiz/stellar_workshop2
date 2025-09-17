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

### 2. Configuration System & Database Drivers

#### CakePHP 3
```php
// config/app.php
return [
    'debug' => filter_var(env('DEBUG', true), FILTER_VALIDATE_BOOLEAN),
    'App' => [
        'namespace' => 'App',
        'encoding' => env('APP_ENCODING', 'UTF-8'),
    ],
    
    // Database configuration
    'Datasources' => [
        'default' => [
            'className' => 'Cake\Database\Connection',
            'driver' => 'Cake\Database\Driver\Mysql',
            'host' => 'localhost',
            'username' => 'my_app',
            'password' => 'secret',
            'database' => 'my_app',
            'encoding' => 'utf8',
            'timezone' => 'UTC',
            'flags' => [],
            'cacheMetadata' => true,
            'log' => false,
            'quoteIdentifiers' => false,
        ]
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
DATABASE_URL="mysql://user:pass@host/database"

// Reading config (same)
Configure::read('App.namespace');

// Database configuration with enhanced drivers
'Datasources' => [
    'default' => [
        'className' => 'Cake\Database\Connection',
        'driver' => 'Cake\Database\Driver\Mysql',
        'host' => env('DB_HOST', 'localhost'),
        'username' => env('DB_USERNAME', 'my_app'),
        'password' => env('DB_PASSWORD', 'secret'),
        'database' => env('DB_DATABASE', 'my_app'),
        'encoding' => 'utf8mb4', // Enhanced UTF-8 support
        'timezone' => 'UTC',
        'flags' => [
            // MySQL 8.0 compatibility flags
            PDO::MYSQL_ATTR_SSL_VERIFY_SERVER_CERT => false,
        ],
        'cacheMetadata' => true,
        'log' => env('DEBUG', false),
        'quoteIdentifiers' => false,
        
        // New: Connection retry and pooling
        'retry' => [
            'count' => 3,
            'delay' => 100, // milliseconds
        ],
        
        // New: Read/write splitting
        'replicas' => [
            'replica1' => [
                'host' => env('DB_READ_HOST', 'replica1.example.com'),
                'username' => env('DB_READ_USERNAME', 'reader'),
                'password' => env('DB_READ_PASSWORD', 'secret'),
            ]
        ],
        
        // Enhanced SSL configuration
        'ssl_key' => env('DB_SSL_KEY', null),
        'ssl_cert' => env('DB_SSL_CERT', null),
        'ssl_ca' => env('DB_SSL_CA', null),
    ]
],

// New: Dependency Injection Container
// config/services.php
use Cake\Core\ServiceConfig;

return function (ServiceConfig $services) {
    $services->add(MyService::class);
};
```

### Database Driver Differences

#### MySQL Driver Changes
```php
// CakePHP 3 - Basic MySQL configuration
'Datasources' => [
    'default' => [
        'driver' => 'Cake\Database\Driver\Mysql',
        'encoding' => 'utf8',
        'persistent' => false,
        'init' => ['SET sql_mode = "STRICT_TRANS_TABLES"']
    ]
]

// CakePHP 5 - Enhanced MySQL with better defaults
'Datasources' => [
    'default' => [
        'driver' => 'Cake\Database\Driver\Mysql',
        'encoding' => 'utf8mb4', // Better Unicode support
        'collation' => 'utf8mb4_unicode_ci',
        'persistent' => false,
        'init' => [
            'SET sql_mode = "ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"'
        ],
        
        // New: Native JSON column support
        'typeMap' => [
            'json' => 'Cake\Database\Type\JsonType',
            'uuid' => 'Cake\Database\Type\UuidType',
        ]
    ]
]
```

#### PostgreSQL Driver Changes
```php
// CakePHP 3 - PostgreSQL
'driver' => 'Cake\Database\Driver\Postgres',
'schema' => 'public'

// CakePHP 5 - Enhanced PostgreSQL
'driver' => 'Cake\Database\Driver\Postgres',
'schema' => env('DB_SCHEMA', 'public'),
// New: Array and JSON type support
'typeMap' => [
    'json' => 'Cake\Database\Type\JsonType',
    'jsonb' => 'Cake\Database\Type\JsonType',
    'uuid' => 'Cake\Database\Type\UuidType',
    'inet' => 'Cake\Database\Type\StringType',
]
```

#### SQLite Driver Changes
```php
// CakePHP 3 - SQLite
'driver' => 'Cake\Database\Driver\Sqlite',
'database' => ROOT . DS . 'tmp' . DS . 'debug_kit.sqlite'

// CakePHP 5 - Enhanced SQLite with WAL mode
'driver' => 'Cake\Database\Driver\Sqlite',
'database' => env('DB_FILE', ROOT . DS . 'tmp' . DS . 'app.sqlite'),
'init' => [
    'PRAGMA journal_mode=WAL', // Better concurrency
    'PRAGMA synchronous=NORMAL',
    'PRAGMA foreign_keys=ON'
]
```

### 3. Database Layer & ORM Changes

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

// Query - Basic find operations
$users = $this->Users->find()
    ->where(['active' => 1])
    ->contain(['Roles'])
    ->all();

// Custom finder methods
public function findActive($query, $options) {
    return $query->where(['active' => true]);
}

// Query result handling
$user = $this->Users->get($id);
$users = $this->Users->find('all')->toArray();

// Count queries
$count = $this->Users->find()->count();

// First/last records
$first = $this->Users->find()->first();
$last = $this->Users->find()->last();
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

// Query - Return type hints required
public function findActive(Query $query, array $options): Query {
    return $query->where(['active' => true]);
}

// Query result handling with types
$user = $this->Users->get($id); // Returns User entity or throws exception
$users = $this->Users->find()->all(); // Returns Collection, not array
$usersArray = $this->Users->find()->toArray(); // Explicit array conversion

// Count queries - return int
$count = $this->Users->find()->count(); // Returns int

// First/last records - nullable return
$first = $this->Users->find()->first(); // Returns User|null
$last = $this->Users->find()->last(); // Returns User|null

// New: Typed queries
$users = $this->Users->find()
    ->where(['active' => true])
    ->contain(['Roles'])
    ->orderByAsc('created')  // orderBy methods added
    ->orderByDesc('modified')
    ->all();

// New: Query expressions with types
use Cake\Database\Expression\QueryExpression;

$query = $this->Users->find()
    ->where(function (QueryExpression $exp) {
        return $exp->gte('age', 18);
    });

// New: Native enum support
enum UserStatus: string {
    case ACTIVE = 'active';
    case INACTIVE = 'inactive';
}

// Use enums in queries
$activeUsers = $this->Users->find()
    ->where(['status' => UserStatus::ACTIVE]);
```

### 4. Controllers & Request/Response Changes

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

    public function view($id) {
        // Request object methods
        if ($this->request->is('ajax')) {
            $this->layout = 'ajax';
        }
        
        // Get request data
        $data = $this->request->data;
        $query = $this->request->query;
        $params = $this->request->params;
        
        // Response methods
        $this->response->type('json');
        $this->response->body(json_encode($user));
        $this->response->statusCode(200);
        
        return $this->response;
    }

    public function add() {
        $user = $this->Users->newEntity();
        if ($this->request->is('post')) {
            $user = $this->Users->patchEntity($user, $this->request->data);
            if ($this->Users->save($user)) {
                $this->Flash->success('User saved');
                return $this->redirect(['action' => 'index']);
            }
        }
        $this->set(compact('user'));
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
use Cake\Event\EventInterface;

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

    public function view(string $id): Response {
        // Request object methods - same but with type safety
        $request = $this->getRequest();
        if ($request->is('ajax')) {
            $this->viewBuilder()->setLayout('ajax');
        }
        
        // Get request data - new methods
        $data = $request->getParsedBody(); // replaces ->data
        $query = $request->getQueryParams(); // replaces ->query
        $params = $request->getAttribute('params'); // replaces ->params
        $headers = $request->getHeaders(); // new method
        
        // Response methods - fluent interface
        return $this->response
            ->withType('json')
            ->withStringBody(json_encode($user))
            ->withStatus(200);
    }

    public function add(): ?Response {
        $user = $this->Users->newEmptyEntity();
        $request = $this->getRequest();
        
        if ($request->is('post')) {
            $user = $this->Users->patchEntity($user, $request->getParsedBody());
            if ($this->Users->save($user)) {
                $this->Flash->success('User saved');
                return $this->redirect(['action' => 'index']);
            }
        }
        $this->set(compact('user'));
        return null;
    }

    // New: File uploads handling
    public function upload(): Response {
        $request = $this->getRequest();
        $uploadedFile = $request->getUploadedFile('avatar');
        
        if ($uploadedFile && $uploadedFile->getError() === UPLOAD_ERR_OK) {
            $filename = $uploadedFile->getClientFilename();
            $uploadedFile->moveTo('/path/to/uploads/' . $filename);
        }
        
        return $this->response->withType('json')
            ->withStringBody(json_encode(['status' => 'success']));
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

### 7. Forms, Helpers & Validation Changes

#### CakePHP 3 - FormHelper & HtmlHelper
```php
// Basic form
<?= $this->Form->create($user) ?>
<?= $this->Form->control('email') ?>
<?= $this->Form->control('password') ?>
<?= $this->Form->button(__('Submit')) ?>
<?= $this->Form->end() ?>

// Form with options
<?= $this->Form->create($user, [
    'type' => 'file',
    'url' => ['action' => 'add']
]) ?>
<?= $this->Form->control('name', [
    'label' => 'Full Name',
    'required' => true
]) ?>
<?= $this->Form->control('avatar', ['type' => 'file']) ?>
<?= $this->Form->control('role_id', [
    'type' => 'select',
    'options' => $roles,
    'empty' => 'Select Role'
]) ?>

// HtmlHelper methods
<?= $this->Html->link('Edit', ['action' => 'edit', $user->id]) ?>
<?= $this->Html->image('logo.png', ['alt' => 'Logo']) ?>
<?= $this->Html->css('styles') ?>
<?= $this->Html->script('app') ?>
<?= $this->Html->meta('description', 'Page description') ?>

// Form helper methods
<?= $this->Form->input('title') ?> // Deprecated
<?= $this->Form->control('title') ?> // Preferred
<?= $this->Form->checkbox('active') ?>
<?= $this->Form->radio('gender', ['M' => 'Male', 'F' => 'Female']) ?>
<?= $this->Form->select('country', $countries) ?>

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

#### CakePHP 5 - Enhanced FormHelper & HtmlHelper
```php
// Basic form with type safety
<?= $this->Form->create($user) ?>
<?= $this->Form->control('email', ['type' => 'email']) ?>
<?= $this->Form->control('password', ['type' => 'password']) ?>
<?= $this->Form->button(__('Submit')) ?>
<?= $this->Form->end() ?>

// Form with enhanced options and attributes
<?= $this->Form->create($user, [
    'type' => 'file',
    'url' => ['action' => 'add'],
    'class' => 'needs-validation',
    'novalidate' => true
]) ?>
<?= $this->Form->control('name', [
    'label' => 'Full Name',
    'required' => true,
    'class' => 'form-control',
    'data-validate' => 'required'
]) ?>
<?= $this->Form->control('avatar', [
    'type' => 'file',
    'accept' => 'image/*'
]) ?>
<?= $this->Form->control('role_id', [
    'type' => 'select',
    'options' => $roles,
    'empty' => 'Select Role',
    'class' => 'form-select'
]) ?>

// New: Enhanced date/time controls
<?= $this->Form->control('birth_date', [
    'type' => 'date',
    'min' => '1900-01-01',
    'max' => date('Y-m-d')
]) ?>
<?= $this->Form->control('meeting_time', [
    'type' => 'datetime-local',
    'step' => 300 // 5 minute steps
]) ?>

// HtmlHelper with enhanced features
<?= $this->Html->link('Edit', ['action' => 'edit', $user->id], [
    'class' => 'btn btn-primary',
    'data-bs-toggle' => 'tooltip',
    'title' => 'Edit user details'
]) ?>
<?= $this->Html->image('logo.png', [
    'alt' => 'Logo',
    'loading' => 'lazy',
    'width' => 200,
    'height' => 50
]) ?>
<?= $this->Html->css('styles', ['defer' => true]) ?>
<?= $this->Html->script('app', ['async' => true]) ?>

// New: Enhanced meta methods
<?= $this->Html->meta([
    'name' => 'description',
    'content' => 'Page description'
]) ?>
<?= $this->Html->meta([
    'property' => 'og:title',
    'content' => 'Page title'
]) ?>

// Form helper improvements
<?= $this->Form->control('title', [
    'type' => 'text',
    'maxlength' => 255,
    'placeholder' => 'Enter title...'
]) ?>
<?= $this->Form->control('active', [
    'type' => 'checkbox',
    'hiddenField' => false // Don't create hidden field
]) ?>
<?= $this->Form->control('tags', [
    'type' => 'select',
    'multiple' => true,
    'options' => $tags
]) ?>

// New: Range and number inputs
<?= $this->Form->control('price', [
    'type' => 'number',
    'min' => 0,
    'step' => 0.01
]) ?>
<?= $this->Form->control('rating', [
    'type' => 'range',
    'min' => 1,
    'max' => 10,
    'value' => 5
]) ?>

// Validation with enhanced type hints
public function validationDefault(Validator $validator): Validator {
    $validator
        ->integer('id')
        ->allowEmptyString('id', null, 'create'); // allowEmptyString instead of allowEmpty

    $validator
        ->email('email')
        ->requirePresence('email', 'create')
        ->notEmptyString('email') // notEmptyString instead of notEmpty
        ->add('email', 'unique', ['rule' => 'validateUnique']);

    // New: Enhanced validation rules
    $validator
        ->scalar('name')
        ->maxLength('name', 255)
        ->requirePresence('name', 'create')
        ->notEmptyString('name')
        ->add('name', 'alphaNumeric', [
            'rule' => ['custom', '/^[a-zA-Z0-9\s]+$/'],
            'message' => 'Name can only contain letters, numbers and spaces'
        ]);

    $validator
        ->decimal('price', 2)
        ->greaterThanOrEqual('price', 0)
        ->requirePresence('price', 'create');

    return $validator;
}

// New: Custom validation methods with types
public function validationUpdate(Validator $validator): Validator {
    $validator = $this->validationDefault($validator);
    
    $validator->remove('email', 'unique');
    
    return $validator;
}
```

### Key FormHelper/HtmlHelper Changes

| Feature | CakePHP 3 | CakePHP 5 |
|---------|-----------|-----------|
| **Input method** | `Form->input()` (deprecated) | `Form->control()` only |
| **Date inputs** | Limited HTML5 support | Full HTML5 date/time types |
| **File uploads** | Basic file type | Enhanced with accept attribute |
| **Validation messages** | `notEmpty()` | `notEmptyString()` for strings |
| **Meta tags** | String-based | Array-based configuration |
| **Asset loading** | Basic defer | Async/defer support |
| **Form attributes** | Limited HTML5 | Full HTML5 form validation |

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

## 🔀 Middleware vs Dispatcher Filters

### CakePHP 3 - Dispatcher Filters

#### CakePHP 3
```php
// src/Filter/CustomFilter.php
namespace App\Filter;

use Cake\Event\Event;
use Cake\Routing\DispatcherFilter;

class CustomFilter extends DispatcherFilter {
    
    public function beforeDispatch(Event $event) {
        $request = $event->getData('request');
        $response = $event->getData('response');
        
        // Custom logic before controller
        if ($request->is('mobile')) {
            $request->setParam('prefix', 'mobile');
        }
        
        return true;
    }
    
    public function afterDispatch(Event $event) {
        $request = $event->getData('request');
        $response = $event->getData('response');
        
        // Custom logic after controller
        $response->header('X-Custom-Header', 'processed');
        
        return $response;
    }
}

// config/bootstrap.php
use Cake\Routing\DispatcherFactory;

DispatcherFactory::add('Custom', ['priority' => 10]);

// Built-in filters
DispatcherFactory::add('Asset');
DispatcherFactory::add('Routing');
DispatcherFactory::add('ControllerFactory');
```

#### CakePHP 5 - Middleware Stack
```php
// src/Middleware/CustomMiddleware.php
namespace App\Middleware;

use Cake\Http\MiddlewareInterface;
use Cake\Http\Response;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ServerRequestInterface;
use Psr\Http\Server\RequestHandlerInterface;

class CustomMiddleware implements MiddlewareInterface {
    
    public function process(
        ServerRequestInterface $request,
        RequestHandlerInterface $handler
    ): ResponseInterface {
        
        // Before controller (like beforeDispatch)
        if (strpos($request->getHeaderLine('User-Agent'), 'Mobile') !== false) {
            $request = $request->withAttribute('isMobile', true);
        }
        
        // Call next middleware/controller
        $response = $handler->handle($request);
        
        // After controller (like afterDispatch)
        return $response->withHeader('X-Custom-Header', 'processed');
    }
}

// src/Application.php
use App\Middleware\CustomMiddleware;
use Cake\Http\Middleware\CsrfProtectionMiddleware;
use Cake\Http\Middleware\EncryptedCookieMiddleware;
use Cake\Http\Middleware\ErrorHandlerMiddleware;
use Cake\Http\Middleware\SecurityHeadersMiddleware;
use Cake\Routing\Middleware\AssetMiddleware;
use Cake\Routing\Middleware\RoutingMiddleware;

public function middleware(MiddlewareQueue $middlewareQueue): MiddlewareQueue {
    $middlewareQueue
        // Error handling first
        ->add(new ErrorHandlerMiddleware(Configure::read('Error'), $this))
        
        // Asset serving
        ->add(new AssetMiddleware([
            'cacheTime' => Configure::read('Asset.cacheTime')
        ]))
        
        // Routing
        ->add(new RoutingMiddleware($this))
        
        // Security headers
        ->add(new SecurityHeadersMiddleware())
        
        // CSRF protection
        ->add(new CsrfProtectionMiddleware([
            'httponly' => true,
        ]))
        
        // Cookie encryption
        ->add(new EncryptedCookieMiddleware(
            ['session'],
            Configure::read('Security.cookieKey')
        ))
        
        // Custom middleware
        ->add(new CustomMiddleware())
        
        // Authentication
        ->add(new AuthenticationMiddleware($this));

    return $middlewareQueue;
}
```

### Migration Strategy: Filters → Middleware

#### 1. Asset Filter → AssetMiddleware
```php
// CakePHP 3
DispatcherFactory::add('Asset', [
    'paths' => [
        'plugin' => Configure::read('App.paths.plugins'),
        'theme' => Configure::read('App.paths.templates')
    ]
]);

// CakePHP 5
->add(new AssetMiddleware([
    'cacheTime' => '+1 year'
]))
```

#### 2. Custom Authentication Filter → Authentication Middleware
```php
// CakePHP 3 - Custom Auth Filter
class AuthFilter extends DispatcherFilter {
    public function beforeDispatch(Event $event) {
        $request = $event->getData('request');
        if (!$this->isAuthorized($request)) {
            throw new UnauthorizedException();
        }
    }
}

// CakePHP 5 - Authentication Middleware
use Authentication\Middleware\AuthenticationMiddleware;

->add(new AuthenticationMiddleware($this))
```

#### 3. Mobile Detection Filter → Custom Middleware
```php
// CakePHP 3
class MobileFilter extends DispatcherFilter {
    public function beforeDispatch(Event $event) {
        $request = $event->getData('request');
        if ($this->isMobile($request)) {
            $request->setParam('prefix', 'mobile');
        }
    }
}

// CakePHP 5
class MobileMiddleware implements MiddlewareInterface {
    public function process(ServerRequestInterface $request, RequestHandlerInterface $handler): ResponseInterface {
        if ($this->isMobile($request)) {
            $request = $request->withAttribute('prefix', 'mobile');
        }
        return $handler->handle($request);
    }
}
```

### Key Differences

| Aspect | CakePHP 3 Filters | CakePHP 5 Middleware |
|--------|-------------------|---------------------|
| **Order** | Priority-based | Sequential queue |
| **Interface** | DispatcherFilter | MiddlewareInterface |
| **Request/Response** | Event data | PSR-7 objects |
| **Execution** | beforeDispatch/afterDispatch | Single process method |
| **Error Handling** | Exception throwing | Response objects |
| **Testability** | Event system | Direct unit testing |

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