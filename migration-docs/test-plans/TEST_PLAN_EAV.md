# Test Plan: EAV (Entity-Attribute-Value) System

## Feature Overview
The EAV plugin provides the Entity-Attribute-Value data model that enables dynamic field creation and flexible content structure throughout QuickAppsCMS. This is the foundation for custom fields, content types, and user profiles.

## File Locations

### Models & Tables
- `src/vendor/quickapps-plugins/eav/src/Model/Table/EavAttributesTable.php` - Attribute definitions
- `src/vendor/quickapps-plugins/eav/src/Model/Table/EavValuesTable.php` - Dynamic values storage
- `src/vendor/quickapps-plugins/eav/src/Model/Entity/EavAttribute.php` - Attribute entity
- `src/vendor/quickapps-plugins/eav/src/Model/Entity/EavValue.php` - Value entity
- `src/vendor/quickapps-plugins/eav/src/Model/Entity/CachedColumn.php` - Performance optimization

### Behaviors
- `src/vendor/quickapps-plugins/eav/src/Model/Behavior/EavBehavior.php` - EAV functionality for tables

### Database Tables
- `eav_attributes` - Field attribute definitions
- `eav_values` - Dynamic field values
- `eav_cache` - Performance cache for queries

### Utility Classes
- `src/vendor/quickapps-plugins/eav/src/Utility/EavQueryBuilder.php` - Complex EAV queries
- `src/vendor/quickapps-plugins/eav/src/Utility/AttributeCache.php` - Attribute caching

## Database Schema

### EAV Attributes Table
```sql
CREATE TABLE eav_attributes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    table_alias VARCHAR(100) NOT NULL,
    bundle VARCHAR(100),
    name VARCHAR(100) NOT NULL,
    type VARCHAR(50) NOT NULL,
    searchable BOOLEAN DEFAULT 0,
    sortable BOOLEAN DEFAULT 0,
    settings TEXT,
    created DATETIME,
    modified DATETIME
);
```

### EAV Values Table
```sql
CREATE TABLE eav_values (
    id INT PRIMARY KEY AUTO_INCREMENT,
    eav_attribute_id INT NOT NULL,
    entity_id INT NOT NULL,
    table_alias VARCHAR(100) NOT NULL,
    value_text TEXT,
    value_int INT,
    value_float DECIMAL(10,4),
    value_datetime DATETIME,
    value_boolean BOOLEAN,
    created DATETIME,
    modified DATETIME,
    FOREIGN KEY (eav_attribute_id) REFERENCES eav_attributes(id)
);
```

## Test Scenarios

### 1. Attribute Management
```gherkin
Feature: EAV Attribute Management
  As a system administrator
  I want to manage dynamic attributes
  To create flexible content structures

  Scenario: Create Text Attribute
    Given I need a custom text field
    When I create attribute:
      | Field | Value |
      | Table | contents |
      | Bundle | article |
      | Name | subtitle |
      | Type | varchar |
      | Searchable | Yes |
      | Max Length | 255 |
    Then attribute should be created
    And available for content type

  Scenario: Create Number Attribute
    Given I need a numeric field
    When I create attribute:
      | Field | Value |
      | Name | price |
      | Type | decimal |
      | Precision | 2 |
      | Min Value | 0 |
      | Max Value | 99999.99 |
    Then attribute should validate numeric input
    And store decimal values correctly

  Scenario: Create Date Attribute
    Given I need a date field
    When I create attribute:
      | Field | Value |
      | Name | event_date |
      | Type | datetime |
      | Required | Yes |
      | Format | Y-m-d H:i:s |
    Then attribute should validate dates
    And store datetime values

  Scenario: Create Reference Attribute
    Given I need to reference other entities
    When I create attribute:
      | Field | Value |
      | Name | related_content |
      | Type | entity_reference |
      | Target Table | contents |
      | Multiple Values | Yes |
    Then attribute should allow entity references
    And maintain referential integrity

  Scenario: Attribute Validation
    Given attribute has validation rules
    When invalid data is provided:
      | Text too long | Number out of range |
      | Invalid date format | Broken reference |
    Then validation should prevent save
    And return appropriate error messages
```

### 2. Value Storage & Retrieval
```gherkin
Feature: EAV Value Management
  As a content editor
  I want to store and retrieve field values
  To manage dynamic content

  Scenario: Store Single Values
    Given entity with EAV attributes
    When I save values:
      | subtitle | "Breaking News Story" |
      | price | 99.99 |
      | event_date | "2024-12-25 18:00:00" |
      | featured | true |
    Then values should be stored in eav_values
    And retrievable by attribute name

  Scenario: Store Multiple Values
    Given attribute allows multiple values
    When I save array of values:
      | tags | ["tech", "news", "breaking"] |
      | related_ids | [1, 5, 12] |
    Then each value should be stored separately
    And retrieved as array

  Scenario: Update Existing Values
    Given entity has existing EAV values
    When I update values:
      | Change price from 99.99 to 89.99 |
      | Add new tag "urgent" |
      | Remove tag "breaking" |
    Then values should be updated
    And change history maintained

  Scenario: Delete Values
    Given entity has EAV values
    When entity is deleted
    Then associated EAV values should cascade delete
    And orphaned data cleaned up

  Scenario: Bulk Operations
    Given 1000 entities with EAV values
    When performing bulk update:
      | Update all prices by 10% |
      | Add category to all items |
    Then bulk operation should be efficient
    And complete within time limit
```

