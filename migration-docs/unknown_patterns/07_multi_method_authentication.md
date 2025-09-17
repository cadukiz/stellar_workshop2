# Multi-Method Authentication System

## What problem was it solving?

Standard CakePHP authentication is session-based and simple. CMSs need:
- "Remember me" functionality with secure cookie authentication
- Token-based authentication for API access
- Anonymous user permissions
- Role-based access control with caching
- Multiple authentication methods tried in sequence

## How does it work?

**Location**: `/vendor/quickapps-plugins/user/src/Auth/AnonymousAuthenticate.php`

```php
class AnonymousAuthenticate extends BaseAuthenticate
{
    public function unauthenticated(Request $request, Response $response)
    {
        // Try cookie-based "remember me" authentication
        if ($this->_cookieLogin($request)) {
            return true;
        }
        
        // Try token-based authentication
        if ($this->_tokenLogin($request)) {
            return true;
        }
        
        // Fall back to anonymous permissions
        return $this->_anonymousPermissions($request);
    }
    
    protected function _cookieLogin(Request $request)
    {
        $cookie = $request->getCookie('User.Cookie');
        if ($cookie) {
            $cookie = json_decode($cookie, true);
            if (isset($cookie['user']) && 
                isset($cookie['hash']) &&
                $cookie['hash'] === Security::hash($cookie['user'], 'sha1', true)) {
                
                $user = $this->_findUser($cookie['user']['username']);
                if ($user) {
                    $this->getController()->Auth->setUser($user);
                    return true;
                }
            }
        }
        return false;
    }
    
    protected function _anonymousPermissions(Request $request)
    {
        // Cache anonymous permissions for performance
        $permissions = Cache::read('permissions_anonymous', 'permissions');
        if ($permissions === false) {
            $permissions = $this->_rolePermissions(ROLE_ID_ANONYMOUS);
            Cache::write('permissions_anonymous', $permissions, 'permissions');
        }
        
        $action = $this->_requestAction($request);
        return isset($permissions[$action]);
    }
    
    private function _rolePermissions($roleId)
    {
        // Load permissions for role from database
        return TableRegistry::get('Acos')
            ->find()
            ->matching('Permissions', function ($q) use ($roleId) {
                return $q->where(['Permissions.aro_id' => $roleId]);
            })
            ->extract('alias')
            ->toArray();
    }
}
```

## What would be the modern CakePHP 5 approach?

CakePHP 5 has a completely rewritten authentication system:

1. **Authentication Middleware**: Use dedicated middleware for each auth method
2. **Identity Objects**: Implement proper identity objects with permissions
3. **JWT Tokens**: Use standard JWT for token authentication
4. **Authorization Plugin**: Use the official authorization plugin

```php
// Modern CakePHP 5 approach
class MultiMethodAuthenticator implements AuthenticatorInterface
{
    private array $authenticators;
    private LoggerInterface $logger;
    
    public function authenticate(ServerRequestInterface $request): ?IdentityInterface
    {
        foreach ($this->authenticators as $authenticator) {
            try {
                $identity = $authenticator->authenticate($request);
                if ($identity) {
                    $this->logger->info("User authenticated via {$authenticator::class}");
                    return $identity;
                }
            } catch (AuthenticationException $e) {
                $this->logger->debug("Authentication failed for {$authenticator::class}: {$e->getMessage()}");
                continue;
            }
        }
        
        // Return anonymous identity with limited permissions
        return new AnonymousIdentity();
    }
}

class CookieAuthenticator implements AuthenticatorInterface
{
    private UserRepositoryInterface $users;
    private EncrypterInterface $encrypter;
    
    public function authenticate(ServerRequestInterface $request): ?IdentityInterface
    {
        $cookie = $request->getCookieParams()['remember_token'] ?? null;
        if (!$cookie) {
            return null;
        }
        
        try {
            $data = $this->encrypter->decrypt($cookie);
            $payload = json_decode($data, true);
            
            if (!$this->validateTokenStructure($payload)) {
                return null;
            }
            
            $user = $this->users->findByRememberToken($payload['token']);
            return $user ? new UserIdentity($user) : null;
            
        } catch (DecryptException $e) {
            return null;
        }
    }
}

class UserIdentity implements IdentityInterface
{
    private User $user;
    private array $permissions;
    
    public function can(string $action, mixed $resource = null): bool
    {
        // Check cached permissions
        return in_array($action, $this->getPermissions());
    }
    
    private function getPermissions(): array
    {
        if (!isset($this->permissions)) {
            $this->permissions = Cache::remember(
                "user_permissions_{$this->user->id}",
                fn() => $this->loadUserPermissions(),
                '1 hour'
            );
        }
        return $this->permissions;
    }
}
```

## What is the migration strategy from now to a modern solution?

1. **Phase 1**: Implement AuthenticatorInterface wrappers around existing auth classes
2. **Phase 2**: Replace custom cookie auth with secure JWT remember tokens
3. **Phase 3**: Migrate to CakePHP 5 Authentication and Authorization plugins
4. **Phase 4**: Implement proper identity objects with permission caching
5. **Phase 5**: Add API authentication with OAuth2/JWT support