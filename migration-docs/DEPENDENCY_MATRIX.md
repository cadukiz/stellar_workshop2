# CakePHP 5 Dependency Compatibility Matrix

## Overview

This document analyzes the compatibility of QuickAppsCMS (CakePHP 3) dependencies with CakePHP 5, providing migration paths and alternatives for each component.

## Current Dependencies Analysis

### Core Application Dependencies (quickapps-cakephp3/src/composer.json)

| Dependency | Current Version | CakePHP 5 Compatible | Migration Path | Status | Notes |
|------------|----------------|---------------------|----------------|---------|-------|
| `cakephp/cakephp` | 3.3.16 | ✅ | Upgrade to ^5.0.0 | **CRITICAL** | Core framework upgrade |
| `mobiledetect/mobiledetectlib` | 2.* | ✅ | Upgrade to ^3.74 | **READY** | Already upgraded in CakePHP 5 target |
| `cakephp/plugin-installer` | * | ✅ | Upgrade to ^2.0 | **READY** | Already upgraded in CakePHP 5 target |
| `quickapps-plugins/installer` | * | ❌ | **REWRITE REQUIRED** | **BLOCKER** | QuickApps-specific, needs full rewrite |

### Critical Third-Party Dependency

| Dependency | Current Version | CakePHP 5 Compatible | Migration Path | Status | Risk Level |
|------------|----------------|---------------------|----------------|---------|-------------|
| `goaop/framework` | 2.1.2 | ❓ | **INVESTIGATION NEEDED** | **HIGH RISK** | 🔴 **CRITICAL** |

**GoAOP Framework Analysis:**
- **Purpose**: Aspect-Oriented Programming framework used by CMS plugin
- **Risk**: Core dependency that affects system architecture
- **PHP Compatibility**: May not support PHP 8.1+ (required by CakePHP 5)
- **Alternatives**: Consider removing AOP patterns or implementing with modern PHP approaches
- **Impact**: **ARCHITECTURAL** - affects core CMS functionality

### QuickApps Plugin Dependencies

#### Foundation Plugins (Zero External Dependencies)
| Plugin | External Dependencies | Migration Status |
|--------|----------------------|------------------|
| `eav` | None | ✅ **PORTABLE** |

#### Core Infrastructure Plugins
| Plugin | Dependencies | CakePHP 5 Status | Migration Priority |
|--------|--------------|------------------|-------------------|
| `cms` | `goaop/framework: 2.1.2` | ❌ **BLOCKER** | 🔴 **P0** |
| `field` | `cms`, `eav` | ⚠️ **DEPENDS ON CMS** | 🟠 **P1** |
| `user` | `cms`, `field`, `locale`, `block` | ⚠️ **DEPENDS ON CMS** | 🟠 **P1** |

#### Feature Plugins
| Plugin | Dependencies | Migration Complexity | Priority |
|--------|--------------|---------------------|----------|
| `block` | `cms`, `locale`, `user` | Medium | 🟡 **P2** |
| `menu` | `cms`, `block` | Medium | 🟡 **P2** |
| `content` | `cms`, `comment`, `field`, `locale`, `user`, `search`, `block` | High | 🟠 **P1** |
| `taxonomy` | `cms`, `field`, `block` | Medium | 🟡 **P2** |
| `comment` | `cms`, `field`, `captcha`, `user` | Medium | 🟡 **P2** |
| `search` | `cms`, `ext-iconv` | Low | 🟢 **P3** |

#### UI/UX Plugins
| Plugin | Dependencies | Migration Complexity | Priority |
|--------|--------------|---------------------|----------|
| `locale` | `cms`, `block` | Medium | 🟡 **P2** |
| `media-manager` | `cms` | Low | 🟢 **P3** |
| `wysiwyg` | `cms` | Low | 🟢 **P3** |
| `bootstrap` | `cms` | Low | 🟢 **P3** |
| `jquery` | `cms` | Low | 🟢 **P3** |
| `captcha` | `cms` | Low | 🟢 **P3** |