### 3. Query Performance & Optimization
```gherkin
Feature: EAV Query Performance
  As a system
  I want efficient EAV queries
  To maintain performance

  Scenario: Simple Value Lookup
    Given entity needs its EAV values
    When loading entity
    Then values should load efficiently:
      | Single query for all values |
      | Proper index usage |
      | < 50ms response time |

  Scenario: Filtered EAV Queries
    Given I need entities by field values
    When I query:
      | Price between 50 and 100 |
      | Category equals "electronics" |
      | Created in last 7 days |
    Then query should execute efficiently:
      | Use appropriate indexes |
      | Avoid full table scans |
      | Return results < 200ms |

  Scenario: Complex Multi-Field Queries
    Given complex search criteria
    When I search for entities where:
      | Price > 100 AND category = "premium" |
      | Tags contains "featured" |
      | Date within next month |
    Then query should be optimized:
      | Join tables efficiently |
      | Use index intersections |
      | Complete within 500ms |

  Scenario: Sorting by EAV Fields
    Given entities need sorting by EAV values
    When I sort by:
      | Price ascending |
      | Date descending |
      | Custom text field |
    Then sorting should work efficiently:
      | Use index for ordering |
      | Handle NULL values correctly |
      | Maintain performance |

  Scenario: Pagination with EAV
    Given large result set with EAV data
    When paginating results
    Then pagination should be efficient:
      | Count queries optimized |
      | Stable sort order |
      | Consistent page sizes |
```

### 4. Data Type Handling
```gherkin
Feature: EAV Data Types
  As a developer
  I want proper data type handling
  To ensure data integrity

  Scenario: String Data Types
    Given string-based attributes
    When storing text values:
      | varchar(255) | text | longtext |
    Then data should be stored correctly:
      | Proper encoding (UTF-8) |
      | Length validation |
      | Special character handling |

  Scenario: Numeric Data Types
    Given numeric attributes
    When storing numbers:
      | integers | floats | decimals |
    Then data should be stored precisely:
      | No precision loss |
      | Proper rounding |
      | Range validation |

  Scenario: Boolean Data Types
    Given boolean attributes
    When storing true/false values:
      | true/false | 1/0 | yes/no |
    Then values should normalize:
      | Convert to boolean |
      | Handle string inputs |
      | Default value support |

  Scenario: Date/Time Data Types
    Given datetime attributes
    When storing dates:
      | Timestamps | Dates only | Times only |
    Then data should be stored correctly:
      | Timezone handling |
      | Format validation |
      | NULL date support |

  Scenario: JSON/Array Data Types
    Given complex data attributes
    When storing structured data:
      | JSON objects | Arrays | Nested structures |
    Then data should serialize correctly:
      | Valid JSON format |
      | Searchable elements |
      | Type preservation |
```

### 5. Caching & Performance
```gherkin
Feature: EAV Caching System
  As a performance optimization
  I want intelligent caching
  To speed up EAV operations

  Scenario: Attribute Definition Caching
    Given attribute definitions don't change often
    When system loads attributes
    Then definitions should be cached:
      | Memory cache for requests |
      | File cache for persistence |
      | Cache invalidation on change |

  Scenario: Value Caching
    Given frequently accessed EAV values
    When values are requested
    Then caching should optimize:
      | Cache entity value sets |
      | Group related values |
      | Intelligent cache keys |

  Scenario: Query Result Caching
    Given complex EAV queries
    When identical queries are executed
    Then results should be cached:
      | Cache result sets |
      | Include query parameters |
      | Time-based expiration |

  Scenario: Cache Invalidation
    Given cached EAV data exists
    When underlying data changes
    Then cache should invalidate:
      | Entity-specific invalidation |
      | Attribute-based clearing |
      | Bulk operation handling |

  Scenario: Cache Performance
    Given caching system is active
    When measuring performance
    Then cache should provide:
      | > 80% cache hit rate |
      | < 10ms cache lookup |
      | Memory usage < 100MB |
```

## API Testing

