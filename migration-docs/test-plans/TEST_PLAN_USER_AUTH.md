# Test Plan: User Authentication & Management

## Feature Overview
The User plugin provides complete authentication, authorization, and user management functionality for QuickAppsCMS.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/user/src/Controller/GatewayController.php` - Login/logout/profile
- `src/vendor/quickapps-plugins/user/src/Controller/Admin/GatewayController.php` - Admin authentication
- `src/vendor/quickapps-plugins/user/src/Controller/Admin/ManageController.php` - User CRUD operations
- `src/vendor/quickapps-plugins/user/src/Controller/Admin/RolesController.php` - Role management
- `src/vendor/quickapps-plugins/user/src/Controller/Admin/PermissionsController.php` - ACL permissions
- `src/vendor/quickapps-plugins/user/src/Controller/Admin/FieldsController.php` - User custom fields

### Models
- `src/vendor/quickapps-plugins/user/src/Model/Table/UsersTable.php`
- `src/vendor/quickapps-plugins/user/src/Model/Table/RolesTable.php`
- `src/vendor/quickapps-plugins/user/src/Model/Table/AcosTable.php`
- `src/vendor/quickapps-plugins/user/src/Model/Table/PermissionsTable.php`
- `src/vendor/quickapps-plugins/user/src/Model/Entity/User.php`
- `src/vendor/quickapps-plugins/user/src/Model/Entity/Role.php`

### Views
- `src/vendor/quickapps-plugins/user/src/Template/Gateway/` - Login/register forms
- `src/vendor/quickapps-plugins/user/src/Template/Admin/Manage/` - User management interface
- `src/vendor/quickapps-plugins/user/src/Template/Admin/Roles/` - Role management interface
- `src/vendor/quickapps-plugins/user/src/Template/Admin/Permissions/` - Permission matrix

### Database Tables
- `users` - User accounts
- `roles` - User roles
- `roles_users` - Role assignments
- `acos` - Access Control Objects
- `permissions` - ACL permissions matrix
- `users_roles` - User-role relationships

## Routes & URLs

### Public Routes (Frontend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Login | `/login` | `User\Controller\GatewayController::login()` | Public login page |
| Logout | `/logout` | `User\Controller\GatewayController::logout()` | Logout endpoint |
| Register | `/user/register` | `User\Controller\GatewayController::register()` | User registration |
| Profile | `/user/me` | `User\Controller\GatewayController::me()` | Current user profile |
| View Profile | `/user/profile/{id}` | `User\Controller\GatewayController::profile()` | View user profile |
| Forgot Password | `/user/forgot-password` | `User\Controller\GatewayController::forgotPassword()` | Password reset |
| Activate | `/user/activate/{token}` | `User\Controller\GatewayController::activate()` | Email activation |
| Unauthorized | `/unauthorized` | `User\Controller\GatewayController::unauthorized()` | Access denied page |

### Admin Routes (Backend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Admin Login | `/admin/login` | `User\Controller\Admin\GatewayController::login()` | Admin login |
| Admin Logout | `/admin/logout` | `User\Controller\Admin\GatewayController::logout()` | Admin logout |
| User List | `/admin/user/manage` | `User\Controller\Admin\ManageController::index()` | List all users |
| Add User | `/admin/user/manage/add` | `User\Controller\Admin\ManageController::add()` | Create new user |
| Edit User | `/admin/user/manage/edit/{id}` | `User\Controller\Admin\ManageController::edit()` | Edit user details |
| Delete User | `/admin/user/manage/delete/{id}` | `User\Controller\Admin\ManageController::delete()` | Delete user |
| Block User | `/admin/user/manage/block/{id}` | `User\Controller\Admin\ManageController::block()` | Block/unblock user |
| Role List | `/admin/user/roles` | `User\Controller\Admin\RolesController::index()` | List roles |
| Add Role | `/admin/user/roles/add` | `User\Controller\Admin\RolesController::add()` | Create role |
| Edit Role | `/admin/user/roles/edit/{id}` | `User\Controller\Admin\RolesController::edit()` | Edit role |
| Permissions | `/admin/user/permissions` | `User\Controller\Admin\PermissionsController::index()` | Permission matrix |
| User Fields | `/admin/user/fields` | `User\Controller\Admin\FieldsController::index()` | Custom user fields |

## Test Scenarios

### 1. User Registration Flow
```gherkin
Feature: User Registration
  As a visitor
  I want to register an account
  So that I can access member features

  Scenario: Successful Registration
    Given I am on "/user/register"
    When I fill in "Username" with "testuser"
    And I fill in "Email" with "test@example.com"
    And I fill in "Password" with "SecurePass123!"
    And I fill in "Confirm Password" with "SecurePass123!"
    And I click "Register"
    Then I should see "Registration successful"
    And an email should be sent to "test@example.com"
    And the database table "users" should contain "testuser"
    And the user status should be "pending"

  Scenario: Registration with Existing Email
    Given a user exists with email "existing@example.com"
    When I register with email "existing@example.com"
    Then I should see "Email already in use"
    And no new user should be created

  Scenario: Weak Password
    Given I am on "/user/register"
    When I fill in "Password" with "123"
    Then I should see "Password too weak"
