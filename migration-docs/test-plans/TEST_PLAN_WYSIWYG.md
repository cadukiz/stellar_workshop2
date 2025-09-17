# Test Plan: WYSIWYG Rich Text Editor

## Feature Overview
The WYSIWYG plugin provides rich text editing functionality including visual content editing, media integration, HTML management, and editor customization for QuickAppsCMS content creation.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/wysiwyg/src/Controller/AppController.php` - Base WYSIWYG controller
- `src/vendor/quickapps-plugins/wysiwyg/src/Controller/Admin/FinderController.php` - File finder for editor

### Models
- No specific models (integrates with existing content models)

### Views & Helpers
- `src/vendor/quickapps-plugins/wysiwyg/src/View/Helper/WysiwygHelper.php` - Editor integration helper
- `src/vendor/quickapps-plugins/wysiwyg/src/Template/Admin/Finder/` - File browser interface

### Assets & Configuration
- `src/vendor/quickapps-plugins/wysiwyg/webroot/js/` - Editor JavaScript files
- `src/vendor/quickapps-plugins/wysiwyg/config/` - Editor configurations
- Editor configurations for TinyMCE, CKEditor, or other editors

### Database Tables
- No plugin-specific tables
- Integrates with content and media tables

## Routes & URLs

### Admin Routes (Backend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| File Finder | `/admin/wysiwyg/finder` | `Wysiwyg\Admin\FinderController::index()` | Media browser for editor |
| Image Browser | `/admin/wysiwyg/finder/images` | `Wysiwyg\Admin\FinderController::images()` | Image selection |
| Link Browser | `/admin/wysiwyg/finder/links` | `Wysiwyg\Admin\FinderController::links()` | Link selection |
| Upload File | `/admin/wysiwyg/finder/upload` | `Wysiwyg\Admin\FinderController::upload()` | File upload via editor |

### AJAX Endpoints
| Route | URL | Description |
|-------|-----|-------------|
| Image Upload | `/wysiwyg/upload-image` | Drag-drop image upload |
| Link Checker | `/wysiwyg/check-link` | Validate external links |
| Content Preview | `/wysiwyg/preview` | Live content preview |

## Test Scenarios

### 1. Editor Integration & Loading
```gherkin
Feature: WYSIWYG Editor Integration
  As a content editor
  I want rich text editing
  To create formatted content easily

  Scenario: Load Editor on Content Form
    Given I am creating/editing content
    When I visit content form with body field
    Then WYSIWYG editor should load:
      | Editor toolbar visible |
      | Content area active |
      | Formatting options available |
      | Media upload buttons present |

  Scenario: Editor Configuration
    Given different content types
    When I open editor for:
      | Basic article | Simple toolbar |
      | Advanced content | Full toolbar |
      | Comment field | Minimal toolbar |
    Then appropriate editor config should load
    With relevant formatting options

  Scenario: Multiple Editors on Page
    Given form has multiple text fields
    When page loads with WYSIWYG fields
    Then each editor should work independently:
      | Separate toolbars |
      | Independent content |
      | No interference |

  Scenario: Editor Language Localization
    Given site language is set to Spanish
    When editor loads
    Then interface should be in Spanish:
      | Toolbar buttons translated |
      | Menu items in Spanish |
      | Error messages localized |

  Scenario: Responsive Editor Interface
    Given I am on mobile device
    When editor loads
    Then interface should adapt:
      | Touch-friendly buttons |
      | Optimized toolbar layout |
      | Proper text input area |
```

### 2. Content Formatting & Styling
```gherkin
Feature: Content Formatting
  As a content editor
  I want formatting options
  To create visually appealing content

  Scenario: Text Formatting
    Given editor is loaded
    When I apply formatting:
      | Bold | Italic | Underline |
      | Strikethrough | Subscript | Superscript |
    Then text should be formatted correctly
    And HTML should be valid

  Scenario: Paragraph and Heading Styles
    Given text content in editor
    When I apply styles:
      | Heading 1 | Heading 2 | Heading 3 |
      | Normal paragraph | Quote block |
    Then appropriate HTML tags should be used
    And styling should display correctly

  Scenario: Lists and Indentation
    Given I want to create lists
    When I use list tools:
      | Bulleted list | Numbered list |
      | Nested lists | Custom indentation |
    Then lists should render properly
    With correct HTML structure

  Scenario: Text Alignment
    Given paragraph content
    When I apply alignment:
      | Left align | Center align |
      | Right align | Justify |
    Then alignment should apply correctly
    And be preserved in saved content

  Scenario: Font and Color Options
    Given editor with font controls
    When I change:
      | Font family | Font size |
      | Text color | Background color |
    Then changes should apply immediately
    And be saved with content
