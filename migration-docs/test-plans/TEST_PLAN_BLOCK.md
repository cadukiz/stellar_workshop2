# Test Plan: Block Management System

## Feature Overview
The Block plugin provides content block management, allowing placement of various content blocks in theme regions with visibility rules and caching.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/block/src/Controller/Admin/ManageController.php` - Block CRUD operations

### Models
- `src/vendor/quickapps-plugins/block/src/Model/Table/BlocksTable.php`
- `src/vendor/quickapps-plugins/block/src/Model/Table/BlockRegionsTable.php`
- `src/vendor/quickapps-plugins/block/src/Model/Entity/Block.php`
- `src/vendor/quickapps-plugins/block/src/Model/Entity/BlockRegion.php`

### Views
- `src/vendor/quickapps-plugins/block/src/Template/Admin/Manage/` - Block management interface
- `src/vendor/quickapps-plugins/block/src/Template/Element/` - Block rendering elements

### Database Tables
- `blocks` - Block definitions
- `block_regions` - Block-region assignments
- `blocks_roles` - Block role permissions

## Routes & URLs

### Admin Routes (Backend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Block List | `/admin/block/manage` | `Block\Admin\ManageController::index()` | List all blocks |
| Add Block | `/admin/block/manage/add` | `Block\Admin\ManageController::add()` | Create new block |
| Edit Block | `/admin/block/manage/edit/{id}` | `Block\Admin\ManageController::edit()` | Edit block |
| Delete Block | `/admin/block/manage/delete/{id}` | `Block\Admin\ManageController::delete()` | Delete block |
| Configure | `/admin/block/manage/configure/{id}` | `Block\Admin\ManageController::configure()` | Block settings |
| Duplicate | `/admin/block/manage/duplicate/{id}` | `Block\Admin\ManageController::duplicate()` | Clone block |

## Test Scenarios

### 1. Block Creation & Management
```gherkin
Feature: Block Management
  As an administrator
  I want to manage blocks
  So that I can add content to regions

  Scenario: Create Custom HTML Block
    Given I am on "/admin/block/manage/add"
    When I fill in:
      | Field | Value |
      | Title | Welcome Message |
      | Machine Name | welcome_block |
      | Body | <h2>Welcome!</h2><p>Content here</p> |
      | Format | Full HTML |
    And I assign to region "Sidebar"
    And click "Save"
    Then block should be created
    And should appear in sidebar region

  Scenario: Create Dynamic Block
    Given I am creating a block
    When I select type "Recent Content"
    And configure:
      | Number of items | 5 |
      | Content types | Article, Blog |
      | Display mode | Titles only |
    Then block should display latest content
    And should update automatically

  Scenario: Configure Block Visibility
    Given block "promo-block" exists
    When I set visibility rules:
      | Show on specific pages | /products/* |
      | Hide for anonymous users | Yes |
      | Show only on mobile | Yes |
    Then block should display conditionally
    And rules should be enforced
```

### 2. Block Placement & Regions
```gherkin
Feature: Block Regions
  As an administrator
  I want to place blocks in regions
  So that content appears correctly

  Scenario: Assign Block to Region
    Given theme has regions:
      | Header | Sidebar | Content | Footer |
    When I assign block to "Header" region
    And set weight to 5
    Then block should appear in header
    And should respect weight ordering

  Scenario: Move Block Between Regions
    Given block is in "Sidebar"
    When I move it to "Footer"
    And adjust weight
    Then block should move to footer
    And should maintain configuration

  Scenario: Multi-theme Block Assignment
    Given site has themes:
      | Frontend Theme | Admin Theme |
    When I configure block for each theme:
      | Frontend: Sidebar |
      | Admin: Hidden |
    Then block should display per theme
```

### 3. Block Types & Widgets
```gherkin
Feature: Block Types
  As an administrator
  I want to use different block types
  For various content needs

  Scenario: Menu Block
    Given I create menu block
    When I select "Main Menu"
    And configure:
      | Starting level | 1 |
      | Maximum depth | 3 |
      | Style | Dropdown |
    Then menu should display in block
    With specified configuration

  Scenario: User Block
    Given I create user block type
    When I select "Who's Online"
    And set refresh interval to 60 seconds
    Then block should show online users
    And update periodically

  Scenario: Custom PHP Block
    Given PHP blocks are enabled
    When I create PHP block with:
      | Code | <?php echo date('Y-m-d'); ?> |
    Then block should execute PHP
    And display current date
    
  Scenario: View Block
    Given I create view block
    When I select view "Recent Articles"
    And configure display settings
    Then block should render view
    With specified filters
```

### 4. Block Caching & Performance
```gherkin
Feature: Block Caching
  As a system
  I want to cache blocks
  For better performance

  Scenario: Cache Static Block
    Given static HTML block exists
    When cache is set to "1 hour"
    Then block should be cached
    And serve from cache within period

  Scenario: Cache Dynamic Block
    Given block shows "Latest Comments"
    When cache is set to "5 minutes"
    Then block should refresh every 5 minutes
    And show updated content

  Scenario: Per-Role Caching
    Given block has role-based content
    When different users view block:
      | Anonymous | Cached version A |
      | Member | Cached version B |
      | Admin | No cache |
    Then appropriate cache should be served
```

## API Testing

### Block API Endpoints
```javascript
// GET /api/blocks
// Response: List of all blocks with regions

// GET /api/blocks/{id}
// Response: Block details with configuration

// POST /api/blocks
{
  "title": "New Block",
  "machine_name": "new_block",
  "body": "<p>Block content</p>",
  "region": "sidebar",
  "weight": 0,
  "status": "enabled",
  "visibility": {
    "pages": "/path/*",
    "roles": [1, 2]
  }
}
// Response: 201 Created

// PUT /api/blocks/{id}
{
  "title": "Updated Title",
  "region": "footer",
  "weight": 10
}
// Response: 200 OK

// DELETE /api/blocks/{id}
// Response: 204 No Content

// PUT /api/blocks/reorder
{
  "blocks": [
    {"id": 1, "region": "sidebar", "weight": 0},
    {"id": 2, "region": "sidebar", "weight": 1}
  ]
}
// Response: 200 OK
```

## Performance Requirements

### Response Times
- Block rendering: < 20ms (cached)
- Block admin page: < 600ms
- Block configuration: < 400ms
- Region assignment: < 200ms

### Caching Strategy
- Static blocks: 1 hour default
- Dynamic blocks: 5-15 minutes
- User-specific: Per-session cache
- Anonymous: Aggressive caching

## Edge Cases & Validation

### Block Configuration
- [ ] Empty block body
- [ ] Invalid HTML in content
- [ ] Circular block references
- [ ] Missing region assignments
- [ ] Duplicate machine names
- [ ] Invalid visibility rules

### Region Management
- [ ] Non-existent regions
- [ ] Weight conflicts
- [ ] Theme switching
- [ ] Responsive visibility
- [ ] Region capacity limits

## Success Metrics

- All block types functional
- Region assignment working
- Visibility rules enforced
- Caching strategy effective
- Performance within limits
- Multi-theme support working
- Weight ordering correct
- API endpoints functional
- No rendering errors
- Mobile responsive display