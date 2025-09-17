# QuickApps CakePHP 3 to 5 Database Migration Issues

## Overview

This document details all database-related considerations, challenges, and issues expected during the migration of QuickApps from CakePHP 3.3.16 to CakePHP 5.x. The migration involves not only framework changes but also significant database compatibility updates from MySQL 5.7 to MySQL 8.0.

## Critical Migration Challenges

### 1. MySQL Version Migration (5.7 → 8.0)

#### SQL Mode Changes
**Issue**: MySQL 8.0 has stricter SQL modes by default compared to 5.7.

**Current Configuration (CakePHP 3)**:
```sql
--sql-mode="NO_ENGINE_SUBSTITUTION"
```

**Required Changes for MySQL 8.0**:
- Remove deprecated `NO_ENGINE_SUBSTITUTION` mode
- Handle stricter data validation
- Address potential query compatibility issues

**Impact**:
- Existing queries may fail due to stricter validation
- Data insertion/update operations may require validation fixes
- Plugin-specific queries need review

**Solution Strategy**:
```sql
-- New MySQL 8.0 configuration
SET SESSION sql_mode = 'ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
```

#### Reserved Keyword Conflicts
**Issue**: MySQL 8.0 introduces new reserved keywords that may conflict with existing column/table names.

**Potentially Affected Tables**:
- `roles` - potential conflict with new MySQL roles system
- `options` - may conflict with MySQL configuration options
- `groups` - if any custom tables use this name

**Solution**: Quote identifiers or rename conflicting columns.

#### Character Set and Collation
**Issue**: MySQL 8.0 default charset is `utf8mb4` with `utf8mb4_0900_ai_ci` collation.

**Current QuickApps Setup**: Uses `utf8mb4_unicode_ci`

**Action Required**:
- Verify all tables use consistent charset/collation
- Update connection configuration
- Test emoji and special character support

### 2. CakePHP 5 Database Layer Changes

#### ORM Improvements and Breaking Changes
**Issue**: CakePHP 5 introduces ORM improvements that may break existing model relationships.

**Affected Areas**:
- Association definitions
- Validation rules
- Behavior configurations
- Query builder syntax

**EAV System Impact**:
The complex EAV system (`eav_attributes`, `eav_values`, `field_instances`) will require significant updates:
- Dynamic model generation
- Association handling for flexible fields
- Type casting for value columns

#### Connection Configuration
**CakePHP 3 Format**:
```php
'Datasources' => [
    'default' => [
        'className' => 'Cake\Database\Connection',
        'driver' => 'Cake\Database\Driver\Mysql',
        // ...
    ]
]
```

**CakePHP 5 Updates Required**:
- Updated driver class names
- New connection pool configuration
- Enhanced SSL/security options

### 3. Schema Definition and Foreign Keys

#### Missing Foreign Key Constraints
**Issue**: QuickApps database lacks explicit foreign key constraints, relying on application-level integrity.

**Tables Requiring FK Constraints**:

```sql
-- User Management
ALTER TABLE users_roles ADD CONSTRAINT fk_users_roles_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE;
ALTER TABLE users_roles ADD CONSTRAINT fk_users_roles_role FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE;
ALTER TABLE permissions ADD CONSTRAINT fk_permissions_aco FOREIGN KEY (aco_id) REFERENCES acos(id) ON DELETE CASCADE;
ALTER TABLE permissions ADD CONSTRAINT fk_permissions_role FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE;

-- Content Management
ALTER TABLE contents ADD CONSTRAINT fk_contents_content_type FOREIGN KEY (content_type_id) REFERENCES content_types(id) ON DELETE RESTRICT;
ALTER TABLE contents ADD CONSTRAINT fk_contents_translation FOREIGN KEY (translation_for) REFERENCES contents(id) ON DELETE CASCADE;
ALTER TABLE contents ADD CONSTRAINT fk_contents_creator FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE SET NULL;
ALTER TABLE contents ADD CONSTRAINT fk_contents_modifier FOREIGN KEY (modified_by) REFERENCES users(id) ON DELETE SET NULL;
ALTER TABLE content_revisions ADD CONSTRAINT fk_revisions_content FOREIGN KEY (content_id) REFERENCES contents(id) ON DELETE CASCADE;
ALTER TABLE content_type_permissions ADD CONSTRAINT fk_ctp_content_type FOREIGN KEY (content_type_id) REFERENCES content_types(id) ON DELETE CASCADE;
ALTER TABLE content_type_permissions ADD CONSTRAINT fk_ctp_role FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE;
ALTER TABLE contents_roles ADD CONSTRAINT fk_contents_roles_content FOREIGN KEY (content_id) REFERENCES contents(id) ON DELETE CASCADE;
ALTER TABLE contents_roles ADD CONSTRAINT fk_contents_roles_role FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE;

-- EAV System
ALTER TABLE eav_values ADD CONSTRAINT fk_eav_values_attribute FOREIGN KEY (eav_attribute_id) REFERENCES eav_attributes(id) ON DELETE CASCADE;
ALTER TABLE field_instances ADD CONSTRAINT fk_field_instances_attribute FOREIGN KEY (eav_attribute_id) REFERENCES eav_attributes(id) ON DELETE CASCADE;

-- Block System
ALTER TABLE block_regions ADD CONSTRAINT fk_block_regions_block FOREIGN KEY (block_id) REFERENCES blocks(id) ON DELETE CASCADE;
ALTER TABLE blocks_roles ADD CONSTRAINT fk_blocks_roles_block FOREIGN KEY (block_id) REFERENCES blocks(id) ON DELETE CASCADE;
ALTER TABLE blocks_roles ADD CONSTRAINT fk_blocks_roles_role FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE;

-- Navigation
ALTER TABLE menu_links ADD CONSTRAINT fk_menu_links_menu FOREIGN KEY (menu_id) REFERENCES menus(id) ON DELETE CASCADE;
ALTER TABLE menu_links ADD CONSTRAINT fk_menu_links_parent FOREIGN KEY (parent_id) REFERENCES menu_links(id) ON DELETE CASCADE;

-- Taxonomy
ALTER TABLE terms ADD CONSTRAINT fk_terms_vocabulary FOREIGN KEY (vocabulary_id) REFERENCES vocabularies(id) ON DELETE CASCADE;
ALTER TABLE terms ADD CONSTRAINT fk_terms_parent FOREIGN KEY (parent_id) REFERENCES terms(id) ON DELETE CASCADE;
ALTER TABLE entities_terms ADD CONSTRAINT fk_entities_terms_term FOREIGN KEY (term_id) REFERENCES terms(id) ON DELETE CASCADE;
ALTER TABLE entities_terms ADD CONSTRAINT fk_entities_terms_field FOREIGN KEY (field_instance_id) REFERENCES field_instances(id) ON DELETE CASCADE;

-- Comments
ALTER TABLE comments ADD CONSTRAINT fk_comments_user FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL;
ALTER TABLE comments ADD CONSTRAINT fk_comments_parent FOREIGN KEY (parent_id) REFERENCES comments(id) ON DELETE CASCADE;
```

#### Index Optimization
**Issue**: Some tables lack proper indexing for performance.

**Required Indexes**:
```sql
-- Performance indexes for large tables
CREATE INDEX idx_eav_values_entity_attribute ON eav_values(entity_id, eav_attribute_id);
CREATE INDEX idx_eav_values_attribute_type ON eav_values(eav_attribute_id, entity_id);
CREATE INDEX idx_search_datasets_entity ON search_datasets(table_alias, entity_id);
CREATE INDEX idx_contents_type_status ON contents(content_type_slug, status, created);
CREATE INDEX idx_contents_language ON contents(language, status);

-- Nested sets performance
CREATE INDEX idx_acos_lft_rght ON acos(lft, rght);
CREATE INDEX idx_menu_links_lft_rght ON menu_links(lft, rght);
CREATE INDEX idx_terms_lft_rght ON terms(lft, rght);
CREATE INDEX idx_comments_lft_rght ON comments(lft, rght);

-- Full-text search indexes
CREATE FULLTEXT INDEX idx_search_words ON search_datasets(words);
```

### 4. Data Type and Field Compatibility Issues

#### Blob Field Serialization
**Issue**: Many tables use `blob` fields to store serialized PHP data.

**Affected Tables and Fields**:
- `content_types.defaults`
- `blocks.settings`
- `menus.settings`
- `field_instances.settings`
- `field_instances.view_modes`
- `eav_values.extra`
- `plugins.settings`

**Migration Challenges**:
- PHP serialization format changes
- Data integrity during unserialization
- Potential encoding issues
- Security concerns with serialized data

**Solution Strategy**:
1. Convert serialized data to JSON format
2. Implement data validation during migration
3. Update application code to handle JSON instead of serialized data