```

### 3. Media Integration
```gherkin
Feature: Media Integration
  As a content editor
  I want to embed media
  To enrich my content

  Scenario: Insert Images
    Given I want to add images
    When I click "Insert Image" button
    Then media browser should open
    And I should be able to:
      | Browse existing images |
      | Upload new images |
      | Set alt text and captions |
      | Choose image alignment |

  Scenario: Drag and Drop Upload
    Given editor is active
    When I drag image file into editor
    Then image should upload automatically
    And be inserted at cursor position
    With upload progress indicator

  Scenario: Image Properties
    Given image inserted in editor
    When I right-click on image
    Then I should be able to modify:
      | Width and height |
      | Alt text |
      | Title attribute |
      | CSS classes |
      | Link destination |

  Scenario: Video Embedding
    Given I want to embed videos
    When I use video embed feature
    Then I should be able to:
      | Paste YouTube/Vimeo URLs |
      | Upload video files |
      | Set video dimensions |
      | Configure playback options |

  Scenario: File Downloads
    Given I want to link to files
    When I insert file link
    Then link should:
      | Point to file URL |
      | Show file size |
      | Include download icon |
      | Work with various file types |
```

### 4. Link Management
```gherkin
Feature: Link Management
  As a content editor
  I want to create links
  To connect content together

  Scenario: Internal Links
    Given I want to link to content
    When I create internal link
    Then I should be able to:
      | Browse content tree |
      | Search for content |
      | Link to specific pages |
      | Set link titles |

  Scenario: External Links
    Given I want to link externally
    When I create external link
    Then I should be able to:
      | Enter URL manually |
      | Set target window |
      | Add rel attributes |
      | Validate URL format |

  Scenario: Email Links
    Given I want to create email link
    When I enter email address
    Then link should:
      | Use mailto: protocol |
      | Include subject line option |
      | Validate email format |

  Scenario: Anchor Links
    Given I want page anchors
    When I create internal anchors
    Then I should be able to:
      | Insert anchor points |
      | Link to anchors |
      | Navigate to sections |

  Scenario: Link Validation
    Given links are created
    When content is saved
    Then system should:
      | Check link validity |
      | Warn about broken links |
      | Update link references |
```

### 5. Code and Source Management
```gherkin
Feature: Source Code Management
  As an advanced editor
  I want source code access
  To fine-tune content

  Scenario: HTML Source View
    Given content in visual editor
    When I switch to source view
    Then I should see:
      | Clean HTML markup |
      | Proper indentation |
      | Syntax highlighting |
      | Edit capabilities |

  Scenario: Code Block Insertion
    Given I want to show code
    When I insert code block
    Then it should:
      | Preserve formatting |
      | Apply syntax highlighting |
      | Prevent HTML parsing |
      | Support multiple languages |

  Scenario: HTML Cleanup
    Given messy HTML content
    When I use cleanup function
    Then HTML should be:
      | Well-formatted |
      | Standards compliant |
      | Unnecessary tags removed |
      | Proper nesting maintained |

  Scenario: Custom HTML Elements
    Given I need custom elements
    When I add custom HTML
    Then editor should:
      | Preserve custom tags |
      | Not strip valid attributes |
      | Maintain element structure |
      | Allow advanced formatting |

  Scenario: Paste from Word
    Given content copied from Word
    When I paste into editor
    Then content should be:
      | Cleaned of Word markup |
      | Properly formatted |
      | Images handled correctly |
      | Styles converted appropriately |
```

### 6. Editor Customization & Configuration
```gherkin
Feature: Editor Customization
  As an administrator
  I want to customize editor
  To meet specific needs

  Scenario: Toolbar Customization
    Given editor configuration access
    When I customize toolbar
    Then I should be able to:
      | Add/remove buttons |
      | Group related functions |
      | Create custom buttons |
      | Save configurations |

  Scenario: Content Filtering
    Given security requirements
    When I configure content filters
    Then editor should:
      | Strip dangerous HTML |
      | Allow safe elements only |
      | Validate attributes |
      | Log filtering actions |

  Scenario: User Role Permissions
    Given different user roles
    When configuring editor access
    Then permissions should control:
      | Available formatting options |
      | Media upload capabilities |
      | Source code access |
      | Advanced features |

  Scenario: Plugin Management
    Given editor supports plugins
    When I manage plugins
    Then I should be able to:
      | Enable/disable features |
      | Configure plugin settings |
      | Add custom plugins |
      | Update plugin versions |

  Scenario: Style Customization
    Given editor styling needs
    When I customize appearance
    Then I should be able to:
      | Set editor themes |
      | Customize colors |
      | Define content styles |
      | Preview changes |