### EAV Management API
```javascript
// GET /api/eav/attributes/{table}/{bundle}
// Response: Attributes for specific content type

{
  "attributes": [
    {
      "id": 1,
      "name": "price",
      "type": "decimal",
      "settings": {"precision": 2, "min": 0},
      "searchable": true,
      "sortable": true
    }
  ]
}

// POST /api/eav/attributes
{
  "table_alias": "contents",
  "bundle": "product",
  "name": "sku",
  "type": "varchar",
  "settings": {"max_length": 50, "required": true}
}
// Response: 201 Created with attribute details

// GET /api/eav/values/{entity_id}
// Response: All EAV values for entity

{
  "entity_id": 123,
  "values": {
    "price": 99.99,
    "sku": "PROD-001",
    "tags": ["electronics", "smartphone"],
    "featured": true
  }
}

// PUT /api/eav/values/{entity_id}
{
  "values": {
    "price": 89.99,
    "description": "Updated description"
  }
}
// Response: 200 OK

// POST /api/eav/query
{
  "table_alias": "contents",
  "conditions": {
    "price": {">=": 50, "<=": 100},
    "category": "electronics"
  },
  "sort": [{"price": "asc"}],
  "limit": 20
}
// Response: Filtered entities with EAV data
```

## Performance Requirements

### Response Times
- Single entity EAV load: < 50ms
- Complex EAV query (10 fields): < 200ms
- Bulk EAV update (100 entities): < 5 seconds
- Attribute definition load: < 10ms (cached)

### Database Performance
```sql
-- Critical EAV indexes for performance
CREATE INDEX idx_eav_values_entity ON eav_values(entity_id, table_alias);
CREATE INDEX idx_eav_values_attribute ON eav_values(eav_attribute_id);
CREATE INDEX idx_eav_values_text ON eav_values(value_text(50));
CREATE INDEX idx_eav_values_int ON eav_values(value_int);
CREATE INDEX idx_eav_values_float ON eav_values(value_float);
CREATE INDEX idx_eav_values_datetime ON eav_values(value_datetime);
CREATE INDEX idx_eav_attributes_bundle ON eav_attributes(table_alias, bundle);
```

### Memory Usage
- EAV behavior overhead: < 2MB per table
- Attribute cache: < 10MB total
- Value cache: < 50MB for 1000 entities
- Query cache: < 20MB for complex queries

### Caching Strategy
- Attribute definitions: Permanent until change
- Entity values: 1 hour default cache
- Query results: 15 minute cache
- Count queries: 5 minute cache

## Edge Cases & Validation

### Data Integrity
- [ ] Orphaned EAV values
- [ ] Missing attribute references
- [ ] Type conversion errors
- [ ] Large text values (>64KB)
- [ ] Unicode handling
- [ ] NULL vs empty value distinction

### Performance Issues
- [ ] Deep EAV queries (>10 fields)
- [ ] Large result sets (>10,000 entities)
- [ ] Complex JOIN operations
- [ ] Index fragmentation
- [ ] Memory usage with large datasets

### Concurrency Issues
- [ ] Simultaneous attribute creation
- [ ] Concurrent value updates
- [ ] Cache coherence
- [ ] Deadlock prevention
- [ ] Transaction isolation

## Security Testing

### Data Protection
- [ ] EAV value sanitization
- [ ] SQL injection via dynamic queries
- [ ] Access control for attributes
- [ ] Data encryption for sensitive fields
- [ ] Audit logging for changes

### Query Security
- [ ] Dynamic query injection
- [ ] Parameter validation
- [ ] Result set access control
- [ ] Information disclosure prevention

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] EAV behavior compatibility
- [ ] Query builder changes
- [ ] Entity accessor methods
- [ ] Association handling
- [ ] Validation system migration
- [ ] Event system updates

### Data Migration
- [ ] EAV schema preservation
- [ ] Value data integrity
- [ ] Index recreation
- [ ] Performance optimization
- [ ] Relationship maintenance

## Test Data Requirements

### Sample Attributes
```sql
INSERT INTO eav_attributes (table_alias, bundle, name, type, settings) VALUES
('contents', 'product', 'price', 'decimal', '{"precision": 2, "min": 0}'),
('contents', 'product', 'sku', 'varchar', '{"max_length": 50, "required": true}'),
('contents', 'article', 'subtitle', 'varchar', '{"max_length": 255}'),
('users', 'user', 'phone', 'varchar', '{"max_length": 20}');
```

### Sample Values
```sql
INSERT INTO eav_values (eav_attribute_id, entity_id, table_alias, value_decimal, value_text) VALUES
(1, 1, 'contents', 99.99, NULL),
(2, 1, 'contents', NULL, 'PROD-001'),
(3, 2, 'contents', NULL, 'Breaking News Subtitle'),
(4, 1, 'users', NULL, '+1-555-0123');
```

### Performance Test Data
- 1,000 entities with 10 EAV fields each
- 10,000 EAV values total
- Multiple data types represented
- Complex query scenarios

## Success Metrics

### Functional Success
- All EAV operations working correctly
- Data integrity maintained
- Type conversion functioning
- Query performance acceptable
- Caching system operational

### Performance Success
- Response times within benchmarks
- Database queries optimized
- Memory usage controlled
- Cache hit rates > 80%
- Bulk operations efficient

### Security Success
- Input validation working
- Access control enforced
- No SQL injection vulnerabilities
- Data encryption functional
- Audit logging active

### Migration Success
- All EAV data preserved
- Performance equal or better
- Functionality fully restored
- Zero data corruption
- System stability maintained