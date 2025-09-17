# Test Plan: Content Management System

## Feature Overview
The Content plugin provides the core CMS functionality including content types, content creation, publishing workflows, and content display.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/content/src/Controller/ServeController.php` - Content display/serving
- `src/vendor/quickapps-plugins/content/src/Controller/Admin/ManageController.php` - Content CRUD operations
- `src/vendor/quickapps-plugins/content/src/Controller/Admin/TypesController.php` - Content type management
- `src/vendor/quickapps-plugins/content/src/Controller/Admin/FieldsController.php` - Field management for content types
- `src/vendor/quickapps-plugins/content/src/Controller/Admin/CommentsController.php` - Comment moderation

### Models
- `src/vendor/quickapps-plugins/content/src/Model/Table/ContentsTable.php`
- `src/vendor/quickapps-plugins/content/src/Model/Table/ContentTypesTable.php`
- `src/vendor/quickapps-plugins/content/src/Model/Table/ContentRevisionsTable.php`
- `src/vendor/quickapps-plugins/content/src/Model/Entity/Content.php`
- `src/vendor/quickapps-plugins/content/src/Model/Entity/ContentType.php`

### Views
- `src/vendor/quickapps-plugins/content/src/Template/Serve/` - Frontend content display
- `src/vendor/quickapps-plugins/content/src/Template/Admin/Manage/` - Content management interface
- `src/vendor/quickapps-plugins/content/src/Template/Admin/Types/` - Content type management
- `src/vendor/quickapps-plugins/content/src/Template/Admin/Fields/` - Field configuration interface

### Database Tables
- `contents` - Content items
- `content_types` - Content type definitions
- `content_revisions` - Version history
- `content_types_contents` - Content-type relationships
- `eav_attributes` - Dynamic field definitions (via EAV)
- `eav_values` - Dynamic field values (via EAV)

## Routes & URLs

### Public Routes (Frontend)
| Route | URL Pattern | Controller Action | Description |
|-------|------------|------------------|-------------|
| Homepage | `/` | `Content\Controller\ServeController::home()` | Site homepage |
| Content View | `/{content_type}/{slug}.html` | `Content\Controller\ServeController::details()` | View content item |
| Search | `/find/{criteria}` | `Content\Controller\ServeController::search()` | Search results |
| RSS Feed | `/rss/{criteria}` | `Content\Controller\ServeController::rss()` | RSS feed |
| Content List | `/{content_type}` | `Content\Controller\ServeController::index()` | List by type |

### Admin Routes (Backend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Content List | `/admin/content/manage` | `Content\Admin\ManageController::index()` | List all content |
| Create Content | `/admin/content/manage/create` | `Content\Admin\ManageController::create()` | Create new content |
| Edit Content | `/admin/content/manage/edit/{id}` | `Content\Admin\ManageController::edit()` | Edit existing content |
| Delete Content | `/admin/content/manage/delete/{id}` | `Content\Admin\ManageController::delete()` | Delete content |
| Publish/Unpublish | `/admin/content/manage/publish/{id}` | `Content\Admin\ManageController::publish()` | Toggle publish status |
| Content Types | `/admin/content/types` | `Content\Admin\TypesController::index()` | Manage content types |
| Add Content Type | `/admin/content/types/add` | `Content\Admin\TypesController::add()` | Create content type |
| Edit Content Type | `/admin/content/types/edit/{id}` | `Content\Admin\TypesController::edit()` | Edit content type |
| Manage Fields | `/admin/content/fields/{type_id}` | `Content\Admin\FieldsController::index()` | Manage type fields |
| Add Field | `/admin/content/fields/add/{type_id}` | `Content\Admin\FieldsController::add()` | Add field to type |
| Moderate Comments | `/admin/content/comments` | `Content\Admin\CommentsController::index()` | Comment moderation |

## Test Scenarios

### 1. Content Creation Workflow
```gherkin
Feature: Content Creation
  As a content editor
  I want to create content
  So that I can publish information

  Scenario: Create Basic Article
    Given I am logged in as editor
    When I visit "/admin/content/manage/create"
    And I select content type "article"
    And I fill in:
      | Field | Value |
      | Title | Test Article |
      | Slug | test-article |
      | Body | Article content here |
      | Status | Published |
    And I click "Save"
    Then content should be created in database
    And content should be accessible at "/article/test-article.html"
    And content should appear in "/admin/content/manage"

  Scenario: Create with Custom Fields
    Given content type "product" has custom fields:
      | Field Name | Field Type |
      | Price | Decimal |
      | SKU | Text |
      | Images | File |
    When I create a product with:
      | Title | Sample Product |
      | Price | 99.99 |
      | SKU | PROD-001 |
    Then values should be stored in EAV tables
    And fields should display on frontend

  Scenario: Save as Draft
    Given I am creating content
    When I set status to "Draft"
    And I save the content
    Then content should not be publicly visible
    But should appear in admin with "Draft" status

  Scenario: Schedule Publication
    Given I am creating content
    When I set "Publish on" to future date "2025-02-01 09:00"
    And I save the content
    Then content status should be "Scheduled"
    And content should auto-publish at specified time
