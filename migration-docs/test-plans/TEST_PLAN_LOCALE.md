# Test Plan: Localization & Internationalization

## Feature Overview
The Locale plugin provides multilingual support, translation management, language detection, and internationalization features for QuickAppsCMS.

## File Locations

### Controllers
- `src/vendor/quickapps-plugins/locale/src/Controller/Admin/ManageController.php` - Language management

### Models
- `src/vendor/quickapps-plugins/locale/src/Model/Table/LanguagesTable.php`
- `src/vendor/quickapps-plugins/locale/src/Model/Table/TranslationsTable.php`
- `src/vendor/quickapps-plugins/locale/src/Model/Entity/Language.php`
- `src/vendor/quickapps-plugins/locale/src/Model/Entity/Translation.php`

### Views
- `src/vendor/quickapps-plugins/locale/src/Template/Admin/Manage/` - Language management interface

### Database Tables
- `languages` - Available languages
- `translations` - Translation strings
- `i18n` - Database table translations

### Locale Files
- `src/Locale/{locale}/` - PO/POT translation files
- `plugins/{plugin}/src/Locale/{locale}/` - Plugin translations

## Routes & URLs

### Admin Routes (Backend)
| Route | URL | Controller Action | Description |
|-------|-----|------------------|-------------|
| Languages | `/admin/locale/manage` | `Locale\Admin\ManageController::index()` | Language management |
| Add Language | `/admin/locale/manage/add` | `Locale\Admin\ManageController::add()` | Add new language |
| Edit Language | `/admin/locale/manage/edit/{id}` | `Locale\Admin\ManageController::edit()` | Edit language |
| Import | `/admin/locale/manage/import` | `Locale\Admin\ManageController::import()` | Import translations |
| Export | `/admin/locale/manage/export` | `Locale\Admin\ManageController::export()` | Export translations |

### Public Routes (Frontend)
| Route | URL | Description |
|-------|-----|-------------|
| Localized Content | `/{locale}/path` | Language-specific URLs |
| Language Switch | `/language-switch/{locale}` | Change language |

## Test Scenarios

### 1. Language Management
```gherkin
Feature: Language Management
  As an administrator
  I want to manage languages
  To provide multilingual support

  Scenario: Add New Language
    Given I am on "/admin/locale/manage/add"
    When I fill in:
      | Field | Value |
      | Name | Español |
      | Code | es |
      | Direction | Left to right |
      | Status | Enabled |
      | Weight | 1 |
    And click "Save"
    Then language should be added
    And appear in language list
    And be available for content

  Scenario: Enable/Disable Language
    Given language "French" exists
    When I disable the language
    Then language should be hidden
    And content should not be accessible
    But translations should be preserved

  Scenario: Set Default Language
    Given multiple languages exist
    When I set "Spanish" as default
    Then site should use Spanish
    And fallback to Spanish for missing translations

  Scenario: Configure Language Detection
    Given language detection options:
      | Browser preference | URL prefix |
      | User account setting | Cookie |
    When I enable detection methods
    Then appropriate language should be selected
    Based on user preference
```

### 2. Translation Management
```gherkin
Feature: Translation Management
  As a translator
  I want to manage translations
  To localize the interface

  Scenario: Import Translation File
    Given I have PO file for Spanish
    When I import the file
    Then translations should be loaded
    And interface should be translated
    And statistics should update

  Scenario: Edit Translations
    Given English string "Welcome"
    When I add Spanish translation "Bienvenido"
    And French translation "Bienvenue"
    Then translations should be saved
    And display correctly per language

  Scenario: Export Translations
    Given translations exist for German
    When I export German translations
    Then PO file should be generated
    With all translated strings
    And proper formatting

  Scenario: Translation Statistics
    Given multiple languages configured
    When I view translation status
    Then I should see:
      | Language | Total strings | Translated | Percentage |
    And identify missing translations

  Scenario: Pluralization Rules
    Given language has plural forms
    When translating strings with counts:
      | "1 item" -> "1 elemento" |
      | "5 items" -> "5 elementos" |
    Then correct plural form should display
    Based on language rules
```

### 3. Content Translation
```gherkin
Feature: Content Translation
  As a content editor
  I want to translate content
  To reach multilingual audience

  Scenario: Translate Content Item
    Given English article exists
    When I create Spanish translation:
      | Title | Spanish title |
      | Body | Spanish content |
      | Slug | spanish-slug |
    Then translation should be linked
    And appear in Spanish site

  Scenario: Language Fallback
    Given content in English only
    When Spanish user visits site
    Then English content should display
    With language fallback notice

  Scenario: Language Switcher
    Given content has multiple translations
    When I use language switcher
    Then I should see available languages
    And switch to corresponding translation

  Scenario: Multilingual Menus
    Given menu has language-specific items
    When language changes
    Then appropriate menu should display
    With translated link titles

  Scenario: Multilingual Taxonomies
    Given vocabulary supports translation
    When I create terms in multiple languages
    Then terms should be language-specific
    And content should use appropriate terms
```