```

## API Testing

### WYSIWYG API Endpoints
```javascript
// GET /api/wysiwyg/config/{user_role}
// Response: Editor configuration for user role

{
  "toolbar": [
    "bold", "italic", "underline", "|",
    "link", "image", "video", "|",
    "bulletlist", "numlist", "|",
    "source"
  ],
  "plugins": ["image", "link", "media", "table"],
  "upload_url": "/admin/wysiwyg/finder/upload",
  "image_browser_url": "/admin/wysiwyg/finder/images",
  "max_file_size": "5MB",
  "allowed_types": ["jpg", "png", "gif"]
}

// POST /api/wysiwyg/upload
// Body: multipart/form-data with image file
// Response: 201 Created with image details

{
  "url": "/files/images/uploaded_image.jpg",
  "alt": "Uploaded image",
  "width": 800,
  "height": 600
}

// POST /api/wysiwyg/validate-link
{
  "url": "https://example.com/page"
}
// Response: 200 OK with validation result

{
  "valid": true,
  "title": "Example Page Title",
  "status_code": 200
}

// POST /api/wysiwyg/cleanup-html
{
  "html": "<p><strong>Content</strong> with <span style='color:red'>formatting</span></p>"
}
// Response: Cleaned HTML

{
  "cleaned_html": "<p><strong>Content</strong> with formatting</p>"
}
```

## Performance Requirements

### Loading Performance
- Editor initialization: < 1 second
- Image upload: < 3 seconds for 5MB file
- Content save: < 500ms
- Link validation: < 2 seconds
- HTML cleanup: < 200ms

### Editor Responsiveness
- Typing lag: < 50ms
- Toolbar actions: < 100ms
- Content formatting: < 200ms
- Undo/redo operations: < 100ms

### Memory Usage
- Editor instance: < 20MB
- Multiple editors: < 50MB total
- Media cache: < 100MB
- Plugin overhead: < 10MB

## Edge Cases & Validation

### Content Issues
- [ ] Extremely large content (>1MB text)
- [ ] Mixed content (HTTP/HTTPS)
- [ ] Special characters and Unicode
- [ ] Malformed HTML input
- [ ] Empty content handling
- [ ] Content with embedded scripts

### Media Handling
- [ ] Large image files (>10MB)
- [ ] Unsupported file formats
- [ ] Corrupted media files
- [ ] Network timeout during upload
- [ ] Storage quota exceeded
- [ ] CDN integration issues

### Browser Compatibility
- [ ] Internet Explorer limitations
- [ ] Mobile browser differences
- [ ] Touch device interactions
- [ ] Keyboard navigation
- [ ] Screen reader compatibility

## Security Testing

### Content Security
- [ ] XSS prevention in editor content
- [ ] Script injection through paste
- [ ] HTML sanitization
- [ ] File upload validation
- [ ] Source code access control

### Upload Security
- [ ] File type validation
- [ ] File size limits
- [ ] Malicious file detection
- [ ] Path traversal prevention
- [ ] Virus scanning integration

### Editor Security
- [ ] CSRF protection on uploads
- [ ] Authentication for file browser
- [ ] Permission-based feature access
- [ ] API endpoint security

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] Helper method compatibility
- [ ] JavaScript asset loading
- [ ] File upload handling
- [ ] Form integration updates
- [ ] Plugin configuration migration

### Editor Migration
- [ ] Existing content compatibility
- [ ] Editor configuration preservation
- [ ] Custom plugin functionality
- [ ] Media references integrity

## Test Data Requirements

### Sample Content
```html
<!-- Rich text sample -->
<h2>Sample Heading</h2>
<p>This is a <strong>bold</strong> paragraph with <em>italic</em> text.</p>
<ul>
  <li>List item 1</li>
  <li>List item 2</li>
</ul>
<p><img src="/files/sample.jpg" alt="Sample image" width="300" /></p>
<p><a href="https://example.com">External link</a></p>
```

### Configuration Data
```json
{
  "editor_config": {
    "height": 400,
    "toolbar": "full",
    "plugins": ["image", "link", "table"],
    "upload_enabled": true,
    "max_file_size": "5MB"
  }
}
```

## Success Metrics

### Functional Success
- Editor loads correctly on all forms
- All formatting options working
- Media integration functional
- File upload working properly
- Link management operational

### Performance Success
- Loading times within limits
- Responsive user interaction
- Efficient content processing
- Memory usage controlled

### Security Success
- Content sanitization working
- File upload security enforced
- No XSS vulnerabilities
- Access control functional

### User Experience Success
- Intuitive interface design
- Mobile-friendly operation
- Accessible markup generated
- Reliable autosave functionality
- Cross-browser compatibility