```

### 2. Content Type Management
```gherkin
Feature: Content Type Management
  As an administrator
  I want to manage content types
  So that I can structure content

  Scenario: Create New Content Type
    Given I am on "/admin/content/types/add"
    When I fill in:
      | Field | Value |
      | Name | Event |
      | Slug | event |
      | Description | Events and activities |
    And I configure:
      | Setting | Value |
      | Comments | Enabled |
      | Revisions | Enabled |
      | Menu | Can appear in menu |
    And I click "Save"
    Then content type should be created
    And should appear in content type list
    And create form should be available at "/admin/content/manage/create?type=event"

  Scenario: Add Fields to Content Type
    Given content type "event" exists
    When I visit "/admin/content/fields/event"
    And I add field:
      | Field Label | Event Date |
      | Field Type | DateTime |
      | Required | Yes |
      | Help Text | Select event date and time |
    Then field should be added to content type
    And field should appear on create/edit forms
    And EAV attributes should be created

  Scenario: Configure Display Modes
    Given content type "article" exists
    When I configure display modes:
      | Mode | Fields Shown |
      | Full | All fields |
      | Teaser | Title, Summary |
      | Search Result | Title, Date, Excerpt |
      | RSS | Title, Body, Link |
    Then display modes should be saved
    And content should render differently per mode
```

### 3. Content Editing & Versioning
```gherkin
Feature: Content Editing
  As a content editor
  I want to edit and version content
  So that I can update and track changes

  Scenario: Edit Published Content
    Given content "test-article" exists and is published
    When I visit "/admin/content/manage/edit/{id}"
    And I change title to "Updated Article"
    And I modify body content
    And I click "Save"
    Then content should be updated
    And revision should be created
    And frontend should show updated content

  Scenario: View Revision History
    Given content has 5 revisions
    When I click "Revisions" tab
    Then I should see revision list with:
      | Revision | Date | Author | Changes |
    And I should be able to view each revision
    And I should be able to compare revisions

  Scenario: Revert to Previous Version
    Given content has multiple revisions
    When I select revision #3
    And I click "Revert to this revision"
    Then content should be reverted
    And new revision should be created
    And frontend should show reverted content

  Scenario: Bulk Operations
    Given I am on content list page
    When I select multiple content items
    And I choose "Unpublish" from bulk actions
    And I click "Apply"
    Then selected items should be unpublished
    And operation should be logged
```

### 4. Content Display & View Modes
```gherkin
Feature: Content Display
  As a site visitor
  I want to view content
  In different formats

  Scenario: View Full Content
    Given published content exists at "/article/test.html"
    When I visit the URL
    Then I should see:
      | Element | Content |
      | Title | Test Article |
      | Body | Full content |
      | Author | John Doe |
      | Date | January 15, 2025 |
      | Tags | Technology, News |
    And comments should be displayed if enabled
    And related content should be shown

  Scenario: View Content List
    Given 10 articles are published
    When I visit "/article"
    Then I should see article list
    And articles should be paginated (5 per page)
    And each should show teaser view:
      | Title | Summary | Read More Link |

  Scenario: Search Results Display
    Given I search for "technology"
    When I view search results
    Then content should display in search-result mode
    And should show:
      | Title with link |
      | Search excerpt with highlighted terms |
      | Content type |
      | Publication date |
```

### 5. Content Relationships & References
```gherkin
Feature: Content Relationships
  As a content editor
  I want to create content relationships
  So that content is interconnected

  Scenario: Add Related Content
    Given I am editing content
    When I add related content references:
      | Related Article 1 |
      | Related Article 2 |
    And I save
    Then relationships should be stored
    And related content should display on frontend

  Scenario: Parent-Child Hierarchy
    Given I have parent page "About Us"
    When I create child pages:
      | Our Team |
      | Our History |
      | Our Mission |
    Then hierarchy should be established
    And breadcrumbs should reflect structure
    And navigation should show parent-child relationship

  Scenario: Content in Multiple Categories
    Given taxonomy vocabularies exist
    When I assign content to terms:
      | Vocabulary | Terms |
      | Category | News, Technology |
      | Tags | AI, Machine Learning |
    Then associations should be saved
    And content should appear in category listings
```

### 6. Comment Management
```gherkin
Feature: Comment System
  As a content moderator
  I want to manage comments
  So that discussions are productive

  Scenario: Moderate New Comments
    Given comments are set to "Require approval"
    When a user posts a comment
    Then comment should appear in moderation queue
    And I should be able to:
      | Approve | Reject | Edit | Mark as Spam |

  Scenario: Comment Threading
    Given threaded comments are enabled
    When users reply to comments
    Then comments should display in tree structure
    And nesting should respect max depth setting

  Scenario: Bulk Comment Actions
    Given moderation queue has 20 comments
    When I select multiple comments
    And apply "Approve" action
    Then comments should be published
    And commenters should be notified
