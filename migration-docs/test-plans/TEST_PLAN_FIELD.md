# Test Plan: Field System & EAV

## Feature Overview
The Field plugin provides the dynamic field system using Entity-Attribute-Value (EAV) model for custom fields, field types, validation, and rendering.

## File Locations

### Controllers
- Field management integrated into content type controllers
- Custom field configuration through admin interfaces

### Models
- `src/vendor/quickapps-plugins/field/src/Model/Table/EavAttributesTable.php`
- `src/vendor/quickapps-plugins/field/src/Model/Table/EavValuesTable.php`
- `src/vendor/quickapps-plugins/field/src/Model/Entity/EavAttribute.php`
- `src/vendor/quickapps-plugins/field/src/Model/Entity/EavValue.php`

### Field Types
- `src/vendor/quickapps-plugins/field/src/Field/` - Core field types
- Each field type includes:
  - Handler class
  - View helper
  - Templates
  - Configuration form

### Database Tables
- `eav_attributes` - Field definitions
- `eav_values` - Field values storage
- `field_instances` - Field instance configuration

## Core Field Types

### Text Fields
- Text (single line)
- Textarea (multi-line)
- Text with Summary
- Formatted Text (WYSIWYG)

### Numeric Fields
- Integer
- Decimal
- Float
- Currency

### Selection Fields
- Boolean (checkbox)
- List (select dropdown)
- Radio buttons
- Checkboxes

### Date/Time Fields
- Date
- DateTime
- Time
- Date Range

### Reference Fields
- Entity Reference
- User Reference
- Taxonomy Reference
- File/Image Reference

### Advanced Fields
- Email
- URL
- Phone
- Color Picker
- Geolocation

## Test Scenarios

### 1. Field Type Management
```gherkin
Feature: Field Types
  As an administrator
  I want to manage field types
  To customize content structure

  Scenario: View Available Field Types
    Given I am adding field to content type
    When I view field type options
    Then I should see all available types:
      | Text | Number | Date | File |
      | List | Boolean | Email | URL |
    With descriptions and capabilities

  Scenario: Configure Text Field
    Given I am adding text field
    When I configure:
      | Label | Product Name |
      | Machine Name | field_product_name |
      | Help Text | Enter product name |
      | Required | Yes |
      | Max Length | 255 |
      | Default Value | New Product |
    Then field should be created
    And validation rules should apply

  Scenario: Configure Selection Field
    Given I am adding list field
    When I configure options:
      | Option 1 | Small |
      | Option 2 | Medium |
      | Option 3 | Large |
    And set multiple selection: No
    Then dropdown should display options
    And save selected value

  Scenario: Configure File Field
    Given I am adding file field
    When I configure:
      | Allowed extensions | jpg, png, gif |
      | Maximum file size | 2MB |
      | Upload location | /files/products |
      | Alt text required | Yes |
    Then field should validate uploads
    And store files correctly
```

### 2. Field Instance Configuration
```gherkin
Feature: Field Instances
  As an administrator
  I want to configure field instances
  Per content type

  Scenario: Add Field to Content Type
    Given content type "Product" exists
    When I add field "Price" of type "Decimal"
    And configure instance settings:
      | Required | Yes |
      | Default | 0.00 |
      | Precision | 2 |
      | Prefix | $ |
    Then field should appear on product forms
    And validation should work

  Scenario: Field Display Settings
    Given field is attached to content type
    When I configure display modes:
      | Full | Show label and value |
      | Teaser | Show value only |
      | Search | Hide field |
    Then field should render accordingly
    In different view modes

  Scenario: Field Widget Settings
    Given field type supports widgets
    When I select widget type:
      | Text field | Single line input |
      | Textarea | Multi-line input |
      | Autocomplete | With suggestions |
    Then appropriate widget should display
    On content forms

  Scenario: Remove Field from Content Type
    Given field is attached to content type
    When I remove the field
    Then field definition should remain
    But instance should be deleted
    And existing data should be preserved
```

### 3. EAV Data Storage & Retrieval
```gherkin
Feature: EAV Storage
  As a system
  I want to store field data efficiently
  Using EAV model

  Scenario: Store Field Values
    Given content with custom fields:
      | Title | Sample Product |
      | Price | 99.99 |
      | Category | Electronics |
      | Tags | [gadget, tech, new] |
    When content is saved
    Then values should be stored in EAV tables
    With proper data types and indexes

  Scenario: Retrieve Field Values
    Given content has EAV field data
    When content is loaded
    Then field values should be populated
    And available as entity properties
    With proper type casting

  Scenario: Query by Field Values
    Given I need to find content by field values
    When I search for:
      | Price between 50 and 100 |
      | Category equals "Electronics" |
      | Tags contains "tech" |
    Then EAV queries should execute
    And return matching content

  Scenario: Update Field Values
    Given content exists with field data
    When I update field values:
      | Price from 99.99 to 89.99 |
      | Add tag "sale" |
    Then EAV values should update
    And version history maintained

  Scenario: Delete Field Values
    Given field instance is removed
    When cleanup process runs
    Then orphaned EAV values should be deleted
    And database should be optimized
```

