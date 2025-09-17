# Test Plan: Taxonomy & Categorization System

## Feature Overview
The Taxonomy plugin provides vocabulary and term management for content categorization, tagging, and hierarchical classification systems.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/taxonomy/src/Controller/Admin/ManageController.php` - Main taxonomy management
- `src/vendor/quickapps-plugins/taxonomy/src/Controller/Admin/VocabulariesController.php` - Vocabulary CRUD
- `src/vendor/quickapps-plugins/taxonomy/src/Controller/Admin/TermsController.php` - Term management
- `src/vendor/quickapps-plugins/taxonomy/src/Controller/Admin/TaggerController.php` - Auto-tagging interface

### Models
- `src/vendor/quickapps-plugins/taxonomy/src/Model/Table/VocabulariesTable.php`
- `src/vendor/quickapps-plugins/taxonomy/src/Model/Table/TermsTable.php`
- `src/vendor/quickapps-plugins/taxonomy/src/Model/Entity/Vocabulary.php`
- `src/vendor/quickapps-plugins/taxonomy/src/Model/Entity/Term.php`

### Views
- `src/vendor/quickapps-plugins/taxonomy/src/Template/Admin/Vocabularies/` - Vocabulary management
- `src/vendor/quickapps-plugins/taxonomy/src/Template/Admin/Terms/` - Term management
- `src/vendor/quickapps-plugins/taxonomy/src/Template/Admin/Tagger/` - Tagging interface

### Database Tables
- `vocabularies` - Taxonomy vocabularies
- `terms` - Taxonomy terms
- `terms_contents` - Content-term relationships
- `terms_cache` - Term hierarchy cache

## Routes & URLs

### Admin Routes (Backend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Taxonomy Overview | `/admin/taxonomy/manage` | `Taxonomy\Admin\ManageController::index()` | Taxonomy dashboard |
| Vocabularies List | `/admin/taxonomy/vocabularies` | `Taxonomy\Admin\VocabulariesController::index()` | List vocabularies |
| Add Vocabulary | `/admin/taxonomy/vocabularies/add` | `Taxonomy\Admin\VocabulariesController::add()` | Create vocabulary |
| Edit Vocabulary | `/admin/taxonomy/vocabularies/edit/{id}` | `Taxonomy\Admin\VocabulariesController::edit()` | Edit vocabulary |
| Terms List | `/admin/taxonomy/terms/{vocab_id}` | `Taxonomy\Admin\TermsController::index()` | List terms |
| Add Term | `/admin/taxonomy/terms/add/{vocab_id}` | `Taxonomy\Admin\TermsController::add()` | Create term |
| Edit Term | `/admin/taxonomy/terms/edit/{id}` | `Taxonomy\Admin\TermsController::edit()` | Edit term |
| Tagger | `/admin/taxonomy/tagger` | `Taxonomy\Admin\TaggerController::index()` | Auto-tagging tool |

### Public Routes (Frontend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Term Page | `/taxonomy/term/{id}` | `Taxonomy\Controller\TermController::view()` | View term content |
| Tag Cloud | `/tags` | `Taxonomy\Controller\TagController::cloud()` | Tag cloud display |

## Test Scenarios

### 1. Vocabulary Management
```gherkin
Feature: Vocabulary Management
  As an administrator
  I want to manage vocabularies
  So that I can organize content classification

  Scenario: Create New Vocabulary
    Given I am on "/admin/taxonomy/vocabularies/add"
    When I fill in:
      | Field | Value |
      | Name | Product Categories |
      | Machine Name | product_categories |
      | Description | Categories for products |
    And I configure:
      | Hierarchy | Multiple hierarchy |
      | Required | No |
      | Multiple values | Yes |
    And assign to content types:
      | Product | Page |
    And click "Save"
    Then vocabulary should be created
    And should appear in content type forms

  Scenario: Configure Vocabulary Settings
    Given vocabulary "tags" exists
    When I edit vocabulary settings:
      | Allow free tagging | Yes |
      | Suggest existing terms | Yes |
      | Maximum depth | 5 |
    Then settings should be saved
    And behavior should change accordingly

  Scenario: Delete Vocabulary
    Given vocabulary "temporary" exists with terms
    When I delete vocabulary
    And choose "Delete all terms"
    Then vocabulary should be removed
    And all associated terms deleted
    And content associations cleared