#### Date/Time Field Handling
**Issue**: CakePHP 5 has stricter datetime handling.

**Affected Fields**:
- All `datetime` fields with `NULL` or `0000-00-00 00:00:00` values
- Timezone handling changes

**Required Updates**:
```sql
-- Clean up invalid datetime values
UPDATE contents SET created = NOW() WHERE created = '0000-00-00 00:00:00';
UPDATE contents SET modified = NOW() WHERE modified = '0000-00-00 00:00:00';
UPDATE users SET last_login = NULL WHERE last_login = '0000-00-00 00:00:00';
```

#### Text Field Size Limits
**Issue**: MySQL 8.0 has different text field size handling.

**Potential Issues**:
- `longtext` fields in `blocks.body` and `search_datasets.words`
- `text` fields may need size specification

### 5. EAV System Migration Complexity

#### Multi-Value Storage Challenge
**Issue**: The EAV system stores different data types in separate columns.

**Storage Pattern**:
```php
// Current EAV value storage
$eavValue = [
    'value_string' => 'text data',
    'value_integer' => 123,
    'value_datetime' => '2023-01-01 12:00:00',
    'value_boolean' => 1,
    // ... other type-specific columns
];
```

**Migration Challenges**:
- Type conversion logic
- NULL value handling across type columns
- Query optimization for type-specific searches
- Plugin compatibility with new ORM

#### Dynamic Field Relationships
**Issue**: EAV fields create runtime relationships between entities.

**Complex Queries Example**:
```sql
-- Current complex EAV query pattern
SELECT c.*, ev.value_string as field_value
FROM contents c
JOIN eav_values ev ON ev.entity_id = c.id
JOIN eav_attributes ea ON ea.id = ev.eav_attribute_id
JOIN field_instances fi ON fi.eav_attribute_id = ea.id
WHERE ea.table_alias = 'contents'
  AND ea.name = 'custom_field'
  AND c.content_type_slug = 'article';
```

**CakePHP 5 ORM Adaptations Required**:
- Association definitions for dynamic fields
- Query builder adaptations
- Caching strategies for field definitions

### 6. Nested Sets Model Migration

#### Tree Structure Integrity
**Issue**: Four tables use nested sets model for hierarchical data.

**Affected Tables**:
- `acos` (ACL hierarchy)
- `menu_links` (menu structure)
- `terms` (taxonomy hierarchy)
- `comments` (comment threading)

**Migration Risks**:
- Left/right boundary corruption during migration
- Performance impact of tree rebuilding
- Data consistency across large hierarchies

**Validation Required**:
```sql
-- Validate nested sets integrity
SELECT 
    table_name,
    COUNT(*) as total_nodes,
    COUNT(DISTINCT lft) as unique_left,
    COUNT(DISTINCT rght) as unique_right,
    MAX(rght) as max_right
FROM (
    SELECT 'acos' as table_name, lft, rght FROM acos
    UNION ALL
    SELECT 'menu_links', lft, rght FROM menu_links
    UNION ALL
    SELECT 'terms', lft, rght FROM terms
    UNION ALL
    SELECT 'comments', lft, rght FROM comments
) nested_data
GROUP BY table_name;
```

### 7. Plugin System Database Dependencies

#### Plugin-Specific Tables
**Issue**: Core plugins may create additional database tables not documented in core schema.

**Investigation Required**:
- Check for plugin-specific tables in `/vendor/quickapps-plugins/*/config/schema/`
- Verify plugin database migrations
- Identify cross-plugin table dependencies

#### Dynamic Table Creation
**Issue**: Some plugins may create tables dynamically.

**Examples**:
- Custom content type tables
- Plugin-specific caching tables
- Temporary data storage

### 8. Search System Migration

#### Full-Text Search Compatibility
**Issue**: `search_datasets` table requires MySQL full-text search updates.

**MySQL 8.0 Full-Text Changes**:
- Improved full-text search capabilities
- New search operators and functions
- Performance optimizations

**Required Updates**:
```sql
-- Update full-text search configuration
ALTER TABLE search_datasets ADD FULLTEXT(words);

-- Test full-text search functionality
SELECT * FROM search_datasets 
WHERE MATCH(words) AGAINST('search term' IN NATURAL LANGUAGE MODE);
```

### 9. Performance and Scalability Issues

#### Large Table Migration
**Issue**: Some tables may be very large in production environments.