### 4. Field Validation
```gherkin
Feature: Field Validation
  As a content editor
  I want field validation
  To ensure data quality

  Scenario: Required Field Validation
    Given field is marked required
    When I try to save without value
    Then validation error should appear
    And form should not submit

  Scenario: Data Type Validation
    Given numeric field configured
    When I enter non-numeric value
    Then validation should fail
    With appropriate error message

  Scenario: Length Validation
    Given text field with max length 50
    When I enter 60 characters
    Then validation should prevent save
    And show character count

  Scenario: Format Validation
    Given email field configured
    When I enter invalid email format
    Then validation should fail
    With format requirement message

  Scenario: Custom Validation Rules
    Given field has custom validation:
      | Must start with uppercase |
      | Cannot contain numbers |
    When I enter invalid value
    Then custom validation should run
    And show specific error

  Scenario: Conditional Validation
    Given field validation depends on other field
    When condition is met
    Then validation rules should apply
    Otherwise field should be optional
```

### 5. Field Rendering & Display
```gherkin
Feature: Field Display
  As a site visitor
  I want to see field data
  Properly formatted

  Scenario: Basic Field Display
    Given content with field values
    When I view the content
    Then fields should display:
      | With appropriate labels |
      | Formatted for data type |
      | In configured order |
      | Respecting permissions |

  Scenario: Field Formatters
    Given field supports multiple formatters
    When I configure display:
      | Date field | Long format |
      | Number field | Currency format |
      | Text field | Trimmed to 100 chars |
    Then values should format correctly

  Scenario: Empty Field Handling
    Given field has no value
    When displayed on frontend
    Then should either:
      | Show default value |
      | Hide field completely |
      | Show empty state message |
    Based on configuration

  Scenario: Multi-value Fields
    Given field allows multiple values
    When displaying field data
    Then all values should show
    With proper separators
    And respecting max display count

  Scenario: Responsive Field Display
    Given field displayed on mobile
    When screen size is small
    Then field should adapt:
      | Stack vertically |
      | Truncate long values |
      | Show expand option |
```

## API Testing

### Field API Endpoints
```javascript
// GET /api/field-types
// Response: Available field types with capabilities

// GET /api/content-types/{type}/fields
// Response: Fields attached to content type

// POST /api/content-types/{type}/fields
{
  "field_type": "text",
  "label": "Product Name",
  "machine_name": "field_product_name",
  "required": true,
  "settings": {
    "max_length": 255,
    "default_value": ""
  }
}
// Response: 201 Created

// PUT /api/fields/{id}/instance
{
  "label": "Updated Label",
  "required": false,
  "weight": 10
}
// Response: 200 OK

// DELETE /api/fields/{id}/instance/{content_type}
// Response: 204 No Content

// GET /api/content/{id}/field-values
// Response: All field values for content

// PUT /api/content/{id}/field-values
{
  "field_price": 99.99,
  "field_category": "electronics",
  "field_tags": ["tag1", "tag2"]
}
// Response: 200 OK
```

## Performance Requirements

### EAV Query Performance
- Field value lookup: < 50ms
- Multi-field queries: < 200ms
- Content with 20 fields: < 100ms load time
- Field search queries: < 500ms

### Database Optimization
```sql
-- Critical EAV indexes
CREATE INDEX idx_eav_values_entity ON eav_values(entity_id, table_alias);
CREATE INDEX idx_eav_values_attribute ON eav_values(eav_attribute_id);
CREATE INDEX idx_eav_values_value ON eav_values(value_text(50), value_int, value_float);
CREATE INDEX idx_eav_attributes_bundle ON eav_attributes(table_alias, bundle);
```

### Caching Strategy
- Field definitions: Permanent cache
- EAV attribute metadata: 1 hour cache  
- Field rendering: Page cache
- Validation rules: Memory cache

## Edge Cases & Validation

### EAV Data Handling
- [ ] Orphaned EAV values
- [ ] Large text values (>64KB)
- [ ] Unicode field values
- [ ] NULL vs empty values
- [ ] Data type conversion
- [ ] Circular field references

### Field Configuration
- [ ] Duplicate machine names
- [ ] Invalid field types
- [ ] Missing field handlers
- [ ] Corrupted field settings
- [ ] Version conflicts

### Performance Issues
- [ ] Deep EAV queries
- [ ] Large result sets
- [ ] Missing indexes
- [ ] Inefficient joins
- [ ] Memory usage with many fields

## Security Testing

### Field Input Security
- [ ] XSS in field values
- [ ] SQL injection via field queries
- [ ] File upload validation
- [ ] HTML sanitization
- [ ] Script injection prevention

### Field Access Control
- [ ] Field-level permissions
- [ ] Edit access validation
- [ ] View access restrictions
- [ ] API endpoint security

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] EAV behavior migration
- [ ] Field type compatibility
- [ ] Validation rule updates
- [ ] Form helper changes
- [ ] Entity accessor methods
- [ ] Query builder updates

## Test Data Requirements

### EAV Attributes
```sql
INSERT INTO eav_attributes (table_alias, bundle, name, type, settings) VALUES
('contents', 'product', 'price', 'decimal', '{"precision": 2}'),
('contents', 'product', 'sku', 'varchar', '{"max_length": 50}'),
('contents', 'article', 'author_bio', 'text', '{}'),
('users', 'user', 'phone', 'varchar', '{"max_length": 20}');
```

### EAV Values
```sql
INSERT INTO eav_values (eav_attribute_id, entity_id, table_alias, value_decimal, value_text) VALUES
(1, 1, 'contents', 99.99, NULL),
(2, 1, 'contents', NULL, 'PROD-001'),
(3, 2, 'contents', NULL, 'Author biography text'),
(4, 1, 'users', NULL, '+1-555-0123');
```

## Success Metrics

- All field types functional
- EAV storage working correctly
- Field validation enforced
- Display modes rendering properly
- Multi-value fields operational
- Performance within benchmarks
- No data corruption
- API endpoints working
- Migration successful
- Zero field data loss
- Custom field types supported
- Complex queries executing efficiently