# Test Plan: Installation System

## Feature Overview
The Installer plugin provides the initial setup wizard for QuickAppsCMS including database configuration, admin user creation, system initialization, and basic site configuration.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/installer/src/Controller/AppController.php` - Base installer controller
- `src/vendor/quickapps-plugins/installer/src/Controller/StartupController.php` - Installation wizard

### Views
- `src/vendor/quickapps-plugins/installer/src/Template/Startup/` - Installation wizard interface
- `src/vendor/quickapps-plugins/installer/src/Template/Layout/` - Installer layout

### Utilities
- `src/vendor/quickapps-plugins/installer/src/Utility/DatabaseInstaller.php` - Database setup
- `src/vendor/quickapps-plugins/installer/src/Utility/ConfigGenerator.php` - Configuration file generation

### Database Tables
- Creates all core QuickAppsCMS tables during installation
- No installer-specific tables

### Installation Assets
- Installation wizard JavaScript and CSS
- Language files for installation interface
- Default content and configuration

## Routes & URLs

### Installation Routes
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Installation Start | `/installer` | `Installer\Controller\StartupController::index()` | Welcome screen |
| Requirements Check | `/installer/requirements` | `Installer\Controller\StartupController::requirements()` | System requirements |
| Database Setup | `/installer/database` | `Installer\Controller\StartupController::database()` | Database configuration |
| Site Configuration | `/installer/site` | `Installer\Controller\StartupController::site()` | Basic site settings |
| Admin User | `/installer/admin` | `Installer\Controller\StartupController::admin()` | Create admin account |
| Installation Complete | `/installer/complete` | `Installer\Controller\StartupController::complete()` | Installation success |

## Test Scenarios

### 1. Installation Prerequisites
```gherkin
Feature: Installation Prerequisites
  As a system administrator
  I want to verify system requirements
  Before installing QuickAppsCMS

  Scenario: System Requirements Check
    Given fresh QuickAppsCMS installation
    When I visit "/installer/requirements"
    Then I should see requirements check for:
      | PHP Version | >= 8.1 |
      | MySQL Version | >= 8.0 |
      | Required Extensions | mbstring, openssl, pdo, curl |
      | File Permissions | tmp/, logs/, webroot/ writable |
      | Memory Limit | >= 128MB |

  Scenario: Requirements Pass
    Given all requirements are met
    When I review requirements page
    Then all checks should show green/pass
    And "Continue" button should be enabled
    And I can proceed to next step

  Scenario: Requirements Fail
    Given some requirements are not met
    When I view requirements page
    Then failed checks should show red/fail
    And "Continue" button should be disabled
    And specific error messages should display
    And guidance for fixing issues provided

  Scenario: PHP Extension Check
    Given PHP extensions are being checked
    When installer validates extensions
    Then required extensions should be verified:
      | mbstring | For multibyte string handling |
      | openssl | For security functions |
      | pdo_mysql | For database connectivity |
      | curl | For HTTP requests |
      | gd | For image processing |

  Scenario: File Permission Check
    Given installer checks file permissions
    When validating directory access
    Then writable directories should be verified:
      | /tmp | Temporary files |
      | /logs | Log files |
      | /webroot/files | Uploaded files |
      | /config | Configuration files |
```

### 2. Database Configuration
```gherkin
Feature: Database Setup
  As an installer
  I want to configure database connection
  To store QuickAppsCMS data

  Scenario: Database Connection Form
    Given I am on database configuration step
    When I view the form
    Then I should see fields for:
      | Database Host | Default: localhost |
      | Database Name | Required field |
      | Username | Required field |
      | Password | Optional field |
      | Port | Default: 3306 |
      | Database Prefix | Optional |

  Scenario: Test Database Connection
    Given I enter database credentials
    When I click "Test Connection"
    Then system should:
      | Attempt connection to database |
      | Display success/failure message |
      | Enable continue if successful |
      | Show specific error if failed |

  Scenario: Create Database
    Given database doesn't exist
    When I check "Create database if not exists"
    And provide valid credentials
    Then installer should:
      | Attempt to create database |
      | Set proper character encoding (utf8mb4) |
      | Verify creation success |
      | Proceed with table creation |

  Scenario: Database Schema Creation
    Given valid database connection
    When I proceed with installation
    Then installer should create:
      | All core CMS tables |
      | Proper indexes and constraints |
      | Default data and settings |
      | Foreign key relationships |

  Scenario: Database Error Handling
    Given invalid database configuration
    When I attempt to proceed
    Then installer should:
      | Display clear error message |
      | Maintain form data |
      | Suggest common solutions |
      | Allow retry without restart |