```

## API Testing

### Content API Endpoints
```javascript
// GET /api/content
// Query params: type, status, limit, offset
// Response: Paginated content list

// GET /api/content/{id}
// Response: Full content object with fields

// POST /api/content
{
  "title": "New Content",
  "type": "article",
  "body": "Content body",
  "status": "published",
  "fields": {
    "field_image": "image.jpg",
    "field_tags": [1, 2, 3]
  }
}
// Response: 201 Created with content object

// PUT /api/content/{id}
{
  "title": "Updated Title",
  "status": "draft"
}
// Response: 200 OK

// DELETE /api/content/{id}
// Response: 204 No Content

// GET /api/content/{id}/revisions
// Response: List of content revisions

// POST /api/content/{id}/revert/{revision_id}
// Response: 200 OK with reverted content
```

### Content Type API
```javascript
// GET /api/content-types
// Response: List of content types

// GET /api/content-types/{slug}
// Response: Content type with field definitions

// GET /api/content-types/{slug}/fields
// Response: Field configuration for content type
```

## Performance Requirements

### Page Load Times
- Homepage: < 1 second
- Content page: < 800ms
- Content list (50 items): < 1.2 seconds
- Search results: < 1.5 seconds
- Admin content list: < 1 second

### Database Optimization
```sql
-- Required indexes for performance
CREATE INDEX idx_contents_status ON contents(status);
CREATE INDEX idx_contents_type ON contents(content_type_id);
CREATE INDEX idx_contents_slug ON contents(slug);
CREATE INDEX idx_contents_created ON contents(created);
CREATE INDEX idx_contents_author ON contents(author_id);
CREATE INDEX idx_eav_values_entity ON eav_values(entity_id, table_alias);
CREATE INDEX idx_content_revisions_content ON content_revisions(content_id);
```

### Caching Strategy
- Content pages: 1 hour cache
- Content lists: 15 minute cache
- RSS feeds: 30 minute cache
- Search results: 5 minute cache
- Admin pages: No cache

## Security Testing

### Content Security
- [ ] XSS prevention in content fields
- [ ] SQL injection in search queries
- [ ] CSRF protection on forms
- [ ] File upload validation
- [ ] Content access control
- [ ] HTML sanitization
- [ ] Script injection prevention

### Authorization Checks
- [ ] Create content permission
- [ ] Edit own content only
- [ ] Edit any content (admin)
- [ ] Delete permission verification
- [ ] Publish/unpublish rights
- [ ] Field-level permissions

## Edge Cases & Validation

### Content Validation
- [ ] Empty required fields
- [ ] Duplicate slugs
- [ ] Invalid HTML in body
- [ ] Extremely long titles (>255 chars)
- [ ] Special characters in slug
- [ ] Invalid date formats
- [ ] Circular content references

### Data Integrity
- [ ] Orphaned revisions
- [ ] Missing content type
- [ ] Deleted user references
- [ ] Broken field references
- [ ] Invalid EAV values
- [ ] Corrupted serialized data

### Concurrent Operations
- [ ] Simultaneous edits
- [ ] Race condition in slug generation
- [ ] Concurrent revision creation
- [ ] Parallel bulk operations
- [ ] Cache invalidation timing

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] Entity accessor compatibility
- [ ] Behavior migration (Timestamp, Sluggable)
- [ ] Validation rule updates
- [ ] Finder method changes
- [ ] Association loading strategies
- [ ] Query builder syntax
- [ ] View cell updates
- [ ] Helper method changes
- [ ] Event system migration
- [ ] Middleware implementation

## Test Data Requirements

### Content Types
```sql
INSERT INTO content_types (name, slug, description) VALUES
('Article', 'article', 'News and blog articles'),
('Page', 'page', 'Static pages'),
('Product', 'product', 'Product listings'),
('Event', 'event', 'Events and activities');
```

### Sample Content
```sql
INSERT INTO contents (title, slug, body, status, content_type_id, author_id) VALUES
('Welcome to QuickAppsCMS', 'welcome', 'Content here...', 'published', 1, 1),
('About Us', 'about-us', 'About content...', 'published', 2, 1),
('Sample Product', 'sample-product', 'Product description...', 'published', 3, 1),
('Upcoming Event', 'upcoming-event', 'Event details...', 'scheduled', 4, 1);
```

### EAV Test Data
```sql
-- Custom fields for content
INSERT INTO eav_attributes (table_alias, bundle, name, type) VALUES
('contents', 'product', 'price', 'decimal'),
('contents', 'product', 'sku', 'varchar'),
('contents', 'event', 'event_date', 'datetime'),
('contents', 'event', 'location', 'text');
```

## Success Metrics

- All content CRUD operations functional
- Content types fully configurable
- EAV fields working correctly
- Display modes rendering properly
- Revision system operational
- Search and indexing functional
- Comments system working
- Performance within benchmarks
- No security vulnerabilities
- Zero data loss in migration
- All routes accessible
- API endpoints responding
- Cache invalidation working
- Bulk operations successful
- Content relationships intact