### 4. URL Structure & SEO
```gherkin
Feature: Multilingual URLs
  As a site visitor
  I want language-specific URLs
  For better user experience

  Scenario: Language Prefix URLs
    Given URL prefix enabled
    When I visit different languages:
      | English | /en/about-us |
      | Spanish | /es/acerca-de |
      | French | /fr/a-propos |
    Then correct content should display
    And language should be detected

  Scenario: SEO Metadata Translation
    Given content has SEO metadata
    When translated to other languages
    Then meta tags should be translated:
      | Title | Description | Keywords |
    And hreflang tags should be added

  Scenario: Multilingual Sitemap
    Given site has multiple languages
    When sitemap is generated
    Then should include all language versions
    With proper hreflang annotations

  Scenario: 404 Error Pages
    Given page doesn't exist in language
    When user visits missing page
    Then 404 should display in user's language
    With translated error message
```

### 5. Date, Number & Currency Formatting
```gherkin
Feature: Localization Formatting
  As a user
  I want proper formatting
  For my locale

  Scenario: Date Formatting
    Given different date formats:
      | US | MM/DD/YYYY |
      | EU | DD/MM/YYYY |
      | ISO | YYYY-MM-DD |
    When date is displayed
    Then format should match locale

  Scenario: Number Formatting
    Given number 1234.56
    When displayed in different locales:
      | US | 1,234.56 |
      | DE | 1.234,56 |
      | FR | 1 234,56 |
    Then formatting should be correct

  Scenario: Currency Display
    Given price in USD
    When displayed in different regions:
      | US | $1,234.56 |
      | EU | $1.234,56 |
      | UK | $1,234.56 |
    Then currency should format correctly

  Scenario: Timezone Handling
    Given content with timestamps
    When user is in different timezone
    Then times should display in user's timezone
    With proper offset calculation
```

## API Testing

### Locale API Endpoints
```javascript
// GET /api/languages
// Response: List of available languages

// POST /api/languages
{
  "name": "Português",
  "code": "pt",
  "direction": "ltr",
  "status": "enabled"
}
// Response: 201 Created

// PUT /api/languages/{code}
{
  "status": "disabled"
}
// Response: 200 OK

// GET /api/translations/{locale}
// Response: All translations for locale

// PUT /api/translations/{locale}
{
  "Welcome": "Bienvenido",
  "Goodbye": "Adiós"
}
// Response: 200 OK

// POST /api/content/{id}/translate
{
  "language": "es",
  "title": "Título en español",
  "body": "Contenido en español"
}
// Response: 201 Created

// GET /api/content/{id}/translations
// Response: All translations of content
```

## Performance Requirements

### Response Times
- Language detection: < 50ms
- Translation lookup: < 10ms (cached)
- Language switch: < 200ms
- Translation import: < 5 seconds per 1000 strings

### Caching Strategy
- Translation strings: Permanent cache until update
- Language detection: Session cache
- Locale files: File system cache
- Database translations: Memory cache

## Edge Cases & Validation

### Language Configuration
- [ ] Invalid language codes
- [ ] Duplicate language entries
- [ ] Missing translation files
- [ ] Corrupted PO files
- [ ] Mixed text directions (LTR/RTL)
- [ ] Unsupported character sets

### Translation Issues
- [ ] Missing translation strings
- [ ] Incomplete pluralization
- [ ] Special character encoding
- [ ] Very long translations
- [ ] HTML in translation strings
- [ ] Variable interpolation

### URL Handling
- [ ] Language prefix conflicts
- [ ] Duplicate slugs across languages
- [ ] Special characters in URLs
- [ ] URL encoding issues
- [ ] Canonical URL handling

## Security Testing

### Translation Security
- [ ] XSS in translation strings
- [ ] HTML injection prevention
- [ ] File upload validation (PO files)
- [ ] Path traversal in locale files
- [ ] Script injection in imports

### Language Switching
- [ ] URL manipulation
- [ ] Session fixation
- [ ] CSRF in language switch
- [ ] Redirect validation

## Migration Validation

### CakePHP 3 → 5 Checks
- [ ] I18n behavior migration
- [ ] Locale file loading
- [ ] Translation function updates
- [ ] Number/date helpers
- [ ] Validation message translation
- [ ] Form helper translation

## Test Data Requirements

### Languages
```sql
INSERT INTO languages (name, code, direction, status) VALUES
('English', 'en', 'ltr', 'enabled'),
('Español', 'es', 'ltr', 'enabled'),
('Français', 'fr', 'ltr', 'enabled'),
('Deutsch', 'de', 'ltr', 'disabled');
```

### Sample Translations
```po
# Spanish translations
msgid "Welcome"
msgstr "Bienvenido"

msgid "Hello %s"
msgstr "Hola %s"

msgid "One item"
msgid_plural "%d items"
msgstr[0] "Un elemento"
msgstr[1] "%d elementos"
```

## Success Metrics

- All languages properly configured
- Translation interface functional
- Content translation working
- Language detection accurate
- URL structure correct
- Formatting rules applied
- Performance within limits
- No character encoding issues
- SEO metadata translated
- API endpoints operational
- Migration successful
- Zero data loss