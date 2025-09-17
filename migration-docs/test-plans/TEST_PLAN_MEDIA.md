# Test Plan: Media Manager

## Feature Overview
The Media Manager plugin provides file upload, media library management, image manipulation, and file serving capabilities.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/media-manager/src/Controller/Admin/ExplorerController.php` - Media browser/explorer

### Models
- `src/vendor/quickapps-plugins/media-manager/src/Model/Table/MediaTable.php`
- `src/vendor/quickapps-plugins/media-manager/src/Model/Entity/Media.php`

### Views
- `src/vendor/quickapps-plugins/media-manager/src/Template/Admin/Explorer/` - Media explorer interface
- `src/vendor/quickapps-plugins/media-manager/src/Template/Element/` - Media picker elements

### Database Tables
- `media` - Media file records
- `media_folders` - Folder organization
- `media_usage` - Track media usage in content

### File Storage
- `webroot/files/` - Public files
- `webroot/files/private/` - Private files
- `webroot/files/thumbnails/` - Generated thumbnails

## Routes & URLs

### Admin Routes (Backend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Media Explorer | `/admin/media-manager/explorer` | `MediaManager\Admin\ExplorerController::index()` | Media browser |
| Upload | `/admin/media-manager/explorer/upload` | `MediaManager\Admin\ExplorerController::upload()` | File upload |
| Create Folder | `/admin/media-manager/explorer/folder` | `MediaManager\Admin\ExplorerController::folder()` | New folder |
| Delete File | `/admin/media-manager/explorer/delete/{id}` | `MediaManager\Admin\ExplorerController::delete()` | Delete media |
| File Info | `/admin/media-manager/explorer/info/{id}` | `MediaManager\Admin\ExplorerController::info()` | File metadata |
| Edit File | `/admin/media-manager/explorer/edit/{id}` | `MediaManager\Admin\ExplorerController::edit()` | Edit properties |

### Public Routes
| Route | URL | Description |
|-------|-----|-------------|
| File Access | `/files/{path}` | Direct file access |
| Thumbnail | `/files/thumbnails/{style}/{file}` | Image thumbnails |
| Download | `/download/{id}` | Force download |

## Test Scenarios

### 1. File Upload & Management
```gherkin
Feature: File Upload
  As a content editor
  I want to upload files
  So that I can use media in content

  Scenario: Upload Single File
    Given I am on media explorer
    When I upload "image.jpg" (2MB)
    Then file should be uploaded
    And thumbnail should be generated
    And file should appear in library
    And metadata should be extracted:
      | Filename | Size | Type | Dimensions |

  Scenario: Upload Multiple Files
    Given I select multiple files:
      | photo1.jpg | photo2.png | document.pdf |
    When I drag-drop to upload area
    Then all files should upload
    And progress bar should show
    And success/failure for each

  Scenario: Upload Validation
    Given file upload limits:
      | Max size | 10MB |
      | Allowed types | jpg, png, pdf, doc |
    When I try to upload:
      | large.jpg (15MB) | Should fail - too large |
      | script.exe | Should fail - invalid type |
      | valid.jpg (1MB) | Should succeed |
    Then appropriate messages should display

  Scenario: Folder Organization
    Given I want to organize files
    When I create folder structure:
      | /images/products |
      | /images/blog |
      | /documents/reports |
    And move files to folders
    Then structure should be maintained
    And files should be accessible
```

### 2. Media Library & Browser
```gherkin
Feature: Media Browser
  As a user
  I want to browse media
  To find and use files

  Scenario: Browse Media Library
    Given library contains 100 files
    When I open media explorer
    Then I should see:
      | Grid view with thumbnails |
      | List view with details |
      | Folder tree navigation |
      | Search and filter options |
    And pagination should work

  Scenario: Search and Filter
    Given various media files exist
    When I filter by:
      | Type | Images only |
      | Date | Last 7 days |
      | Size | < 1MB |
      | Search | "logo" |
    Then filtered results should display
    And count should update

  Scenario: File Selection
    Given I need to select media
    When using media picker:
      | Single selection mode |
      | Multiple selection mode |
      | Recently used files |
    Then selection should work
    And return file references

  Scenario: File Details View
    Given I select a file
    When I view details
    Then I should see:
      | Preview/thumbnail |
      | File information |
      | Usage statistics |
      | Edit options |
      | Download link |
```

### 3. Image Manipulation
```gherkin
Feature: Image Processing
  As a content editor
  I want to manipulate images
  For different display needs

  Scenario: Generate Image Styles
    Given image styles configured:
      | Thumbnail | 150x150 crop |
      | Medium | 300x300 scale |
      | Large | 800x600 scale |
    When image is uploaded
    Then styles should be generated
    And accessible via URLs

  Scenario: Image Editing
    Given I select an image
    When I use image editor:
      | Crop to selection |
      | Rotate 90 degrees |
      | Adjust brightness |
      | Resize to 500px |
    Then changes should be saved
    And new version created

  Scenario: Responsive Images
    Given responsive display needed
    When image is rendered
    Then srcset should include:
      | Mobile | 320px |
      | Tablet | 768px |
      | Desktop | 1200px |
    And browser should select appropriate

  Scenario: Image Optimization
    Given large image uploaded
    When processing occurs
    Then image should be:
      | Compressed without quality loss |
      | Converted to WebP if supported |
      | Lazy loading enabled |
      | CDN URL if configured |
