# Test Plan: Comment System

## Feature Overview
The Comment plugin provides comprehensive commenting functionality including threaded discussions, comment moderation, spam protection, and user interaction features for QuickAppsCMS content.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/comment/src/Controller/AppController.php` - Base comment controller
- `src/vendor/quickapps-plugins/comment/src/Controller/CommentController.php` - Public comment actions
- `src/vendor/quickapps-plugins/comment/src/Controller/Admin/CommentsController.php` - Comment moderation

### Models
- `src/vendor/quickapps-plugins/comment/src/Model/Table/CommentsTable.php` - Comment management
- `src/vendor/quickapps-plugins/comment/src/Model/Entity/Comment.php` - Comment entity

### Views & Helpers
- `src/vendor/quickapps-plugins/comment/src/View/Helper/CommentHelper.php` - Comment rendering
- `src/vendor/quickapps-plugins/comment/src/Template/Comment/` - Comment templates
- `src/vendor/quickapps-plugins/comment/src/Template/Admin/Comments/` - Admin interface

### Components
- `src/vendor/quickapps-plugins/comment/src/Controller/Component/CommentComponent.php` - Comment functionality

### Database Tables
- `comments` - Comment storage
- `comment_types` - Comment type definitions
- `comments_meta` - Additional comment metadata

## Routes & URLs

### Public Routes (Frontend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Post Comment | `/comment/add/{content_id}` | `Comment\Controller\CommentController::add()` | Submit new comment |
| Reply to Comment | `/comment/reply/{comment_id}` | `Comment\Controller\CommentController::reply()` | Reply to comment |
| Edit Comment | `/comment/edit/{id}` | `Comment\Controller\CommentController::edit()` | Edit own comment |
| Delete Comment | `/comment/delete/{id}` | `Comment\Controller\CommentController::delete()` | Delete own comment |
| Report Comment | `/comment/report/{id}` | `Comment\Controller\CommentController::report()` | Report inappropriate comment |

### Admin Routes (Backend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Comment List | `/admin/comment/comments` | `Comment\Admin\CommentsController::index()` | List all comments |
| Pending Comments | `/admin/comment/comments/pending` | `Comment\Admin\CommentsController::pending()` | Moderation queue |
| Approve Comment | `/admin/comment/comments/approve/{id}` | `Comment\Admin\CommentsController::approve()` | Approve comment |
| Reject Comment | `/admin/comment/comments/reject/{id}` | `Comment\Admin\CommentsController::reject()` | Reject comment |
| Bulk Actions | `/admin/comment/comments/bulk` | `Comment\Admin\CommentsController::bulk()` | Bulk operations |
| Comment Settings | `/admin/comment/settings` | `Comment\Admin\SettingsController::index()` | Configure commenting |

## Test Scenarios

### 1. Comment Creation & Posting
```gherkin
Feature: Comment Posting
  As a site visitor
  I want to post comments
  To engage with content

  Scenario: Post Comment as Registered User
    Given I am logged in
    And I am viewing content with comments enabled
    When I fill in comment form:
      | Field | Value |
      | Comment | This is my comment about the article |
      | Name | Auto-filled from profile |
      | Email | Auto-filled from profile |
    And I click "Post Comment"
    Then comment should be submitted
    And appear in comment list (if auto-approved)
    Or appear in moderation queue (if requires approval)

  Scenario: Post Comment as Anonymous User
    Given I am not logged in
    And anonymous comments are allowed
    When I fill in comment form:
      | Field | Value |
      | Comment | Anonymous comment text |
      | Name | John Visitor |
      | Email | john@example.com |
      | Website | http://example.com |
    And I solve CAPTCHA (if enabled)
    And I click "Post Comment"
    Then comment should be submitted
    And email notification sent to moderators

  Scenario: Reply to Existing Comment
    Given existing comment thread
    When I click "Reply" on a comment
    And I fill in reply form:
      | Reply text | "I agree with your point" |
    And submit reply
    Then reply should be nested under parent
    And threading should be maintained
    And parent commenter notified (if enabled)

  Scenario: Comment Validation
    Given I am posting a comment
    When I submit invalid data:
      | Empty comment body |
      | Invalid email format |
      | Comment too long (>5000 chars) |
      | Spam-like content |
    Then validation errors should display
    And comment should not be saved

  Scenario: Comment with Mentions
    Given I am posting a comment
    When I mention other users:
      | "@johndoe thanks for sharing" |
    Then mentioned users should be notified
    And mentions should be highlighted