```

### 2. Authentication Flow
```gherkin
Feature: User Authentication
  As a registered user
  I want to log in
  So that I can access my account

  Scenario: Successful Login
    Given I have an account "user@example.com" with password "Pass123!"
    When I visit "/login"
    And I fill in "Email" with "user@example.com"
    And I fill in "Password" with "Pass123!"
    And I click "Login"
    Then I should be redirected to "/"
    And I should see "Welcome back"
    And session should contain user data

  Scenario: Failed Login - Wrong Password
    Given I have an account "user@example.com"
    When I login with wrong password
    Then I should see "Invalid credentials"
    And I should remain on "/login"
    And no session should be created

  Scenario: Blocked User Login
    Given my account is blocked
    When I try to login
    Then I should see "Account blocked"
```

### 3. Admin User Management
```gherkin
Feature: Admin User Management
  As an administrator
  I want to manage users
  So that I can control system access

  Scenario: View User List
    Given I am logged in as admin
    When I visit "/admin/user/manage"
    Then I should see user list table with columns:
      | Username | Email | Roles | Status | Actions |
    And I should see pagination controls
    And I should see search filters

  Scenario: Create New User
    Given I am on "/admin/user/manage/add"
    When I fill in user details:
      | Field | Value |
      | Username | newuser |
      | Email | new@example.com |
      | Password | SecurePass123! |
      | Role | Editor |
    And I click "Save"
    Then user should be created in database
    And user should have role "Editor"

  Scenario: Edit User Details
    Given a user "testuser" exists
    When I visit "/admin/user/manage/edit/{id}"
    And I change email to "newemail@example.com"
    And I click "Save"
    Then user email should be updated
    And activity log should record the change

  Scenario: Block/Unblock User
    Given a user "testuser" is active
    When I click "Block" for this user
    Then user status should be "blocked"
    And user should not be able to login
```

### 4. Role & Permission Management
```gherkin
Feature: Role Management
  As an administrator
  I want to manage roles and permissions
  So that I can control access levels

  Scenario: Create New Role
    Given I am on "/admin/user/roles/add"
    When I fill in:
      | Field | Value |
      | Name | Content Editor |
      | Slug | content-editor |
    And I select permissions:
      | Create content |
      | Edit content |
      | Delete own content |
    And I click "Save"
    Then role should be created
    And permissions should be saved to "permissions" table

  Scenario: Edit Role Permissions
    Given a role "Editor" exists
    When I visit "/admin/user/permissions"
    And I check "Administer users" for "Editor" role
    And I click "Save Permissions"
    Then permission matrix should be updated
    And users with "Editor" role should have new permission

  Scenario: Delete Role
    Given a role "Temporary" exists with no users
    When I delete the role
    Then role should be removed from database
    And associated permissions should be deleted
```

### 5. Password Reset Flow
```gherkin
Feature: Password Reset
  As a user
  I want to reset my password
  When I forget it

  Scenario: Request Password Reset
    Given I have account "user@example.com"
    When I visit "/user/forgot-password"
    And I enter "user@example.com"
    And I click "Reset Password"
    Then reset token should be generated
    And email should be sent with reset link
    And token should expire in 24 hours

  Scenario: Reset Password with Valid Token
    Given I have a valid reset token "abc123"
    When I visit "/user/reset-password/abc123"
    And I enter new password "NewPass456!"
    And I confirm password "NewPass456!"
    And I click "Reset"
    Then password should be updated
    And token should be invalidated
    And I should be able to login with new password

  Scenario: Expired Token
    Given I have an expired reset token
    When I try to use it
    Then I should see "Token expired"
    And I should be redirected to forgot password page
```

### 6. Profile Management
```gherkin
Feature: User Profile
  As a logged-in user
  I want to manage my profile
  So that I can update my information

  Scenario: View Own Profile
    Given I am logged in
    When I visit "/user/me"
    Then I should see my profile information:
      | Username | Email | Member Since | Last Login |
    And I should see "Edit Profile" button

  Scenario: Edit Profile
    Given I am on my profile page
    When I click "Edit Profile"
    And I update:
      | Field | Value |
      | Name | John Doe |
      | Bio | Software Developer |
      | Website | https://example.com |
    And I click "Save"
    Then profile should be updated
    And I should see "Profile updated successfully"

  Scenario: Change Password
    Given I am on profile edit page
    When I fill in:
      | Current Password | OldPass123! |
      | New Password | NewPass456! |
      | Confirm Password | NewPass456! |
    And I click "Change Password"
    Then password should be updated
    And I should receive confirmation email
```

## API Testing

### Authentication API
```javascript
// POST /api/auth/login
{
  "email": "user@example.com",
  "password": "Pass123!"
}
// Response: 200 OK with JWT token

// POST /api/auth/refresh
{
  "refresh_token": "..."
}
// Response: 200 OK with new access token

