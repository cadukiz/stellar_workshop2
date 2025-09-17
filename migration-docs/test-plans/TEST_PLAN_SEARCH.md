# Test Plan: Search Functionality

## Feature Overview
The Search plugin provides full-text search capabilities, content indexing, search result display, and advanced search features for QuickAppsCMS.

## File Locations

### Controllers
- Search functionality integrated into Content plugin
- `/find/{criteria}` route in content routes

### Models
- Search indexing through Content models
- Search behavior and finders

### Views
- Search results templates
- Search forms and widgets

### Database Tables
- `search_index` - Full-text search index
- `search_statistics` - Search analytics
- Content tables with search-enabled fields

### Search Index
- Full-text indexing of content
- Field-specific search weights
- Language-aware indexing

## Routes & URLs

### Public Routes (Frontend)
| Route | URL Pattern | Controller Action | Description |
|-------|------------|------------------|-------------|
| Search | `/find/{criteria}` | `Content\Controller\ServeController::search()` | Search results |
| Advanced Search | `/search/advanced` | `Content\Controller\SearchController::advanced()` | Advanced search form |
| Search API | `/api/search` | `Api\SearchController::index()` | Search API endpoint |

### Admin Routes (Backend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Search Config | `/admin/search/configuration` | `Search\Admin\ConfigController::index()` | Search settings |
| Reindex | `/admin/search/reindex` | `Search\Admin\IndexController::rebuild()` | Rebuild search index |
| Statistics | `/admin/search/statistics` | `Search\Admin\StatsController::index()` | Search analytics |

## Test Scenarios

### 1. Basic Search Functionality
```gherkin
Feature: Basic Search
  As a site visitor
  I want to search for content
  To find relevant information

  Scenario: Simple Text Search
    Given site has indexed content
    When I search for "technology"
    Then I should see search results
    With highlighted search terms
    And relevance-based ordering

  Scenario: Multi-word Search
    Given I search for "artificial intelligence"
    When search is performed
    Then results should include:
      | Exact phrase matches (highest priority) |
      | All words present |
      | Some words present |
    Ordered by relevance

  Scenario: Empty Search
    Given I perform empty search
    Then I should see:
      | "Please enter search terms" message |
      | Popular content or recent articles |
      | Search suggestions |

  Scenario: No Results Found
    Given I search for "xyznonexistent"
    When no content matches
    Then I should see:
      | "No results found" message |
      | Search suggestions |
      | Alternative search options |
      | Popular content links |

  Scenario: Search Result Display
    Given search returns results
    When viewing results page
    Then each result should show:
      | Title with search term highlighting |
      | Content excerpt with context |
      | Content type and date |
      | URL/breadcrumb |
      | Relevance score (if admin) |
```

### 2. Advanced Search Features
```gherkin
Feature: Advanced Search
  As a power user
  I want advanced search options
  For precise content discovery

  Scenario: Search by Content Type
    Given I want to filter results
    When I search for "guide" in "Articles" only
    Then results should only include article content
    And show content type filter was applied

  Scenario: Date Range Search
    Given I need recent content
    When I search with date filter:
      | From | 2024-01-01 |
      | To | 2024-12-31 |
    Then results should be within date range
    And sorted by date or relevance

  Scenario: Author Search
    Given I want content by specific author
    When I search by author "John Doe"
    Then results should only show John's content
    And include author information

  Scenario: Tag-based Search
    Given content is tagged
    When I search within "technology" tag
    Then results should be pre-filtered
    And show tag-specific content only

  Scenario: Field-specific Search
    Given custom fields are searchable
    When I search in "Product SKU" field
    Then search should target that field
    And return precise matches

  Scenario: Boolean Search Operators
    Given I use search operators:
      | "artificial intelligence" | Exact phrase |
      | AI OR ML | Either term |
      | technology -social | Exclude term |
      | +required optional | Required/optional |
    Then search should interpret operators
    And return appropriate results
```