```

### 3. Site Configuration
```gherkin
Feature: Site Configuration
  As a site owner
  I want to configure basic settings
  During installation

  Scenario: Basic Site Information
    Given I am on site configuration step
    When I fill in site details:
      | Field | Value |
      | Site Name | My QuickAppsCMS Site |
      | Site Email | admin@example.com |
      | Default Language | English |
      | Timezone | America/New_York |
      | URL Rewriting | Enabled/Disabled |
    Then settings should be saved to configuration

  Scenario: Language Selection
    Given multiple languages available
    When I select default language
    Then installer should:
      | Set site default language |
      | Load appropriate translations |
      | Configure locale settings |
      | Install language pack |

  Scenario: Timezone Configuration
    Given timezone selection dropdown
    When I choose timezone
    Then system should:
      | Set PHP default timezone |
      | Configure database timezone |
      | Update configuration files |
      | Test timezone functionality |

  Scenario: URL Rewriting Test
    Given web server configuration
    When I test URL rewriting
    Then installer should:
      | Check .htaccess file |
      | Test clean URLs |
      | Verify server support |
      | Configure appropriately |

  Scenario: Email Configuration
    Given email settings form
    When I configure email:
      | SMTP Server | mail.example.com |
      | SMTP Port | 587 |
      | Username | user@example.com |
      | Password | [password] |
      | Encryption | TLS |
    Then installer should test email sending
```

### 4. Admin User Creation
```gherkin
Feature: Admin User Setup
  As an installer
  I want to create initial admin user
  To manage the CMS

  Scenario: Admin User Form
    Given I am on admin user creation step
    When I view the form
    Then I should see fields for:
      | Username | Required, unique |
      | Email | Required, valid format |
      | Password | Required, strong |
      | Confirm Password | Must match |
      | First Name | Optional |
      | Last Name | Optional |

  Scenario: Password Strength Validation
    Given I am creating admin password
    When I enter password
    Then system should validate:
      | Minimum 8 characters |
      | Mix of letters and numbers |
      | At least one special character |
      | Not common/weak password |
      | Password strength indicator |

  Scenario: Username Validation
    Given I enter admin username
    When system validates username
    Then it should check:
      | Minimum length requirements |
      | Valid characters only |
      | Not reserved words |
      | Uniqueness in system |

  Scenario: Admin Role Assignment
    Given admin user is created
    When installation completes
    Then admin user should have:
      | Administrator role assigned |
      | All permissions granted |
      | Access to admin interface |
      | Ability to manage system |

  Scenario: Default Content Creation
    Given admin user exists
    When installation proceeds
    Then system should create:
      | Welcome page content |
      | Default menu structure |
      | Basic content types |
      | Sample configuration |
```

### 5. Installation Completion
```gherkin
Feature: Installation Completion
  As an installer
  I want to finalize installation
  And verify system functionality

  Scenario: Final Configuration Generation
    Given all installation steps completed
    When finalizing installation
    Then system should:
      | Generate app.php configuration |
      | Set security salt and keys |
      | Configure caching settings |
      | Set file permissions |
      | Create initial content |

  Scenario: Success Page Display
    Given installation completed successfully
    When I view completion page
    Then I should see:
      | Success confirmation message |
      | Link to admin interface |
      | Link to public site |
      | Security recommendations |
      | Next steps guidance |

  Scenario: Security Cleanup
    Given installation is complete
    When cleanup process runs
    Then installer should:
      | Remove installation files |
      | Create security markers |
      | Set production configuration |
      | Disable debug mode |

  Scenario: Post-Installation Verification
    Given installation claims success
    When I test system functionality
    Then I should verify:
      | Admin login works |
      | Database connection active |
      | File uploads functional |
      | Basic page rendering |
      | Core plugins loaded |

  Scenario: Installation Lockout
    Given installation is complete
    When I try to access installer again
    Then system should:
      | Redirect to homepage |
      | Show "already installed" message |
      | Prevent reinstallation |
      | Maintain security |