#### System Plugins
| Plugin | Dependencies | Notes | Priority |
|--------|--------------|-------|----------|
| `system` | `cms`, `locale`, `installer` | System configuration | 🟡 **P2** |
| `installer` | **ALL PLUGINS** | Meta-package, needs rewrite | 🔴 **P0** |

## PHP Extension Dependencies

| Extension | Current Status | CakePHP 5 Status | Migration Action |
|-----------|----------------|------------------|------------------|
| `ext-iconv` | Required by search plugin | ✅ **COMPATIBLE** | None |

## Migration Strategy Recommendations

### Phase 1: Critical Blockers Resolution 🔴
1. **Investigate GoAOP Framework compatibility** with PHP 8.1+
2. **Evaluate AOP usage** in CMS plugin - can it be refactored?
3. **Plan CMS plugin rewrite** if GoAOP is incompatible
4. **Rewrite installer plugin** for CakePHP 5 architecture

### Phase 2: Foundation Migration 🟠
1. Migrate `eav` plugin (no external dependencies)
2. Migrate `cms` plugin (after GoAOP resolution)
3. Migrate `field` plugin (depends on cms + eav)
4. Migrate `user` plugin (core authentication)

### Phase 3: Feature Migration 🟡
1. Migrate content management plugins (`content`, `block`, `menu`)
2. Migrate internationalization (`locale`)
3. Migrate taxonomy and commenting systems

### Phase 4: UI/Enhancement Migration 🟢
1. Migrate media management and WYSIWYG
2. Migrate UI frameworks (Bootstrap, jQuery)
3. Migrate search functionality
4. Migrate captcha system

## Compatibility Matrix Summary

### ✅ **Ready for Migration** (4 dependencies)
- `cakephp/cakephp` → Version 5.x
- `mobiledetect/mobiledetectlib` → Already upgraded
- `cakephp/plugin-installer` → Already upgraded  
- `ext-iconv` → Native compatibility

### ❓ **Investigation Required** (1 dependency)
- `goaop/framework` → **CRITICAL PATH BLOCKER**

### ❌ **Requires Rewrite** (19 dependencies)
- All QuickApps plugins → Custom architecture needs adaptation
- `quickapps-plugins/installer` → Meta-package rewrite

## Risk Assessment

### 🔴 **Critical Risks**
- **GoAOP Framework**: May break with PHP 8.1+, affects core CMS functionality
- **Plugin Architecture**: All 17 plugins need CakePHP 5 adaptation
- **AOP Patterns**: May need architectural refactoring

### 🟠 **High Risks**  
- **Plugin Interdependencies**: Complex dependency chain requires careful migration order
- **Custom Auth System**: User authentication may need significant changes
- **EAV Model**: Database relationships might need updates

### 🟡 **Medium Risks**
- **Template System**: Back/Common/Front structure needs validation
- **Event System**: Plugin communication patterns may need updates

## Next Actions

1. **IMMEDIATE**: Test `goaop/framework` compatibility with PHP 8.1+
2. **HIGH PRIORITY**: Analyze CMS plugin AOP usage patterns
3. **PLANNING**: Design plugin migration sequence based on dependency order
4. **PREPARATION**: Set up isolated testing environment for plugin migration

## Migration Timeline Estimate

- **Phase 1 (Blockers)**: 4-6 weeks
- **Phase 2 (Foundation)**: 6-8 weeks  
- **Phase 3 (Features)**: 8-10 weeks
- **Phase 4 (UI/Enhancement)**: 4-6 weeks

**Total Estimated Duration**: 22-30 weeks

## Compatibility Legend

- ✅ **Compatible**: Ready for CakePHP 5
- ❓ **Investigation Needed**: Requires compatibility testing
- ⚠️ **Conditional**: Depends on other components
- ❌ **Incompatible**: Requires rewrite or alternative
- 🔴 **P0**: Critical path blocker
- 🟠 **P1**: High priority
- 🟡 **P2**: Medium priority  
- 🟢 **P3**: Low priority