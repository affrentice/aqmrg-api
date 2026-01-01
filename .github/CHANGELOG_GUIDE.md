# CHANGELOG Maintenance Guide

This guide explains how to maintain the CHANGELOG.md file for the AQMRG AI Analytics Backend Platform.

## 📋 Overview

The CHANGELOG.md follows the [Keep a Changelog](https://keepachangelog.com/en/1.0.0/) format and adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## 🎯 Purpose

The changelog serves to:

- Document all notable changes in the project
- Help users and developers understand what changed between versions
- Provide a historical record of the project's evolution
- Make it easy to find when a specific feature was added or bug was fixed

## 📝 Format Structure

```markdown
# Changelog

## [Unreleased]

### Added

- New features not yet released

### Changed

- Changes to existing features

### Fixed

- Bug fixes

## [X.Y.Z] - YYYY-MM-DD

### Added

- New features in this release

### Changed

- Changes to existing features

### Deprecated

- Features that will be removed soon

### Removed

- Features that were removed

### Fixed

- Bug fixes

### Security

- Security improvements
```

## 🔢 Versioning

This project uses [Semantic Versioning](https://semver.org/):

### Version Format: MAJOR.MINOR.PATCH

- **MAJOR** (X.0.0): Breaking changes requiring migration
- **MINOR** (0.X.0): New features, backward compatible
- **PATCH** (0.0.X): Bug fixes, backward compatible

### Examples:

```
0.1.0 → 0.2.0   (New feature added)
0.2.0 → 0.2.1   (Bug fix)
0.2.1 → 1.0.0   (Breaking change, production ready)
1.0.0 → 1.1.0   (New feature)
1.1.0 → 2.0.0   (Breaking change)
```

## 📂 Categories

### Added

For new features.

**Example:**

```markdown
### Added

- API Gateway service with request routing and rate limiting
- JWT authentication in Auth Service
- Real-time sensor data ingestion from AirQo sensors
- 24-hour air quality predictions
```

### Changed

For changes in existing functionality.

**Example:**

```markdown
### Changed

- Updated prediction model from v1.0 to v2.0 for improved accuracy
- Modified database schema to support additional sensor types
- Improved API response time by implementing Redis caching
```

### Deprecated

For features that will be removed in upcoming releases.

**Example:**

```markdown
### Deprecated

- Legacy `/api/v1/sensors/list` endpoint (use `/api/v1/sensors` instead)
- Old authentication flow (will be removed in v2.0.0)
```

### Removed

For features removed in this release.

**Example:**

```markdown
### Removed

- Deprecated `/api/v1/old-endpoint` removed (use `/api/v2/new-endpoint`)
- Support for Python 3.8 (minimum version now 3.10)
```

### Fixed

For bug fixes.

**Example:**

```markdown
### Fixed

- Fixed memory leak in data ingestion service
- Corrected AQI calculation for PM10 values above 150
- Resolved authentication token refresh issue
- Fixed timezone handling in prediction timestamps
```

### Security

For security improvements.

**Example:**

```markdown
### Security

- Updated dependencies to patch CVE-2024-XXXXX
- Implemented rate limiting on authentication endpoints
- Added input validation to prevent SQL injection
- Enabled HTTPS-only mode for all production APIs
```

## 🔄 Workflow

### During Development (Unreleased Section)

1. **As you work**, add entries to the `[Unreleased]` section:

```markdown
## [Unreleased]

### Added

- New export service for CSV and Excel data exports (#123)
- Health impact prediction endpoint (#125)

### Fixed

- Memory leak in Kafka consumer (#124)
```

2. **Group by category**, not by pull request or commit

3. **Link to issues/PRs** using GitHub issue numbers

### Before Release

1. **Decide the version number** based on changes:

   - Breaking changes? → Bump MAJOR
   - New features? → Bump MINOR
   - Bug fixes only? → Bump PATCH

2. **Create a new version section**:

```markdown
## [0.2.0] - 2026-01-15

### Added

- Export service for CSV and Excel data exports (#123)
- Health impact prediction endpoint (#125)

### Fixed

- Memory leak in Kafka consumer (#124)

## [0.1.0] - 2026-01-01

...
```

3. **Clear the Unreleased section** (move everything to the version)

4. **Update the comparison links** at the bottom:

```markdown
[Unreleased]: https://github.com/affrentice/aqmrg-api/compare/v0.2.0...HEAD
[0.2.0]: https://github.com/affrentice/aqmrg-api/compare/v0.1.0...v0.2.0
[0.1.0]: https://github.com/affrentice/aqmrg-api/releases/tag/v0.1.0
```

## ✍️ Writing Good Entries

### DO ✅

- **Be specific**: "Added JWT refresh token support" not "Auth improvements"
- **Use present tense**: "Add feature" not "Added feature" or "Adds feature"
- **Link to issues**: "Fixed login bug (#123)"
- **Group related changes**: Combine related items under one bullet
- **Explain impact**: Mention breaking changes clearly

### DON'T ❌

- **Don't use generic descriptions**: "Various fixes" or "Code improvements"
- **Don't include internal changes**: Refactoring unless it affects users
- **Don't add every commit**: Only notable changes
- **Don't forget issue links**: Always reference issues/PRs
- **Don't mix categories**: Keep fixes separate from features

## 📝 Examples

### Good Entry ✅

```markdown
### Added

- Real-time air quality predictions with 4hr, 24hr, and 72hr forecasts (#145)
- Kafka integration for sensor data streaming from multiple manufacturers (#146)
- Admin dashboard for model deployment and monitoring (#147)

### Changed

- Improved prediction accuracy by 15% using ensemble models (#148)
- Updated API response format to include confidence scores (BREAKING CHANGE) (#149)

### Fixed

- Resolved authentication token expiration issue affecting long-running sessions (#150)
- Corrected PM2.5 to AQI conversion formula for values above 200 (#151)

### Security

- Updated all dependencies to patch security vulnerabilities (#152)
- Implemented rate limiting on public endpoints (100 req/15min) (#153)
```

### Bad Entry ❌

```markdown
### Changed

- Various improvements
- Bug fixes
- Updated code
- Made things better
```

## 🔧 Automation (Optional)

### Auto-generate from PRs

You can use tools to auto-generate changelog entries:

1. **conventional-changelog**

```bash
npm install -g conventional-changelog-cli
conventional-changelog -p angular -i CHANGELOG.md -s
```

2. **GitHub Actions**

```yaml
name: Update Changelog
on:
  pull_request:
    types: [closed]

jobs:
  update:
    if: github.event.pull_request.merged == true
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Update Changelog
        run: |
          # Add PR to unreleased section
          echo "- ${{ github.event.pull_request.title }} (#${{ github.event.pull_request.number }})" >> CHANGELOG.md
```

## 📅 Release Checklist

Before each release:

- [ ] Move all `[Unreleased]` items to new version section
- [ ] Verify version number follows semantic versioning
- [ ] Add release date in YYYY-MM-DD format
- [ ] Update comparison links at bottom of file
- [ ] Review for clarity and completeness
- [ ] Ensure all entries link to issues/PRs
- [ ] Check for proper categorization
- [ ] Commit changelog with release
- [ ] Create Git tag matching version
- [ ] Create GitHub release with changelog excerpt

## 🎯 Template for New Release

```markdown
## [X.Y.Z] - YYYY-MM-DD

### Added

-

### Changed

-

### Deprecated

-

### Removed

-

### Fixed

-

### Security

-
```

## 📊 Changelog at Different Stages

### Pre-1.0.0 (Development)

```markdown
## [0.3.0] - 2026-02-01

### Added

- Basic features

## [0.2.0] - 2026-01-15

### Added

- More features

## [0.1.0] - 2026-01-01

### Added

- Initial release
```

### Post-1.0.0 (Production)

```markdown
## [2.0.0] - 2026-06-01

### Changed

- BREAKING: New API authentication method

## [1.5.0] - 2026-05-01

### Added

- New analytics features

## [1.4.1] - 2026-04-15

### Fixed

- Critical security patch
```

## 🔗 Resources

- [Keep a Changelog](https://keepachangelog.com/en/1.0.0/)
- [Semantic Versioning](https://semver.org/spec/v2.0.0.html)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging)

## ❓ Common Questions

**Q: Should I include every commit?**  
A: No, only notable changes that affect users or developers.

**Q: How often should I update the changelog?**  
A: Update `[Unreleased]` as you work, create version sections at release time.

**Q: What if I forgot to add something?**  
A: Add it to the appropriate version section and note it was retroactively added.

**Q: Should internal refactoring be included?**  
A: Only if it significantly affects performance or behavior.

**Q: How to handle breaking changes?**  
A: Mark them clearly with "BREAKING CHANGE" or "BREAKING" prefix.

## 📧 Questions?

For questions about changelog maintenance:

- Email: hello@affrentice.com
- Issues: https://github.com/affrentice/aqmrg-api/issues

---

**Remember**: A good changelog makes life easier for everyone! 🎉
