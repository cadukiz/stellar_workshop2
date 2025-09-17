# Test Plan: Menu & Navigation System

## Feature Overview
The Menu plugin provides navigation management including menu creation, hierarchical menu items, and dynamic menu rendering across the site.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/menu/src/Controller/Admin/ManageController.php` - Menu CRUD operations
- `src/vendor/quickapps-plugins/menu/src/Controller/Admin/LinksController.php` - Menu links management

### Models
- `src/vendor/quickapps-plugins/menu/src/Model/Table/MenusTable.php`
- `src/vendor/quickapps-plugins/menu/src/Model/Table/MenuLinksTable.php`
- `src/vendor/quickapps-plugins/menu/src/Model/Entity/Menu.php`
- `src/vendor/quickapps-plugins/menu/src/Model/Entity/MenuLink.php`

### Views
- `src/vendor/quickapps-plugins/menu/src/Template/Admin/Manage/` - Menu management interface
- `src/vendor/quickapps-plugins/menu/src/Template/Admin/Links/` - Menu links interface
- `src/vendor/quickapps-plugins/menu/src/Template/Element/` - Menu rendering elements

### Database Tables
- `menus` - Menu containers
- `menu_links` - Individual menu items
- `menu_links_menus` - Menu-link relationships

## Routes & URLs

### Admin Routes (Backend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Menu List | `/admin/menu/manage` | `Menu\Admin\ManageController::index()` | List all menus |
| Create Menu | `/admin/menu/manage/add` | `Menu\Admin\ManageController::add()` | Create new menu |
| Edit Menu | `/admin/menu/manage/edit/{id}` | `Menu\Admin\ManageController::edit()` | Edit menu settings |
| Delete Menu | `/admin/menu/manage/delete/{id}` | `Menu\Admin\ManageController::delete()` | Delete menu |
| Menu Links | `/admin/menu/links/{menu_id}` | `Menu\Admin\LinksController::index()` | Manage menu items |
| Add Link | `/admin/menu/links/add/{menu_id}` | `Menu\Admin\LinksController::add()` | Add menu item |
| Edit Link | `/admin/menu/links/edit/{id}` | `Menu\Admin\LinksController::edit()` | Edit menu item |
| Reorder Links | `/admin/menu/links/reorder/{menu_id}` | `Menu\Admin\LinksController::reorder()` | Drag-drop reorder |

## Test Scenarios

### 1. Menu Management
```gherkin
Feature: Menu Management
  As an administrator
  I want to manage menus
  So that I can organize site navigation

  Scenario: Create New Menu
    Given I am on "/admin/menu/manage/add"
    When I fill in:
      | Field | Value |
      | Title | Main Navigation |
      | Machine Name | main-nav |
      | Description | Primary site navigation |
    And I click "Save"
    Then menu should be created
    And should appear in menu list
    And should be available for block placement

  Scenario: Edit Menu Properties
    Given menu "footer-menu" exists
    When I edit the menu
    And change title to "Footer Links"
    And update description
    And click "Save"
    Then menu should be updated
    And changes should reflect in frontend

  Scenario: Delete Menu
    Given menu "temporary-menu" exists with no links
    When I delete the menu
    Then menu should be removed
    And associated blocks should be removed
```

### 2. Menu Link Management
```gherkin
Feature: Menu Links
  As an administrator
  I want to manage menu items
  So that users can navigate the site

  Scenario: Add Content Link
    Given I am adding link to "main-menu"
    When I select "Content" as link type
    And I search and select "About Us" page
    And I set:
      | Title | About |
      | Weight | 10 |
      | Expanded | No |
    And click "Save"
    Then link should be added to menu
    And should appear in menu tree

  Scenario: Add Custom URL
    Given I am adding link to menu
    When I select "Custom URL" as type
    And enter:
      | URL | https://external.com |
      | Title | External Link |
      | Open in | New Window |
      | CSS Class | external-link |
    Then link should be created
    And should have target="_blank"

  Scenario: Create Nested Menu Structure
    Given I have menu "main-menu"
    When I create hierarchical structure:
      | Level 1 | Level 2 | Level 3 |
      | Products | Hardware | Computers |
      | Products | Hardware | Monitors |
      | Products | Software | Operating Systems |
      | Services | Support | |
      | Services | Training | |
    Then menu should display nested structure
    And frontend should render dropdown menus

  Scenario: Reorder Menu Items
    Given menu has 10 items
    When I drag "Contact" above "About"
    And I drag "Services" into "Products"
    And click "Save Order"
    Then new order should be saved
    And frontend should reflect new structure
