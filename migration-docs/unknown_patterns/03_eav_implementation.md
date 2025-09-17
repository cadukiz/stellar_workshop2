# EAV (Entity-Attribute-Value) Implementation

## What problem was it solving?

CMSs need flexible content structures where users can add custom fields to content types without database migrations. Standard relational databases require fixed schemas, but CMS users want to:
- Add custom fields through the admin interface
- Create different content types with different field sets
- Search and filter by custom field values
- Maintain referential integrity

## How does it work?

**Location**: `/vendor/quickapps-plugins/eav/src/Model/Behavior/EavBehavior.php`

The EAV system uses three tables:
- `eav_attributes`: Defines virtual columns/fields
- `eav_values`: Stores the actual field values
- Original entity table: Stores base entity data

```php
class EavBehavior extends Behavior
{
    // Add virtual column to entity
    public function addColumn($name, array $options = [], $errors = true)
    {
        // Validate column name doesn't conflict
        if (in_array($name, (array)$this->_table->schema()->columns())) {
            throw new FatalErrorException(__d('eav', 'Column "{0}" already exists', $name));
        }
        
        // Store virtual column definition
        $attr = TableRegistry::get('Eav.EavAttributes')->newEntity([
            'name' => $name,
            'type' => $options['type'] ?? 'string',
            'bundle' => $options['bundle'] ?? null,
            'searchable' => $options['searchable'] ?? true,
        ]);
        
        return (bool)TableRegistry::get('Eav.EavAttributes')->save($attr);
    }
    
    // Modify queries to include virtual columns
    public function beforeFind(Event $event, Query $query, ArrayObject $options, $primary)
    {
        // Get virtual columns for this bundle/entity type
        $selectedVirtual = $this->_queryScopes['SelectScope']->getVirtualColumns($query, $options['bundle']);
        
        // Join with eav_values table
        $query = $this->_scopeQuery($query, $options['bundle']);
        
        // Hydrate virtual properties into entities
        return $query->formatResults(function ($results) use ($selectedVirtual) {
            return $this->_hydrateEntities($results, $selectedVirtual);
        }, Query::PREPEND);
    }
    
    // Save virtual column values
    public function afterSave(Event $event, EntityInterface $entity, ArrayObject $options)
    {
        foreach ($entity->visibleProperties() as $property) {
            if ($this->_isVirtualColumn($property)) {
                $this->_saveVirtualProperty($entity, $property);
            }
        }
    }
}
```

## What would be the modern CakePHP 5 approach?

Modern CakePHP 5 offers better alternatives to EAV:

1. **JSON Columns**: Use native JSON column support for flexible schemas
2. **Custom Types**: Implement custom column types with proper validation
3. **Form Schemas**: Use dynamic form schemas instead of database EAV
4. **NoSQL Integration**: Consider document databases for truly flexible schemas

```php
// Modern CakePHP 5 approach using JSON columns
class FlexibleContentTable extends Table
{
    public function initialize(array $config): void
    {
        parent::initialize($config);
        
        // Use JSON column for custom fields
        $this->getSchema()->setColumnType('custom_fields', 'json');
    }
    
    public function addCustomField(string $name, string $type, array $options = []): void
    {
        // Store field definition in JSON schema table
        $schema = $this->CustomFieldSchemas->newEntity([
            'content_type' => $this->getAlias(),
            'field_name' => $name,
            'field_type' => $type,
            'options' => $options,
        ]);
        
        $this->CustomFieldSchemas->saveOrFail($schema);
        
        // Clear schema cache
        $this->getEventManager()->dispatch(new Event('Schema.updated', $this));
    }
    
    public function beforeMarshal(EventInterface $event, ArrayObject $data, ArrayObject $options)
    {
        // Validate custom fields against schema
        $schema = $this->getCustomFieldSchema($data['content_type'] ?? null);
        
        foreach ($data['custom_fields'] ?? [] as $field => $value) {
            if (!isset($schema[$field])) {
                unset($data['custom_fields'][$field]);
                continue;
            }
            
            // Apply validation and type conversion
            $data['custom_fields'][$field] = $this->convertType($value, $schema[$field]['type']);
        }
    }
}
```

## What is the migration strategy from now to a modern solution?

1. **Phase 1**: Add JSON column support alongside existing EAV tables
2. **Phase 2**: Create migration tools to move EAV data to JSON columns
3. **Phase 3**: Implement custom field validation using JSON schemas
4. **Phase 4**: Replace EAV queries with JSON path queries
5. **Phase 5**: Remove EAV tables after full migration and testing