### 3. Search Indexing & Performance
```gherkin
Feature: Search Indexing
  As a system administrator
  I want efficient search indexing
  For fast and accurate search

  Scenario: Content Indexing
    Given new content is published
    When indexing process runs
    Then content should be searchable
    Within configured time (5 minutes)

  Scenario: Real-time vs Batch Indexing
    Given indexing configuration:
      | Real-time | Index immediately on save |
      | Batch | Index every 15 minutes |
    When content is updated
    Then indexing should occur per configuration

  Scenario: Full Reindex
    Given search index needs rebuilding
    When I trigger full reindex
    Then all content should be re-indexed
    And search should remain available
    With progress indicator

  Scenario: Index Optimization
    Given large amount of content
    When index optimization runs
    Then search performance should improve
    And index size should be optimized

  Scenario: Multilingual Indexing
    Given content in multiple languages
    When indexing occurs
    Then each language should be indexed separately
    With language-specific stemming
```

### 4. Search Analytics & Statistics
```gherkin
Feature: Search Analytics
  As a content manager
  I want search analytics
  To understand user behavior

  Scenario: Search Term Tracking
    Given users perform searches
    When I view search statistics
    Then I should see:
      | Most popular search terms |
      | Search frequency |
      | Zero-result searches |
      | Peak search times |

  Scenario: Result Click Tracking
    Given search results are displayed
    When users click results
    Then clicks should be tracked
    And help improve relevance scoring

  Scenario: Search Performance Metrics
    Given search system is monitored
    When I view performance stats
    Then I should see:
      | Average search time |
      | Index size and growth |
      | Search volume trends |
      | Error rates |

  Scenario: Content Gap Analysis
    Given zero-result searches tracked
    When I analyze failed searches
    Then I can identify:
      | Missing content topics |
      | Popular unmet demands |
      | Content creation opportunities |
```

### 5. Search UI & User Experience
```gherkin
Feature: Search Interface
  As a user
  I want intuitive search interface
  For easy content discovery

  Scenario: Auto-complete Suggestions
    Given I start typing search term
    When typing "tech"
    Then suggestions should appear:
      | "technology" | "technical" | "technique" |
    Based on popular searches and content

  Scenario: Search Filters UI
    Given search results are displayed
    When I apply filters:
      | Content type | Date range |
      | Author | Category |
    Then results should update dynamically
    And filters should be clearly visible

  Scenario: Faceted Search
    Given search supports facets
    When I view search results
    Then I should see filter options:
      | By content type (5) |
      | By author (3) |
      | By date (This year: 10) |
    With result counts per facet

  Scenario: Search Result Pagination
    Given many search results
    When I navigate pages
    Then pagination should work smoothly
    And preserve search context
    And show total result count

  Scenario: Search within Results
    Given I have search results
    When I want to narrow down further
    Then I should be able to:
      | Search within current results |
      | Add additional terms |
      | Refine existing query |

  Scenario: Mobile Search Experience
    Given I use mobile device
    When I search for content
    Then interface should be:
      | Touch-friendly |
      | Fast loading |
      | Easy to navigate |
      | Thumb-optimized |
```

## API Testing

### Search API Endpoints
```javascript
// GET /api/search?q={query}
// Query params: q, type, author, from_date, to_date, limit, offset
// Response: Paginated search results

{
  "query": "technology",
  "total": 150,
  "results": [
    {
      "id": 1,
      "title": "AI Technology Trends",
      "excerpt": "Latest trends in artificial intelligence...",
      "type": "article",
      "author": "John Doe",
      "date": "2024-01-15",
      "url": "/article/ai-technology-trends.html",
      "score": 0.95
    }
  ],
  "facets": {
    "content_type": {"article": 45, "page": 10},
    "author": {"John Doe": 5, "Jane Smith": 8}
  }
}

// POST /api/search/advanced
{
  "query": "artificial intelligence",
  "filters": {
    "content_type": ["article", "blog"],
    "date_from": "2024-01-01",
    "date_to": "2024-12-31",
    "author": "John Doe"
  },
  "sort": "relevance"
}
// Response: Filtered search results

// GET /api/search/suggestions?q={partial}
// Response: Auto-complete suggestions

// POST /api/search/track
{
  "query": "technology",
  "result_id": 123,
  "action": "click"
}
// Response: 200 OK (analytics tracking)

// GET /api/search/statistics
// Response: Search analytics data
```

