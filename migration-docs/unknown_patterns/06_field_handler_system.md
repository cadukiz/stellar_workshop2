# Pluggable Field Handler System

## What problem was it solving?

CMSs need extensible field types (text, image, date, etc.) that can be:
- Added through plugins
- Attached to any content type
- Have custom validation, rendering, and storage logic
- Provide admin configuration interfaces
- Handle complex data types and relationships

## How does it work?

**Location**: `/vendor/quickapps-plugins/field/src/Handler.php`

Each field type extends an abstract Handler class:

```php
abstract class Handler
{
    // Field metadata and capabilities
    public function info()
    {
        return [
            'type' => 'varchar',
            'name' => 'Field Name',
            'description' => 'Field description',
            'hidden' => false,
            'maxInstances' => 0, // 0 = unlimited
            'searchable' => true,
        ];
    }
    
    // Rendering methods for different contexts
    public function render(Field $field, View $view) { return ''; }
    public function edit(Field $field, View $view) { return ''; }
    public function settings(FieldInstance $instance, View $view) { return ''; }
    
    // Validation and data handling
    public function validate(Field $field, Validator $validator) { return true; }
    public function beforeSave(Field $field, $post) { return true; }
    public function afterSave(Field $field) {}
    
    // Lifecycle hooks
    public function fieldAttached(Field $field) {}
    public function beforeFind(Field $field, array $options, $primary) { return true; }
    public function beforeDelete(Field $field) { return true; }
    public function afterDelete(Field $field) {}
    
    // Configuration
    public function defaultSettings(FieldInstance $instance) { return []; }
    public function validateSettings(FieldInstance $instance, array $settings, Validator $validator) {}
}

// Example implementation - TextField
class TextFieldHandler extends Handler
{
    public function info()
    {
        return [
            'type' => 'varchar',
            'name' => __d('field', 'Text Field'),
            'description' => __d('field', 'Allows to store short text'),
            'maxInstances' => 1,
        ];
    }
    
    public function render(Field $field, View $view)
    {
        return $view->element('Field.TextField/display', compact('field'));
    }
    
    public function edit(Field $field, View $view)
    {
        return $view->Form->input($field->name, [
            'type' => 'text',
            'value' => $field->value,
            'maxlength' => $field->metadata->settings['max_len'],
        ]);
    }
}
```

## What would be the modern CakePHP 5 approach?

Modern CakePHP 5 would use more advanced patterns:

1. **Form Field Objects**: Use dedicated field classes with validation
2. **Type System**: Leverage CakePHP 5's enhanced type system
3. **Component System**: Use composition over inheritance
4. **Attribute-Based Configuration**: Use PHP 8 attributes for metadata

```php
// Modern CakePHP 5 approach
#[FieldType(
    name: 'text',
    label: 'Text Field',
    description: 'Short text input field',
    maxInstances: 1
)]
class TextField implements FieldInterface
{
    private FieldConfig $config;
    private ValidatorInterface $validator;
    
    public function __construct(FieldConfig $config, ValidatorInterface $validator)
    {
        $this->config = $config;
        $this->validator = $validator;
    }
    
    public function buildFormField(FormHelper $form, array $options = []): string
    {
        return $form->control($this->config->name, [
            'type' => 'text',
            'maxlength' => $this->config->getSetting('max_length', 255),
            'required' => $this->config->required,
            ...$options
        ]);
    }
    
    public function validate(mixed $value): ValidationResult
    {
        $validator = new Validator();
        $validator->scalar($this->config->name)
                 ->maxLength($this->config->name, $this->config->getSetting('max_length', 255));
                 
        $errors = $validator->validate([$this->config->name => $value]);
        return new ValidationResult($errors);
    }
    
    public function serialize(mixed $value): string
    {
        return (string) $value;
    }
    
    public function unserialize(string $value): mixed
    {
        return $value;
    }
}

// Field registry
class FieldRegistry
{
    private array $fields = [];
    
    public function register(string $name, FieldInterface $field): void
    {
        $this->fields[$name] = $field;
    }
    
    public function create(string $type, FieldConfig $config): FieldInterface
    {
        if (!isset($this->fields[$type])) {
            throw new InvalidArgumentException("Unknown field type: {$type}");
        }
        
        return clone $this->fields[$type];
    }
}
```

## What is the migration strategy from now to a modern solution?

1. **Phase 1**: Create FieldInterface and base implementations wrapping current handlers
2. **Phase 2**: Implement field registry with dependency injection
3. **Phase 3**: Convert handlers to use modern validation and form builder
4. **Phase 4**: Add PHP 8 attribute support for field metadata
5. **Phase 5**: Replace abstract Handler system with modern field objects