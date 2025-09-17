# QuickAppsCMS API Documentation

## Overview

QuickAppsCMS (CakePHP 3.3.16) provides a comprehensive REST-like API through its plugin architecture. This document details all available routes, authentication methods, request/response formats, and middleware configurations.

**Base URL**: `http://localhost:8080` (Development)  
**API Version**: Based on CakePHP 3.3.16  
**Authentication**: Session-based with custom User plugin  
**Response Formats**: HTML, JSON, XML, RSS

## Table of Contents

1. [Authentication & Security](#authentication--security)
2. [User Management API](#user-management-api)
3. [Content Management API](#content-management-api)
4. [File & Media API](#file--media-api)
5. [System Administration API](#system-administration-api)
6. [Search & RSS API](#search--rss-api)
7. [Plugin Management API](#plugin-management-api)
8. [Common Response Patterns](#common-response-patterns)
9. [Error Handling](#error-handling)

---

## Authentication & Security

### Authentication System

QuickAppsCMS uses a custom authentication system built on CakePHP's Auth component with the following configuration:

```php
'Auth' => [
    'className' => 'User.Auth',
    'authenticate' => [
        'User.Form',      // Username/password authentication
        'User.Anonymous', // Anonymous user handling
    ],
    'authorize' => ['User.Cached'], // Cached role-based authorization
    'loginAction' => ['plugin' => 'User', 'controller' => 'gateway', 'action' => 'login'],
    'unauthorizedRedirect' => ['plugin' => 'User', 'controller' => 'gateway', 'action' => 'unauthorized'],
]
```

### Security Middleware

- **CSRF Protection**: Built into forms automatically
- **Maintenance Mode**: Site-wide maintenance with IP whitelisting
- **Role-based Access Control**: Cached permission system
- **Input Validation**: Entity-level validation rules
- **File Upload Security**: Extension and MIME type validation

### User Roles

1. **Anonymous Users**: Public content access only
2. **Authenticated Users**: Profile management, restricted content
3. **Admin Users**: Full system access via `/admin` prefix
4. **Custom Roles**: Plugin-defined roles with specific permissions

---

## User Management API

### Authentication Endpoints

#### Login
```
POST /login
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
username=user@example.com
password=userpassword
```

**Response (Success):**
```
HTTP/1.1 302 Found
Location: /
Set-Cookie: CAKEPHP=session_token; Path=/
```

**Response (Failure):**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Login form with error message -->
```

#### Logout
```
GET /logout
```

**Response:**
```
HTTP/1.1 302 Found
Location: /
```

### User Registration

#### Register New User
```
POST /register
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
name=John Doe
username=johndoe
email=john@example.com
password=securepassword
password_confirm=securepassword
web=http://johndoe.com
locale=en_US
public_profile=1
```

**Response (Success):**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Registration success page -->
```

#### Account Activation
```
GET /activate/{token}
```

**Parameters:**
- `token` (string): Activation token from email

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Activation confirmation page -->
```

### Password Recovery

#### Request Password Reset
```
POST /forgot
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
username=user@example.com
```

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Password recovery confirmation -->
```

### User Profile Management

#### Get Current User Profile
```
GET /user/me
Authorization: Session-based
```

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- User profile form -->
```

#### Update Current User Profile
```
POST /user/me
Authorization: Session-based
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
name=Updated Name
email=updated@example.com
web=http://newsite.com
locale=es_ES
public_profile=0
```

#### View User Profile
```
GET /user/profile/{id}
```

**Parameters:**
- `id` (integer, optional): User ID (defaults to current user)

**Authorization:** Public profiles or own profile only

---

## Content Management API

### Public Content Endpoints

#### Homepage
```
GET /
```

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Homepage with promoted content -->
```

**Content Structure:**
- Promoted content items
- View mode: `teaser`
- Ordered by sticky status, then creation date

#### Content Details
```
GET /{content_type_slug}/{content_slug}.html
```

**Parameters:**
- `content_type_slug` (string): Content type identifier (e.g., 'article', 'basic-page')
- `content_slug` (string): Content slug identifier

**Example:**
```
GET /article/introducing-quickapps-cms.html
```

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Full content page with comments -->
```

**Access Control:**
- Public content: No authentication required
- Restricted content: Role-based access control
- Draft content: Admin users only

### Admin Content Management

#### List All Content
```
GET /admin/content/manage
Authorization: Admin required
```

**Query Parameters:**
- `filter` (string, optional): Search term for content title/body
- `page` (integer, optional): Page number for pagination
- `sort` (string, optional): Sort field
- `direction` (string, optional): Sort direction ('asc' or 'desc')

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Paginated content list with management options -->
```

#### Create New Content
```
GET /admin/content/manage/create
Authorization: Admin required
```

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Content type selection page -->
```

#### Add Content Form
```
GET /admin/content/manage/add/{typeSlug}
Authorization: Admin required
```

**Parameters:**
- `typeSlug` (string): Content type slug

#### Add Content (Submit)
```
POST /admin/content/manage/add/{typeSlug}
Authorization: Admin required
Content-Type: application/x-www-form-urlencoded
```

**Request Body:**
```
title=Article Title
body=Article content body
description=Article description
content_type_slug=article
promote=1
sticky=0
comment_status=1
status=1
language=en_US
```

#### Edit Content
```
GET /admin/content/manage/edit/{id}
POST /admin/content/manage/edit/{id}
Authorization: Admin required
```

**Parameters:**
- `id` (integer): Content ID

#### Delete Content
```
POST /admin/content/manage/delete/{id}
Authorization: Admin required
```

**Parameters:**
- `id` (integer): Content ID

**Response:**
```
HTTP/1.1 302 Found
Location: /admin/content/manage
```

---

## File & Media API

### File Upload Endpoints

#### Upload File
```
POST /field/file-handler/upload/{name}
Authorization: Field permission based
Content-Type: multipart/form-data
```

**Parameters:**
- `name` (string): Field instance name

**Request Body:**
```
Filedata: [file upload]
```

**Response (Success):**
```json
{
    "file_name": "document.pdf",
    "file_size": 2048576,
    "mime_icon": "/files/file_icons/pdf.png",
    "settings": {
        "upload_folder": "/files/uploads/",
        "extensions": "pdf,doc,docx,txt",
        "file_size": "5MB"
    }
}
```

#### Delete File
```
POST /field/file-handler/delete/{name}
Authorization: Field permission based
```

**Parameters:**
- `name` (string): Field instance name

**Query Parameters:**
- `file` (string): Filename to delete

**Response:**
```json
{
    "status": "success",
    "message": "File deleted successfully"
}
```

### Image Upload Endpoints

#### Upload Image
```
POST /field/image-handler/upload/{name}
Authorization: Field permission based
Content-Type: multipart/form-data
```

**Parameters:**
- `name` (string): Field instance name

**Request Body:**
```
Filedata: [image upload]
```

**Response (Success):**
```json
{
    "file_name": "photo.jpg",
    "file_size": 1024000,
    "dimensions": {
        "width": 1920,
        "height": 1080
    },
    "thumbnails": {
        "small": "/files/uploads/photo_thumb_small.jpg",
        "medium": "/files/uploads/photo_thumb_medium.jpg"
    }
}
```

#### Get Image Thumbnail
```
GET /field/image-handler/thumbnail/{name}
Authorization: Field permission based
```

**Parameters:**
- `name` (string): Field instance name

**Query Parameters:**
- `file` (string): Image filename
- `size` (string): Thumbnail size ('small', 'medium', 'large')

**Response:**
```
HTTP/1.1 200 OK
Content-Type: image/jpeg

[Binary image data]
```

### Media Manager (Admin)

#### File Manager Interface
```
GET /admin/media-manager/explorer
Authorization: Admin required
```

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- elFinder-based file manager interface -->
```

#### File Manager Connector
```
POST /admin/media-manager/explorer/connector
Authorization: Admin required
```

**Request/Response:** elFinder connector protocol

---

## System Administration API

### Dashboard

#### Admin Dashboard
```
GET /admin
Authorization: Admin required
```

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Admin dashboard with system overview -->
```

### Plugin Management

#### List Plugins
```
GET /admin/system/plugins
Authorization: Admin required
```

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Plugin list with status and actions -->
```

#### Install Plugin
```
POST /admin/system/plugins/install
Authorization: Admin required
Content-Type: multipart/form-data
```

**Request Body (File Upload):**
```
file: [plugin zip file]
activate: 1
```

**Request Body (URL Download):**
```
download: 1
url: https://example.com/plugin.zip
activate: 1
```

**Request Body (File System):**
```
file_system: 1
path: /path/to/plugin/
activate: 1
```

#### Enable Plugin
```
POST /admin/system/plugins/enable/{pluginName}
Authorization: Admin required
```

**Parameters:**
- `pluginName` (string): Plugin name

#### Disable Plugin
```
POST /admin/system/plugins/disable/{pluginName}
Authorization: Admin required
```

#### Delete Plugin
```
POST /admin/system/plugins/delete/{pluginName}
Authorization: Admin required
```

#### Plugin Settings
```
GET /admin/system/plugins/settings/{pluginName}
POST /admin/system/plugins/settings/{pluginName}
Authorization: Admin required
```

---

## Search & RSS API

### Content Search

#### Search Content
```
GET /find/{criteria}
```

**Parameters:**
- `criteria` (string): Search criteria with operators

**Search Operators:**
- `"exact phrase"`: Exact phrase matching
- `-excluded`: Exclude terms
- `OR`: Logical OR operator
- `language:en`: Language-specific search

**Examples:**
```
GET /find/quickapps cms
GET /find/"content management" -wordpress
GET /find/cms OR framework
GET /find/language:es tutorial
```

**Query Parameters:**
- `page` (integer): Page number for pagination
- `limit` (integer): Results per page

**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html

<!-- Search results page with pagination -->
```

### RSS Feeds

#### RSS Content Feed
```
GET /rss/{criteria}
```

**Parameters:**
- `criteria` (string): Search criteria (same as content search)

**Response:**
```
HTTP/1.1 200 OK
Content-Type: application/rss+xml

<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
    <channel>
        <title>QuickAppsCMS</title>
        <description>Search results for: {criteria}</description>
        <item>
            <title>Content Title</title>
            <description>Content teaser...</description>
            <link>http://localhost:8080/article/content-slug.html</link>
            <pubDate>Wed, 01 Jan 2020 00:00:00 GMT</pubDate>
        </item>
    </channel>
</rss>
```

---

## Plugin Management API

### Generic Plugin Routes

All active plugins automatically receive the following route patterns:

#### Public Plugin Routes
```
GET /{plugin}
GET /{plugin}/{controller}
GET /{plugin}/{controller}/{action}
POST /{plugin}/{controller}/{action}
```

#### Admin Plugin Routes
```
GET /admin/{plugin}
GET /admin/{plugin}/{controller}
GET /admin/{plugin}/{controller}/{action}
POST /admin/{plugin}/{controller}/{action}
```

### Plugin-Specific APIs

#### Block Plugin
- `GET /admin/block/manage` - Block management
- `POST /admin/block/manage/add` - Create block
- `POST /admin/block/manage/edit/{id}` - Edit block

#### Menu Plugin
- `GET /admin/menu/manage` - Menu management
- `POST /admin/menu/links/add` - Add menu link
- `POST /admin/menu/links/edit/{id}` - Edit menu link

#### Taxonomy Plugin
- `GET /admin/taxonomy/vocabularies` - Vocabulary management
- `GET /admin/taxonomy/terms` - Term management
- `POST /admin/taxonomy/tagger` - Tag content

#### User Plugin (Admin)
- `GET /admin/user/manage` - User management
- `GET /admin/user/roles` - Role management
- `GET /admin/user/permissions` - Permission management

---

## Common Response Patterns

### Success Responses

#### HTML Responses
```
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Language: en
X-Generator: QuickAppsCMS 2.0.0 (http://quickappscms.org)

<!DOCTYPE html>
<html>
<!-- Themed HTML content -->
</html>
```

#### JSON Responses
```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status": "success",
    "data": { ... },
    "message": "Operation completed successfully"
}
```

#### Redirects
```
HTTP/1.1 302 Found
Location: /destination/url
Set-Cookie: CakePHP=session_data; Path=/
```

### Flash Messages

QuickAppsCMS uses a flash message system for user feedback:

```php
// Success messages
$this->Flash->success('Operation successful');

// Error messages  
$this->Flash->danger('Operation failed');

// Warning messages
$this->Flash->warning('Please check your input');
```

### Pagination

Paginated responses include pagination metadata:

```php
'paginate' => [
    'limit' => 10,
    'page' => 1,
    'count' => 150,
    'perPage' => 10,
    'prevPage' => false,
    'nextPage' => true,
    'pageCount' => 15
]
```

---

## Error Handling

### HTTP Status Codes

- `200 OK` - Successful request
- `302 Found` - Redirect response
- `400 Bad Request` - Invalid request data
- `401 Unauthorized` - Authentication required
- `403 Forbidden` - Access denied
- `404 Not Found` - Resource not found
- `500 Internal Server Error` - Server error

### Error Response Format

#### HTML Error Pages
```
HTTP/1.1 404 Not Found
Content-Type: text/html

<!-- Themed error page -->
```

#### JSON Error Responses
```json
{
    "status": "error",
    "message": "Resource not found",
    "code": 404,
    "errors": {
        "field_name": ["Validation error message"]
    }
}
```

### Custom Exceptions

- `ContentNotFoundException` - Content not found
- `SiteUnderMaintenanceException` - Site maintenance mode
- `ForbiddenException` - Access denied

### Validation Errors

Entity validation errors are returned in the response:

```php
$entity->errors(); // Returns array of validation errors
```

---

## Request/Response Examples

### Complete User Registration Flow

#### 1. Get Registration Form
```
GET /register
```

#### 2. Submit Registration
```
POST /register
Content-Type: application/x-www-form-urlencoded

name=John Doe&username=johndoe&email=john@example.com&password=secret&password_confirm=secret
```

#### 3. Activation Email Sent
```
HTTP/1.1 200 OK

<!-- Success page with activation instructions -->
```

#### 4. Activate Account
```
GET /activate/abc123def456
```

#### 5. Account Activated
```
HTTP/1.1 200 OK

<!-- Activation success page -->
```

### Content Creation Flow

#### 1. Access Admin Panel
```
GET /admin
Authorization: Required
```

#### 2. Navigate to Content Management
```
GET /admin/content/manage
```

#### 3. Create New Content
```
GET /admin/content/manage/create
```

#### 4. Select Content Type
```
GET /admin/content/manage/add/article
```

#### 5. Submit Content
```
POST /admin/content/manage/add/article
Content-Type: application/x-www-form-urlencoded

title=My Article&body=Article content&promote=1&status=1
```

### File Upload Flow

#### 1. Upload File
```
POST /field/file-handler/upload/field_instance_name
Content-Type: multipart/form-data

Filedata: [file data]
```

#### 2. File Upload Response
```json
{
    "file_name": "document.pdf",
    "file_size": 1024000,
    "mime_icon": "/files/file_icons/pdf.png"
}
```

---

## API Versioning & Migration Notes

### Current Version
- **CakePHP**: 3.3.16
- **PHP**: 7.4
- **QuickAppsCMS**: 2.0.0

### Migration Considerations

When migrating to CakePHP 5:

1. **Authentication**: Custom Auth component needs updating
2. **Routing**: Route syntax changes required
3. **Controllers**: Method signatures may change
4. **Components**: Some components deprecated/changed
5. **Request/Response**: New request/response handling

### Deprecation Warnings

- `AuthComponent` configuration syntax
- Some helper methods
- Controller property declarations
- Event system changes

---

## Security Best Practices

### Authentication
- Use strong passwords
- Enable account activation
- Implement session timeout
- Monitor failed login attempts

### Authorization
- Implement role-based access control
- Validate permissions on each request
- Use least privilege principle
- Audit permission changes

### Data Validation
- Validate all input data
- Sanitize output data
- Use parameterized queries
- Implement CSRF protection

### File Uploads
- Validate file extensions
- Check MIME types
- Limit file sizes
- Scan for malware
- Store uploads outside web root

---

This documentation represents the current state of the QuickAppsCMS API as implemented in CakePHP 3.3.16. When migrating to CakePHP 5, significant changes to authentication, routing, and request handling will be required.