```

### 2. Comment Moderation & Administration
```gherkin
Feature: Comment Moderation
  As a content moderator
  I want to manage comments
  To maintain discussion quality

  Scenario: View Moderation Queue
    Given I am logged in as moderator
    When I visit "/admin/comment/comments/pending"
    Then I should see pending comments:
      | Comment text preview |
      | Author information |
      | Associated content |
      | Submission date |
      | Moderation actions |

  Scenario: Approve Comments
    Given comments awaiting approval
    When I review comment content
    And I click "Approve"
    Then comment should be published
    And author should be notified (if enabled)
    And comment appears on frontend

  Scenario: Reject Comments
    Given inappropriate comment in queue
    When I click "Reject"
    And optionally provide reason
    Then comment should be marked as rejected
    And author notified of rejection
    And comment hidden from public

  Scenario: Edit Comments
    Given published comment with typos
    When I edit the comment content
    And save changes
    Then comment should be updated
    And edit history maintained
    And "edited by moderator" flag shown

  Scenario: Bulk Comment Operations
    Given multiple comments selected
    When I perform bulk action:
      | Approve all | Reject all |
      | Mark as spam | Delete all |
    Then action should apply to all selected
    And appropriate notifications sent

  Scenario: Spam Detection
    Given comment contains spam indicators:
      | Multiple links |
      | Keyword stuffing |
      | Suspicious patterns |
    Then comment should be flagged
    And sent to spam folder
    And auto-blocked if confidence high
```

### 3. Threaded Discussion Management
```gherkin
Feature: Comment Threading
  As a user
  I want threaded discussions
  To follow conversation flow

  Scenario: Multi-level Threading
    Given comments support threading
    When users create comment hierarchy:
      | Level 1: Original comment |
      | Level 2: Reply to original |
      | Level 3: Reply to reply |
      | Level 4: Deep nested reply |
    Then threading should display correctly
    With proper indentation and nesting

  Scenario: Thread Collapse/Expand
    Given long comment thread
    When I click "Collapse thread"
    Then child comments should hide
    And "Expand" option should appear
    When I click "Expand"
    Then full thread should show again

  Scenario: Thread Navigation
    Given deep comment thread
    When I click "Jump to parent"
    Then view should scroll to parent comment
    And parent should be highlighted

  Scenario: Thread Ordering
    Given multiple comments at same level
    When I configure sorting:
      | Newest first | Oldest first |
      | Best rated | Most replies |
    Then comments should reorder accordingly
    And threading structure maintained

  Scenario: Maximum Thread Depth
    Given thread depth limit is 5
    When user tries to reply at level 6
    Then reply option should be disabled
    Or redirected to level 5 reply
```

### 4. Comment Settings & Configuration
```gherkin
Feature: Comment Configuration
  As an administrator
  I want to configure comment system
  To control commenting behavior

  Scenario: Enable/Disable Comments
    Given content type configuration
    When I enable/disable comments for:
      | Articles | Enabled |
      | Pages | Disabled |
      | Products | Enabled with approval |
    Then comment forms should appear/disappear
    According to configuration

  Scenario: Comment Approval Settings
    Given comment approval options
    When I configure:
      | Auto-approve all | Require approval |
      | Approve registered users only |
      | Require approval for links |
    Then comments should follow approval rules

  Scenario: Anonymous Comment Settings
    Given anonymous commenting options
    When I configure:
      | Allow anonymous | Require registration |
      | Require email verification |
      | CAPTCHA for anonymous |
    Then anonymous posting should behave accordingly

  Scenario: Comment Closing Rules
    Given comment closing configuration
    When I set rules:
      | Close after 30 days |
      | Close for archived content |
      | Manual close only |
    Then commenting should be disabled per rules

  Scenario: Notification Settings
    Given notification preferences
    When I configure notifications for:
      | New comments | Comment replies |
      | Moderation needs | Spam detection |
    Then appropriate notifications should be sent
```

### 5. Comment Display & Rendering
```gherkin
Feature: Comment Display
  As a site visitor
  I want to read comments easily
  With good user experience

  Scenario: Comment List Display
    Given content with comments
    When I view the content
    Then comments should display:
      | Author name and avatar |
      | Comment timestamp |
      | Comment content formatted |
      | Reply/Report links |
      | Like/Dislike buttons (if enabled) |

  Scenario: Pagination of Comments
    Given content with 100+ comments
    When viewing comment section
    Then comments should be paginated:
      | Show 20 comments per page |
      | Load more button option |
      | AJAX pagination |
      | Maintain thread structure |

  Scenario: Comment Formatting
    Given comment with various content
    When comment is displayed
    Then formatting should be applied:
      | Line breaks preserved |
      | URLs auto-linked |
      | Mentions highlighted |
      | Basic HTML allowed (if enabled) |
      | Code blocks formatted |

  Scenario: Mobile Comment Display
    Given I am on mobile device
    When viewing comments
    Then display should be optimized:
      | Touch-friendly buttons |
      | Readable text size |
      | Efficient use of space |
      | Thumb-friendly interaction |

  Scenario: Comment Highlighting
    Given I am viewing comments
    When I click direct comment link
    Then target comment should be:
      | Scrolled into view |
      | Highlighted temporarily |
      | Thread expanded if collapsed |
```

## API Testing

### Comment API Endpoints
```javascript
// GET /api/comments/{content_id}
// Response: Comments for specific content