// POST /api/auth/logout
// Headers: Authorization: Bearer {token}
// Response: 200 OK
```

### User Management API
```javascript
// GET /api/users
// Headers: Authorization: Bearer {admin_token}
// Response: List of users with pagination

// POST /api/users
{
  "username": "newuser",
  "email": "new@example.com",
  "password": "Pass123!",
  "role_id": 2
}
// Response: 201 Created

// PUT /api/users/{id}
{
  "email": "updated@example.com",
  "status": "active"
}
// Response: 200 OK

// DELETE /api/users/{id}
// Response: 204 No Content
```

## Security Testing

### Authentication Security
- [ ] SQL injection in login form
- [ ] XSS in username/profile fields
- [ ] CSRF token validation
- [ ] Session fixation protection
- [ ] Brute force protection (rate limiting)
- [ ] Password complexity requirements
- [ ] Secure password storage (bcrypt/argon2)

### Authorization Security
- [ ] Vertical privilege escalation
- [ ] Horizontal privilege escalation
- [ ] Direct object reference
- [ ] Missing function level access control
- [ ] JWT token validation
- [ ] Token expiration handling

### Session Management
- [ ] Session timeout (idle/absolute)
- [ ] Concurrent session handling
- [ ] Session invalidation on logout
- [ ] Remember me functionality security
- [ ] Secure cookie flags (HttpOnly, Secure, SameSite)

## Performance Benchmarks

### Expected Response Times
- Login page load: < 500ms
- Authentication process: < 1000ms
- User list (100 users): < 800ms
- Profile page load: < 600ms
- Permission check: < 50ms

### Database Queries
- User authentication: 1-2 queries
- Load user with roles: 2-3 queries
- Permission check: 1 cached query
- User list with pagination: 2 queries

## Edge Cases

### Data Validation
- [ ] Maximum username length (255 chars)
- [ ] Special characters in username
- [ ] Unicode in names/bio
- [ ] Email validation (RFC compliant)
- [ ] Duplicate email with different case
- [ ] Empty required fields
- [ ] SQL reserved words as username

### Concurrency Issues
- [ ] Simultaneous role assignment
- [ ] Concurrent password changes
- [ ] Race condition in registration
- [ ] Multiple login attempts
- [ ] Token generation collision

### Boundary Conditions
- [ ] User with no roles
- [ ] User with all roles
- [ ] Role with no permissions
- [ ] Role with all permissions
- [ ] Deleted user references
- [ ] Circular role dependencies

## Database Integrity

### Required Indexes
```sql
-- Performance critical indexes
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_username ON users(username);
CREATE INDEX idx_users_status ON users(status);
CREATE INDEX idx_roles_users_user_id ON roles_users(user_id);
CREATE INDEX idx_roles_users_role_id ON roles_users(role_id);
CREATE INDEX idx_permissions_role_id ON permissions(role_id);
```

### Foreign Key Constraints
```sql
-- Maintain referential integrity
ALTER TABLE roles_users 
  ADD CONSTRAINT fk_roles_users_user 
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;

ALTER TABLE roles_users 
  ADD CONSTRAINT fk_roles_users_role 
  FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE;

ALTER TABLE permissions 
  ADD CONSTRAINT fk_permissions_role 
  FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE;
```

## Migration Validation

### CakePHP 3 → 5 Specific Checks
- [ ] Authentication component migration
- [ ] Authorization middleware implementation
- [ ] Password hasher compatibility
- [ ] Session handler changes
- [ ] Cookie authentication updates
- [ ] ACL to Authorization plugin
- [ ] Form helper changes in views
- [ ] Validation rule updates
- [ ] Entity accessor changes
- [ ] Table finder methods

## Test Data Requirements

### Users
```sql
-- Test users for different scenarios
INSERT INTO users (username, email, password, status, created) VALUES
('admin', 'admin@example.com', '$2y$10$...', 'active', NOW()),
('editor', 'editor@example.com', '$2y$10$...', 'active', NOW()),
('author', 'author@example.com', '$2y$10$...', 'active', NOW()),
('blocked', 'blocked@example.com', '$2y$10$...', 'blocked', NOW()),
('pending', 'pending@example.com', '$2y$10$...', 'pending', NOW());
```

### Roles
```sql
-- Standard roles
INSERT INTO roles (name, slug) VALUES
('Administrator', 'administrator'),
('Editor', 'editor'),
('Author', 'author'),
('Subscriber', 'subscriber');
```

### Permissions Matrix
```sql
-- Sample permission setup
INSERT INTO permissions (role_id, aco_id, allowed) VALUES
(1, 1, 1), -- Admin: all permissions
(2, 2, 1), -- Editor: content management
(3, 3, 1), -- Author: create content
(4, 4, 0); -- Subscriber: read only
```

## Success Metrics

- All authentication flows working
- 100% of user CRUD operations functional
- Role-based access control enforced
- No security vulnerabilities
- Response times within benchmarks
- Zero data loss during migration
- Session management stable
- Password reset flow operational
- API endpoints responding correctly
- Admin interface fully functional