```

### 2. Term Management
```gherkin
Feature: Term Management
  As an administrator
  I want to manage taxonomy terms
  So that content can be categorized

  Scenario: Create Term
    Given I am adding term to "categories"
    When I fill in:
      | Field | Value |
      | Name | Technology |
      | Slug | technology |
      | Description | Tech related content |
      | Parent | [Root] |
      | Weight | 0 |
    And click "Save"
    Then term should be created
    And should appear in term list

  Scenario: Create Hierarchical Terms
    Given vocabulary allows hierarchy
    When I create term structure:
      | Level 1 | Level 2 | Level 3 |
      | Electronics | Computers | Laptops |
      | Electronics | Computers | Desktops |
      | Electronics | Mobile | Smartphones |
      | Electronics | Mobile | Tablets |
    Then hierarchy should be established
    And tree view should display structure

  Scenario: Move Terms
    Given term "Laptops" is under "Computers"
    When I move it under "Mobile"
    Then hierarchy should update
    And content associations should remain

  Scenario: Merge Terms
    Given terms "Tech" and "Technology" exist
    When I merge "Tech" into "Technology"
    Then "Tech" should be deleted
    And content should be reassigned
    And redirects should be created

  Scenario: Bulk Term Operations
    Given I select multiple terms
    When I perform bulk action:
      | Delete selected | Move to parent | Export |
    Then operation should apply to all
    And confirmation should be required
```

### 3. Content Tagging
```gherkin
Feature: Content Tagging
  As a content editor
  I want to tag content
  For better organization

  Scenario: Tag Content with Existing Terms
    Given I am editing content
    When I select terms from "Categories":
      | Technology | News | Featured |
    And save content
    Then terms should be associated
    And content should appear in term listings

  Scenario: Free Tagging
    Given vocabulary allows free tagging
    When I type new tags:
      | "artificial intelligence" | "machine learning" |
    Then new terms should be created
    And associated with content

  Scenario: Auto-complete Suggestions
    Given I start typing "tech"
    Then suggestions should appear:
      | Technology | Technical | Tech News |
    And I can select from suggestions

  Scenario: Required Vocabulary
    Given "Category" is required
    When I try to save without selecting
    Then validation error should appear
    And content should not save

  Scenario: Tag Limits
    Given vocabulary has max 3 terms limit
    When I try to select 4 terms
    Then only 3 should be allowed
    And warning should display
```

### 4. Term Display & Navigation
```gherkin
Feature: Term Pages
  As a site visitor
  I want to browse by terms
  To find related content

  Scenario: View Term Page
    Given term "Technology" has 10 contents
    When I visit "/taxonomy/term/5"
    Then I should see:
      | Term name and description |
      | List of tagged content |
      | Pagination if needed |
      | Related terms |

  Scenario: Browse Term Hierarchy
    Given hierarchical terms exist
    When I view parent term
    Then I should see child terms
    And breadcrumb navigation
    And drill-down capability

  Scenario: Tag Cloud
    Given tag cloud block is displayed
    When I view the cloud
    Then terms should be sized by usage
    And linked to term pages
    And show post counts

  Scenario: Term RSS Feed
    Given term has RSS enabled
    When I visit "/taxonomy/term/5/feed"
    Then RSS feed should display
    With latest content for term
```

### 5. Advanced Taxonomy Features
```gherkin
Feature: Advanced Taxonomy
  As an administrator
  I want advanced taxonomy features
  For complex categorization needs

  Scenario: Synonyms Management
    Given term "AI" exists
    When I add synonyms:
      | Artificial Intelligence | A.I. | Machine Intelligence |
    Then searches for synonyms
    Should find main term content

  Scenario: Term Relations
    Given I have term "PHP"
    When I add related terms:
      | Laravel | Symfony | WordPress |
    Then relations should be stored
    And displayed on term pages

  Scenario: Term Weights & Ordering
    Given terms in vocabulary
    When I adjust weights via drag-drop
    Then order should be saved
    And reflected in displays

  Scenario: Import/Export Terms
    Given I have CSV with terms
    When I import the file
    Then terms should be created
    With hierarchy preserved
    And I can export back to CSV