```

### 3. Menu Display & Rendering
```gherkin
Feature: Menu Display
  As a site visitor
  I want to use navigation menus
  So that I can browse the site

  Scenario: Primary Navigation Display
    Given "main-menu" is assigned to header region
    When I visit the homepage
    Then I should see main menu items
    And active trail should be highlighted
    And current page should have "active" class

  Scenario: Mobile Menu
    Given I am on mobile device
    When I click hamburger menu icon
    Then mobile menu should expand
    And should show all menu levels
    And should be touch-friendly

  Scenario: Breadcrumb Navigation
    Given I am on "/products/software/cms"
    Then breadcrumb should show:
      | Home > Products > Software > CMS |
    And each level should be clickable
    And current page should not be linked

  Scenario: Contextual Menu Display
    Given menu has visibility rules:
      | Show only for logged-in users |
      | Hide on pages matching /admin/* |
    When conditions are met
    Then menu should display accordingly
```

### 4. Menu Permissions & Access
```gherkin
Feature: Menu Access Control
  As an administrator
  I want to control menu visibility
  Based on user permissions

  Scenario: Role-based Menu Items
    Given menu item requires "editor" role
    When I view site as:
      | Anonymous | Item hidden |
      | Subscriber | Item hidden |
      | Editor | Item visible |
      | Admin | Item visible |
    Then menu should respect permissions

  Scenario: Conditional Menu Display
    Given menu item has conditions:
      | Show only on content type "article" |
      | Show only for language "en" |
    When conditions match
    Then menu item should display
    Otherwise menu item should be hidden
```

## API Testing

### Menu API Endpoints
```javascript
// GET /api/menus
// Response: List of all menus

// GET /api/menus/{id}
// Response: Menu with all links in tree structure

// POST /api/menus
{
  "title": "New Menu",
  "machine_name": "new_menu",
  "description": "Menu description"
}
// Response: 201 Created

// PUT /api/menus/{id}
{
  "title": "Updated Title",
  "description": "Updated description"
}
// Response: 200 OK

// DELETE /api/menus/{id}
// Response: 204 No Content

// POST /api/menu-links
{
  "menu_id": 1,
  "parent_id": null,
  "title": "New Link",
  "url": "/path/to/page",
  "weight": 0,
  "expanded": false
}
// Response: 201 Created

// PUT /api/menu-links/reorder
{
  "links": [
    {"id": 1, "parent_id": null, "weight": 0},
    {"id": 2, "parent_id": 1, "weight": 0},
    {"id": 3, "parent_id": 1, "weight": 1}
  ]
}
// Response: 200 OK
```

## Performance Requirements

### Response Times
- Menu rendering: < 50ms (cached)
- Menu admin page: < 500ms
- Link reordering: < 300ms
- Menu tree building: < 100ms

### Database Queries
- Menu display: 1-2 cached queries
- Menu admin: 3-4 queries
- Link management: 2-3 queries

### Caching
- Menu trees: 1 hour cache
- Menu blocks: 30 minute cache
- Breadcrumbs: 15 minute cache

## Edge Cases & Validation

### Menu Structure
- [ ] Circular parent references
- [ ] Orphaned menu items
- [ ] Maximum nesting depth (5 levels)
- [ ] Empty menus
- [ ] Duplicate machine names
- [ ] Special characters in URLs

### Link Validation
- [ ] Invalid URLs
- [ ] Broken content references
- [ ] Missing parent items
- [ ] Weight conflicts
- [ ] Access to unpublished content
- [ ] Deleted content references

## Success Metrics

- All menu CRUD operations working
- Hierarchical structure maintained
- Drag-drop reordering functional
- Menu rendering correct
- Mobile menu working
- Breadcrumbs generating correctly
- Permissions enforced
- Cache invalidation working
- No broken links
- Performance within limits