**Potentially Large Tables**:
- `eav_values` - Can grow extremely large with many custom fields
- `search_datasets` - Contains indexed content from all entities
- `content_revisions` - Stores all content history
- `comments` - User-generated content

**Migration Strategies**:
1. **Chunked Migration**: Process large tables in batches
2. **Parallel Processing**: Migrate independent tables simultaneously
3. **Incremental Migration**: Migrate with minimal downtime
4. **Data Archival**: Archive old data before migration

#### Query Optimization
**Issue**: CakePHP 5 query patterns may differ from CakePHP 3.

**Areas Requiring Optimization**:
- EAV queries with multiple joins
- Nested sets traversal queries
- Full-text search queries
- Complex content filtering

### 10. Data Integrity and Validation

#### Referential Integrity Issues
**Issue**: Current system relies on application-level constraints.

**Data Cleanup Required Before Migration**:
```sql
-- Find orphaned records that will prevent FK creation
SELECT 'users_roles' as table_name, COUNT(*) as orphans 
FROM users_roles ur 
LEFT JOIN users u ON u.id = ur.user_id 
WHERE u.id IS NULL

UNION ALL

SELECT 'contents', COUNT(*) 
FROM contents c 
LEFT JOIN content_types ct ON ct.id = c.content_type_id 
WHERE ct.id IS NULL

UNION ALL

SELECT 'eav_values', COUNT(*) 
FROM eav_values ev 
LEFT JOIN eav_attributes ea ON ea.id = ev.eav_attribute_id 
WHERE ea.id IS NULL;
```

#### Data Format Validation
**Issue**: Inconsistent data formats may cause migration failures.

**Validation Required**:
- Email format validation in `users.email`
- URL format validation in `menu_links.url`
- Date format consistency
- JSON/serialized data integrity

## Migration Strategy and Timeline

### Phase 1: Database Schema Preparation (Week 1-2)
1. **Schema Analysis**: Complete current documentation
2. **Constraint Addition**: Add foreign key constraints
3. **Index Optimization**: Create performance indexes
4. **Data Cleanup**: Fix orphaned records and invalid data

### Phase 2: MySQL 8.0 Upgrade (Week 2-3)
1. **Test Environment**: Upgrade test database to MySQL 8.0
2. **Compatibility Testing**: Test all queries and operations
3. **Performance Benchmarking**: Compare performance metrics
4. **Configuration Tuning**: Optimize MySQL 8.0 settings

### Phase 3: CakePHP 5 ORM Migration (Week 3-5)
1. **Model Updates**: Migrate all model classes
2. **Association Fixes**: Update relationship definitions
3. **EAV System**: Redesign flexible field system
4. **Query Optimization**: Update complex queries

### Phase 4: Data Migration and Testing (Week 5-6)
1. **Data Migration**: Execute full data migration
2. **Integrity Testing**: Validate all relationships
3. **Performance Testing**: Ensure acceptable performance
4. **User Acceptance**: Test all functionality

### Phase 5: Production Deployment (Week 6-7)
1. **Backup Strategy**: Complete database backup
2. **Migration Execution**: Execute production migration
3. **Monitoring**: Monitor system performance
4. **Rollback Planning**: Prepare rollback procedures

## Risk Assessment and Mitigation

### High Risk Items
1. **EAV System Complexity**: Most complex migration component
   - **Mitigation**: Extensive testing, gradual migration
2. **Nested Sets Integrity**: Critical for site navigation
   - **Mitigation**: Backup and validation scripts
3. **Plugin Dependencies**: Unknown third-party requirements
   - **Mitigation**: Comprehensive plugin audit

### Medium Risk Items
1. **Performance Degradation**: Potential slowdowns
   - **Mitigation**: Performance benchmarking and optimization
2. **Data Loss**: Incomplete migration
   - **Mitigation**: Multiple backup strategies

### Low Risk Items
1. **User Interface**: Database changes shouldn't affect UI
2. **Basic CRUD**: Simple operations should migrate cleanly

## Conclusion

The database migration from QuickApps CakePHP 3 to CakePHP 5 involves significant complexity due to:
- MySQL version upgrade requirements
- Complex EAV system architecture
- Nested sets model dependencies
- Plugin system database integration

Success requires careful planning, extensive testing, and phased implementation with proper backup and rollback strategies. The migration timeline should allow for thorough testing at each phase to ensure data integrity and system functionality.