```

### 6. Error Handling & Recovery
```gherkin
Feature: Installation Error Handling
  As an installer
  I want graceful error handling
  To recover from installation issues

  Scenario: Database Connection Failure
    Given database connection fails
    When error occurs during installation
    Then system should:
      | Display clear error message |
      | Maintain installation progress |
      | Allow connection retry |
      | Provide troubleshooting tips |

  Scenario: File Permission Errors
    Given insufficient file permissions
    When installation tries to write files
    Then system should:
      | Identify specific permission issues |
      | Show required permissions |
      | Suggest fix commands |
      | Allow retry after fix |

  Scenario: Partial Installation Recovery
    Given installation was interrupted
    When I restart installation process
    Then system should:
      | Detect existing progress |
      | Resume from appropriate step |
      | Validate existing data |
      | Continue safely |

  Scenario: Installation Rollback
    Given critical error during installation
    When system cannot proceed
    Then installer should:
      | Clean up partial installation |
      | Remove created database tables |
      | Reset configuration files |
      | Allow fresh restart |

  Scenario: Memory/Timeout Issues
    Given large database creation
    When operation times out
    Then system should:
      | Break operation into chunks |
      | Show progress indicator |
      | Resume from checkpoint |
      | Handle script timeouts |
```

## API Testing

### Installation API Endpoints
```javascript
// POST /installer/check-requirements
// Response: System requirements validation

{
  "php_version": {"required": "8.1", "current": "8.2", "status": "pass"},
  "mysql_version": {"required": "8.0", "current": "8.0.25", "status": "pass"},
  "extensions": {
    "mbstring": {"status": "pass"},
    "openssl": {"status": "pass"},
    "pdo_mysql": {"status": "fail", "message": "Extension not installed"}
  },
  "permissions": {
    "/tmp": {"status": "pass"},
    "/logs": {"status": "fail", "message": "Directory not writable"}
  }
}

// POST /installer/test-database
{
  "host": "localhost",
  "database": "quickapps_test",
  "username": "root",
  "password": "password",
  "port": 3306
}
// Response: Connection test result

{
  "success": true,
  "message": "Database connection successful",
  "database_exists": false,
  "can_create": true
}

// POST /installer/create-admin
{
  "username": "admin",
  "email": "admin@example.com",
  "password": "SecurePass123!",
  "first_name": "Admin",
  "last_name": "User"
}
// Response: Admin user creation result

// GET /installer/progress
// Response: Installation progress status

{
  "step": 3,
  "total_steps": 5,
  "current_step": "database",
  "completed": ["requirements", "configuration"],
  "remaining": ["admin", "finalize"]
}
```

## Performance Requirements

### Installation Times
- Requirements check: < 5 seconds
- Database setup: < 30 seconds
- Schema creation: < 60 seconds
- Admin user creation: < 5 seconds
- Final configuration: < 15 seconds

### Resource Usage
- Memory usage: < 128MB during installation
- Disk space: Minimum 50MB for base installation
- Database size: ~5MB after initial setup

## Edge Cases & Validation

### System Variations
- [ ] Different PHP versions (8.1, 8.2, 8.3)
- [ ] MySQL vs MariaDB databases
- [ ] Different web servers (Apache, Nginx)
- [ ] Various hosting environments
- [ ] Windows vs Linux systems

### Installation Scenarios
- [ ] Fresh installation
- [ ] Upgrade installation
- [ ] Reinstallation over existing
- [ ] Partial installation recovery
- [ ] Multi-language installation

### Error Conditions
- [ ] Network connectivity issues
- [ ] Insufficient disk space
- [ ] Database server down
- [ ] File permission errors
- [ ] Memory limit exceeded

## Security Testing

### Installation Security
- [ ] Installer file removal after completion
- [ ] Secure configuration generation
- [ ] Strong default passwords
- [ ] SQL injection prevention
- [ ] File upload validation

### Configuration Security
- [ ] Database credential handling
- [ ] Configuration file permissions
- [ ] Security key generation
- [ ] Debug mode disabled in production

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] Installation wizard compatibility
- [ ] Database schema updates
- [ ] Configuration format changes
- [ ] Asset loading updates
- [ ] Form helper compatibility

## Test Data Requirements

### Installation Test Cases
```json
{
  "valid_database": {
    "host": "localhost",
    "database": "quickapps_test",
    "username": "root",
    "password": "password"
  },
  "admin_user": {
    "username": "admin",
    "email": "admin@test.com",
    "password": "SecurePass123!"
  },
  "site_config": {
    "name": "Test Site",
    "email": "test@example.com",
    "timezone": "UTC"
  }
}
```

## Success Metrics

### Functional Success
- Installation wizard completes successfully
- Database schema created correctly
- Admin user account functional
- Site configuration applied
- System ready for use

### Performance Success
- Installation completes within time limits
- No memory limit errors
- Database operations efficient
- File operations successful

### Security Success
- Installer properly secured after completion
- Strong passwords enforced
- Secure configuration generated
- File permissions set correctly

### User Experience Success
- Clear installation progress indication
- Helpful error messages
- Intuitive wizard interface
- Comprehensive success confirmation