{
  "content_id": 123,
  "total": 25,
  "comments": [
    {
      "id": 1,
      "content": "Great article!",
      "author": "John Doe",
      "created": "2024-01-15T10:30:00Z",
      "parent_id": null,
      "status": "approved",
      "replies": [
        {
          "id": 2,
          "content": "I agree!",
          "author": "Jane Smith",
          "parent_id": 1
        }
      ]
    }
  ]
}

// POST /api/comments
{
  "content_id": 123,
  "content": "This is my comment",
  "author_name": "John Doe",
  "author_email": "john@example.com",
  "parent_id": null
}
// Response: 201 Created with comment object

// PUT /api/comments/{id}
{
  "content": "Updated comment text",
  "status": "approved"
}
// Response: 200 OK

// DELETE /api/comments/{id}
// Response: 204 No Content

// POST /api/comments/{id}/report
{
  "reason": "spam",
  "description": "This looks like spam content"
}
// Response: 200 OK

// GET /api/admin/comments/pending
// Response: Comments awaiting moderation

// PUT /api/admin/comments/bulk
{
  "action": "approve",
  "comment_ids": [1, 2, 3, 4, 5]
}
// Response: 200 OK with operation summary
```

## Performance Requirements

### Response Times
- Comment list load: < 500ms (50 comments)
- Comment submission: < 300ms
- Moderation actions: < 200ms
- Threaded display: < 800ms (nested 5 levels)
- Search comments: < 1 second

### Database Performance
```sql
-- Critical comment indexes
CREATE INDEX idx_comments_content ON comments(content_id, status);
CREATE INDEX idx_comments_parent ON comments(parent_id);
CREATE INDEX idx_comments_author ON comments(author_id);
CREATE INDEX idx_comments_created ON comments(created);
CREATE INDEX idx_comments_status ON comments(status);
```

### Caching Strategy
- Comment trees: 15 minute cache
- Comment counts: 30 minute cache
- Moderation queue: No cache
- Comment forms: 1 hour cache

## Edge Cases & Validation

### Comment Content
- [ ] Empty comments
- [ ] Extremely long comments (>10,000 chars)
- [ ] HTML injection attempts
- [ ] Script injection
- [ ] Unicode and emoji handling
- [ ] Special characters in names

### Threading Issues
- [ ] Circular parent references
- [ ] Orphaned comments
- [ ] Maximum depth exceeded
- [ ] Deleted parent comments
- [ ] Thread corruption

### Moderation Edge Cases
- [ ] Comment approved/rejected simultaneously
- [ ] Deleted content with comments
- [ ] User deleted with comments
- [ ] Bulk operations timing out
- [ ] Spam filter false positives

## Security Testing

### Input Security
- [ ] XSS prevention in comment content
- [ ] SQL injection via comment fields
- [ ] CSRF protection on comment forms
- [ ] Rate limiting for comment posting
- [ ] Email validation for anonymous comments

### Access Control
- [ ] Edit own comments only
- [ ] Moderation permission checks
- [ ] Admin-only bulk operations
- [ ] Content access verification
- [ ] API authentication

### Spam Protection
- [ ] CAPTCHA integration
- [ ] Rate limiting per IP/user
- [ ] Content analysis for spam
- [ ] Blacklist word filtering
- [ ] Honeypot fields

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] Comment model associations
- [ ] Helper method compatibility
- [ ] Component integration
- [ ] Form helper updates
- [ ] Validation rule migration
- [ ] Event system changes

### Data Migration
- [ ] Comment thread integrity
- [ ] Author relationships preserved
- [ ] Moderation status maintained
- [ ] Timestamps accurate
- [ ] Content associations intact

## Test Data Requirements

### Sample Comments
```sql
INSERT INTO comments (content_id, parent_id, author_name, author_email, content, status) VALUES
(1, NULL, 'John Doe', 'john@example.com', 'Great article! Very informative.', 'approved'),
(1, 1, 'Jane Smith', 'jane@example.com', 'I agree, thanks for sharing!', 'approved'),
(1, NULL, 'Bob Wilson', 'bob@example.com', 'This needs more details.', 'pending'),
(2, NULL, 'Anonymous', 'anon@example.com', 'Anonymous comment here', 'spam');
```

### Comment Thread Structure
```
Comment 1 (Root)
├── Comment 2 (Reply)
│   ├── Comment 3 (Reply to Reply)
│   └── Comment 4 (Reply to Reply)
├── Comment 5 (Reply)
└── Comment 6 (Reply)
```

## Success Metrics

### Functional Success
- Comment posting working correctly
- Threading displaying properly
- Moderation system operational
- Spam protection active
- Notifications functioning

### Performance Success
- Response times within limits
- Database queries optimized
- Page load impact minimal
- AJAX functionality smooth

### Security Success
- No XSS vulnerabilities
- Input validation working
- Access control enforced
- Spam detection effective
- Rate limiting functional

### User Experience Success
- Intuitive comment interface
- Mobile-friendly display
- Accessible markup
- Clear moderation feedback
- Reliable notification system