## Performance Requirements

### Search Response Times
- Simple search: < 200ms
- Advanced search: < 500ms
- Auto-complete: < 100ms
- Search within 1000 items: < 300ms

### Indexing Performance
- Index single content: < 50ms
- Batch index 100 items: < 10 seconds
- Full reindex (10,000 items): < 30 minutes
- Index optimization: < 5 minutes

### Database Optimization
```sql
-- Search performance indexes
CREATE FULLTEXT INDEX idx_search_content ON search_index(title, content);
CREATE INDEX idx_search_type ON search_index(content_type);
CREATE INDEX idx_search_date ON search_index(created);
CREATE INDEX idx_search_author ON search_index(author_id);
CREATE INDEX idx_search_status ON search_index(status);
```

### Caching Strategy
- Search results: 5 minute cache
- Auto-complete: 1 hour cache
- Popular searches: 24 hour cache
- Search statistics: 1 hour cache

## Edge Cases & Validation

### Search Query Handling
- [ ] Very long search queries (>1000 chars)
- [ ] Special characters and symbols
- [ ] SQL injection attempts
- [ ] Unicode and international text
- [ ] Empty and whitespace-only queries
- [ ] Malformed boolean operators

### Search Results
- [ ] Large result sets (>10,000 items)
- [ ] Identical titles/content
- [ ] Deleted content in index
- [ ] Permission-restricted content
- [ ] Multilingual result mixing
- [ ] Corrupted index entries

### Performance Issues
- [ ] Concurrent search requests
- [ ] Search index lock contention
- [ ] Memory usage with large indexes
- [ ] Disk space for search index
- [ ] Network timeouts
- [ ] Database connection limits

## Security Testing

### Search Input Security
- [ ] XSS in search queries
- [ ] SQL injection prevention
- [ ] Search result manipulation
- [ ] Access control bypass
- [ ] Information disclosure

### Search Analytics Security
- [ ] Search query logging
- [ ] Personal data in search terms
- [ ] Analytics data protection
- [ ] Search history privacy

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] Search behavior migration
- [ ] Full-text index compatibility
- [ ] Query builder updates
- [ ] Search result pagination
- [ ] Highlight functionality
- [ ] Search form helpers

## Test Data Requirements

### Content for Indexing
```sql
-- Sample content with searchable text
INSERT INTO contents (title, body, content_type_id, status) VALUES
('Artificial Intelligence Guide', 'Comprehensive guide to AI technology...', 1, 'published'),
('Machine Learning Basics', 'Introduction to ML concepts and applications...', 1, 'published'),
('Technology Trends 2024', 'Latest trends in tech industry...', 1, 'published'),
('Web Development Tips', 'Best practices for modern web development...', 1, 'published');
```

### Search Index
```sql
-- Search index entries
INSERT INTO search_index (entity_id, table_alias, title, content, content_type, author_id) VALUES
(1, 'contents', 'Artificial Intelligence Guide', 'ai artificial intelligence machine learning...', 'article', 1),
(2, 'contents', 'Machine Learning Basics', 'machine learning ml algorithms data science...', 'article', 1);
```

## Success Metrics

- Search functionality working correctly
- Indexing process operational
- Search results accurate and relevant
- Performance within benchmarks
- Auto-complete functional
- Advanced search features working
- Analytics tracking properly
- API endpoints responding
- Mobile search experience optimal
- Zero data loss during migration
- Search result highlighting working
- Faceted search operational
- Multi-language search functional
- Security properly implemented