```

## API Testing

### Taxonomy API Endpoints
```javascript
// GET /api/vocabularies
// Response: List of vocabularies

// GET /api/vocabularies/{id}/terms
// Response: Terms in vocabulary (tree structure)

// POST /api/vocabularies
{
  "name": "New Vocabulary",
  "machine_name": "new_vocab",
  "hierarchy": "multiple",
  "required": false
}
// Response: 201 Created

// POST /api/terms
{
  "vocabulary_id": 1,
  "name": "New Term",
  "parent_id": null,
  "weight": 0,
  "description": "Term description"
}
// Response: 201 Created

// PUT /api/terms/{id}
{
  "name": "Updated Term",
  "parent_id": 5
}
// Response: 200 OK

// POST /api/terms/merge
{
  "source_id": 10,
  "target_id": 5
}
// Response: 200 OK

// GET /api/content/{id}/terms
// Response: Terms assigned to content

// POST /api/content/{id}/terms
{
  "term_ids": [1, 2, 3]
}
// Response: 200 OK
```

## Performance Requirements

### Response Times
- Term page load: < 800ms
- Vocabulary admin: < 600ms
- Term autocomplete: < 200ms
- Hierarchy building: < 500ms
- Tag cloud generation: < 400ms

### Database Optimization
```sql
-- Required indexes
CREATE INDEX idx_terms_vocabulary ON terms(vocabulary_id);
CREATE INDEX idx_terms_parent ON terms(parent_id);
CREATE INDEX idx_terms_slug ON terms(slug);
CREATE INDEX idx_terms_contents_content ON terms_contents(content_id);
CREATE INDEX idx_terms_contents_term ON terms_contents(term_id);
CREATE INDEX idx_terms_cache_ancestry ON terms_cache(ancestry);
```

### Caching Strategy
- Term hierarchy: 1 hour cache
- Term counts: 15 minute cache
- Tag clouds: 30 minute cache
- Autocomplete: 5 minute cache

## Edge Cases & Validation

### Term Management
- [ ] Circular parent references
- [ ] Orphaned terms
- [ ] Maximum hierarchy depth (10 levels)
- [ ] Duplicate term names in vocabulary
- [ ] Special characters in slugs
- [ ] Empty vocabulary handling

### Content Association
- [ ] Deleted term references
- [ ] Maximum terms per content
- [ ] Required vocabulary validation
- [ ] Free tagging duplicates
- [ ] Case sensitivity in tags
- [ ] Unicode term names

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] Tree behavior migration
- [ ] Counter cache updates
- [ ] Slug generation changes
- [ ] Association loading
- [ ] Finder methods
- [ ] Validation rules
- [ ] Event handlers

## Test Data Requirements

### Vocabularies
```sql
INSERT INTO vocabularies (name, machine_name, hierarchy) VALUES
('Categories', 'categories', 'multiple'),
('Tags', 'tags', 'single'),
('Product Types', 'product_types', 'multiple');
```

### Terms
```sql
INSERT INTO terms (vocabulary_id, name, slug, parent_id) VALUES
(1, 'Technology', 'technology', NULL),
(1, 'Software', 'software', 1),
(1, 'Hardware', 'hardware', 1),
(2, 'PHP', 'php', NULL),
(2, 'JavaScript', 'javascript', NULL);
```

## Success Metrics

- All vocabulary CRUD operations working
- Term hierarchy functional
- Content tagging operational
- Free tagging working
- Autocomplete functional
- Term pages displaying
- Tag clouds generating
- API endpoints working
- Performance within limits
- No data loss in migration
- Bulk operations successful
- Import/export functional