```

### 4. File Security & Access
```gherkin
Feature: Media Security
  As an administrator
  I want to control file access
  For security reasons

  Scenario: Private Files
    Given file marked as private
    When anonymous user tries access
    Then access should be denied
    And login required

  Scenario: File Permissions
    Given file with restrictions:
      | View | Authenticated users |
      | Download | Editors only |
      | Delete | Owner only |
    When different users access
    Then permissions should be enforced

  Scenario: Secure Upload
    Given security settings enabled
    When files are uploaded
    Then should be validated for:
      | Virus scanning |
      | File type verification |
      | Executable prevention |
      | Path traversal protection |

  Scenario: Usage Tracking
    Given file is used in content
    When I view file details
    Then I should see:
      | Where file is used |
      | Number of references |
      | Prevent deletion if in use |
```

## API Testing

### Media API Endpoints
```javascript
// GET /api/media
// Query params: folder, type, search, limit, offset
// Response: Paginated media list

// POST /api/media/upload
// Body: multipart/form-data with file
{
  "file": [binary],
  "folder": "/images/blog",
  "alt_text": "Description"
}
// Response: 201 Created with file details

// GET /api/media/{id}
// Response: File metadata and URLs

// PUT /api/media/{id}
{
  "alt_text": "Updated description",
  "title": "New Title",
  "folder": "/images/products"
}
// Response: 200 OK

// DELETE /api/media/{id}
// Response: 204 No Content

// POST /api/media/folder
{
  "name": "New Folder",
  "parent": "/images"
}
// Response: 201 Created

// GET /api/media/{id}/usage
// Response: List of content using this file

// POST /api/media/{id}/styles
{
  "style": "thumbnail",
  "regenerate": true
}
// Response: 200 OK with new URLs
```

## Performance Requirements

### Upload Performance
- Single file < 5MB: < 2 seconds
- Multiple files (10): < 10 seconds
- Thumbnail generation: < 500ms
- Metadata extraction: < 200ms

### Browser Performance
- Media library load: < 1 second
- Thumbnail grid (50 items): < 800ms
- Search/filter: < 300ms
- Image preview: < 500ms

### Storage Optimization
- Image compression: 20-30% reduction
- Thumbnail caching: Permanent
- CDN integration: Optional
- Cleanup orphaned files: Daily cron

## Edge Cases & Validation

### Upload Issues
- [ ] Zero byte files
- [ ] Corrupted files
- [ ] Duplicate filenames
- [ ] Special characters in names
- [ ] Path traversal attempts
- [ ] Disk space exhaustion

### File Processing
- [ ] Invalid image formats
- [ ] Extremely large dimensions
- [ ] Animated GIFs
- [ ] EXIF orientation
- [ ] Color profile handling
- [ ] Transparency preservation

### Browser Issues
- [ ] Missing thumbnails
- [ ] Broken folder structure
- [ ] Orphaned database records
- [ ] Permission conflicts
- [ ] Concurrent uploads
- [ ] Network interruptions

## Security Testing

### Upload Security
- [ ] Malicious file upload
- [ ] Script injection
- [ ] Path traversal
- [ ] File type spoofing
- [ ] Size limit bypass
- [ ] Memory exhaustion

### Access Control
- [ ] Direct file access
- [ ] Private file protection
- [ ] URL manipulation
- [ ] Session validation
- [ ] CORS configuration

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] Upload behavior migration
- [ ] File validation rules
- [ ] Storage adapter changes
- [ ] Image processing library
- [ ] AJAX upload handlers
- [ ] File field migration

## Test Data Requirements

### Media Files
```bash
# Test files needed
/test-media/
  ├── images/
  │   ├── small.jpg (100KB)
  │   ├── large.jpg (5MB)
  │   ├── transparent.png
  │   └── animated.gif
  ├── documents/
  │   ├── report.pdf
  │   └── spreadsheet.xlsx
  └── invalid/
      ├── script.php
      └── corrupted.jpg
```

### Database Seed
```sql
INSERT INTO media (filename, filepath, mimetype, filesize) VALUES
('logo.png', '/images/logo.png', 'image/png', 50000),
('document.pdf', '/docs/document.pdf', 'application/pdf', 1000000),
('video.mp4', '/videos/video.mp4', 'video/mp4', 10000000);
```

## Success Metrics

- File upload functional
- Media browser working
- Image styles generating
- Folder organization working
- Search/filter operational
- Security enforced
- Performance within limits
- API endpoints working
- No data loss
- Migration successful