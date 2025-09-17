# Three-Tier Template System

## What problem was it solving?

Standard CakePHP has a single template directory, making it difficult to:
- Separate admin interface templates from public templates
- Share common components between admin and public interfaces
- Allow themes to only override public templates while preserving admin interface
- Provide different template override hierarchies

## How does it work?

**Template Structure**:
```
templates/
├── Back/     # Admin interface templates
│   ├── Plugin/
│   ├── Element/
│   └── Layout/
├── Common/   # Shared components and layouts
│   ├── Element/
│   └── Layout/
└── Front/    # Public-facing templates
    ├── Plugin/
    ├── Element/
    └── Layout/
```

**Override Hierarchy**:
- Plugin template: `Node.Element/render_node.ctp`
- Override path: `ROOT/templates/Front/Plugin/Node/Element/render_node.ctp` (for frontend)
- Override path: `ROOT/templates/Back/Plugin/Node/Element/render_node.ctp` (for admin)

**Implementation**: The system detects admin context and adjusts template paths accordingly.

## What would be the modern CakePHP 5 approach?

CakePHP 5 provides better template organization options:

1. **Template Path Configuration**: Configure multiple template paths
2. **Context-Aware View**: Use different view classes for admin/frontend
3. **Theme Plugins**: Implement themes as plugins with proper inheritance
4. **View Blocks**: Use view blocks for shared components

```php
// Modern CakePHP 5 approach
class ContextualView extends View
{
    protected array $templatePaths = [];
    
    public function initialize(): void
    {
        parent::initialize();
        
        // Set template paths based on context
        if ($this->getRequest()->getParam('prefix') === 'Admin') {
            $this->setTemplatePath([
                ROOT . DS . 'templates' . DS . 'Admin' . DS,
                ROOT . DS . 'templates' . DS . 'Common' . DS,
            ]);
        } else {
            $this->setTemplatePath([
                ROOT . DS . 'templates' . DS . 'Frontend' . DS,
                ROOT . DS . 'templates' . DS . 'Common' . DS,
            ]);
        }
    }
    
    // Override template location logic
    protected function _getTemplateFileName(?string $name = null): string
    {
        foreach ($this->templatePaths as $path) {
            $file = $path . $this->templatePath . $name . $this->_ext;
            if (is_file($file)) {
                return $file;
            }
        }
        
        return parent::_getTemplateFileName($name);
    }
}
```

## What is the migration strategy from now to a modern solution?

1. **Phase 1**: Implement custom View class with multiple template path support
2. **Phase 2**: Reorganize templates into clearer Admin/Frontend/Shared structure
3. **Phase 3**: Create theme system using CakePHP 5 plugin architecture
4. **Phase 4**: Add template fallback and override validation
5. **Phase 5**: Integrate with CakePHP 5's improved view layer features