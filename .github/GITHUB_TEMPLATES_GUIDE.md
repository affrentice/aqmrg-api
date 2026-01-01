# GitHub Templates Setup Guide

This guide explains the GitHub issue and pull request templates included in this repository.

## 📁 Template Structure

```
.github/
├── ISSUE_TEMPLATE/
│   ├── bug_report.md           # Bug report template
│   ├── feature_request.md      # Feature request template
│   ├── documentation.md        # Documentation improvement template
│   ├── question.md             # Question/help template
│   └── config.yml              # Issue template configuration
└── pull_request_template.md   # Default PR template
```

## 🎯 How Templates Work

### Issue Templates

When users click "New Issue" on GitHub, they'll see a menu with these options:

1. **🐛 Bug Report** - Report bugs or unexpected behavior
2. **✨ Feature Request** - Suggest new features or enhancements
3. **📚 Documentation** - Request documentation improvements
4. **❓ Question** - Ask questions or get help

### Pull Request Template

When creating a new PR, the template automatically loads with sections for:

- Description and motivation
- Related issues
- Type of change
- Affected services
- Testing information
- Breaking changes
- Checklist

## 🚀 Setup Instructions

### Step 1: Verify Templates Location

Templates are already in the correct location:

```bash
.github/ISSUE_TEMPLATE/
.github/pull_request_template.md
```

### Step 2: Commit Templates to Repository

```bash
# Add GitHub templates
git add .github/

# Commit templates
git commit -m "Add GitHub issue and PR templates"

# Push to your repository
git push origin feat-monorepo
```

### Step 3: Verify on GitHub

1. Go to your repository on GitHub
2. Click on **"Issues"** tab
3. Click **"New Issue"** button
4. You should see the template chooser with all 4 options

For PR template:

1. Create a new branch and make changes
2. Open a pull request
3. The PR template will auto-populate

## 📝 Template Files Overview

### Bug Report (`bug_report.md`)

**Sections:**

- Bug description
- Steps to reproduce
- Expected vs actual behavior
- Screenshots
- Environment details (service, OS, version)
- Logs/error messages
- Possible solution
- Checklist

**Labels:** Automatically adds `bug` label

### Feature Request (`feature_request.md`)

**Sections:**

- Feature description
- Problem it solves
- Proposed solution
- Alternatives considered
- Impact assessment
- Implementation details
- Affected services
- Checklist

**Labels:** Automatically adds `enhancement` label

### Documentation (`documentation.md`)

**Sections:**

- Documentation issue description
- Current state
- Suggested improvement
- Location (files/URLs)
- Type of documentation
- Priority
- Checklist

**Labels:** Automatically adds `documentation` label

### Question (`question.md`)

**Sections:**

- Question/topic
- What you've tried
- Context
- Expected information
- Related documentation
- Checklist

**Labels:** Automatically adds `question` label

### Pull Request Template (`pull_request_template.md`)

**Sections:**

- Description (what & why)
- Related issues
- Type of change (bug fix, feature, docs, etc.)
- Affected services
- Testing approach
- Breaking changes
- Additional notes
- Checklist

## 🎨 Customizing Templates

### Adding New Issue Templates

1. Create a new `.md` file in `.github/ISSUE_TEMPLATE/`
2. Add front matter with metadata:

```yaml
---
name: Template Name
about: Template description
title: "[PREFIX] "
labels: label1, label2
assignees: username
---
```

3. Add your template content using Markdown

### Modifying Existing Templates

Edit the `.md` files directly. Changes will be reflected immediately after committing.

### Customizing the Issue Template Chooser

Edit `.github/ISSUE_TEMPLATE/config.yml`:

```yaml
blank_issues_enabled: false
contact_links:
  - name: 📧 Email Support
    url: mailto:hello@affrentice.com
    about: Contact us directly for private issues
  - name: 📖 Documentation
    url: https://docs.aqmrg.org
    about: Check our documentation first
```

## 🏷️ Labels Configuration

GitHub will automatically create these labels when issues are created:

- `bug` - Bug reports
- `enhancement` - Feature requests
- `documentation` - Documentation issues
- `question` - Questions and help

### Recommended Additional Labels

Create these labels in your repository settings:

**Priority:**

- `priority: critical` (Red: #d73a4a)
- `priority: high` (Orange: #ff9800)
- `priority: medium` (Yellow: #ffeb3b)
- `priority: low` (Blue: #2196f3)

**Status:**

- `status: needs-triage` (Purple: #9c27b0)
- `status: in-progress` (Green: #4caf50)
- `status: blocked` (Red: #f44336)
- `status: ready-for-review` (Teal: #009688)

**Services:**

- `service: api-gateway`
- `service: auth`
- `service: data-ingestion`
- `service: model-serving`
- `service: analytics`
- `service: notifications`
- `service: sensors`
- `service: export`

**Type:**

- `type: infrastructure`
- `type: security`
- `type: performance`
- `type: refactor`

## 📋 Best Practices

### For Issue Templates:

1. **Keep them concise** - Don't overwhelm users with too many fields
2. **Use clear language** - Avoid jargon where possible
3. **Include examples** - Show users what good input looks like
4. **Add helpful hints** - Use HTML comments for guidance
5. **Make critical fields required** - Use validation where needed

### For PR Templates:

1. **Link to issues** - Always reference related issues
2. **Require testing** - Make testing a mandatory checklist item
3. **Document breaking changes** - Make this prominent
4. **List affected services** - Help reviewers understand scope
5. **Include deployment notes** - Help operations team

## 🔄 Workflow Integration

### Automated Workflows

You can create GitHub Actions that trigger based on templates:

```yaml
# .github/workflows/label-issues.yml
name: Label Issues
on:
  issues:
    types: [opened]

jobs:
  label:
    runs-on: ubuntu-latest
    steps:
      - name: Label bug reports
        if: contains(github.event.issue.title, '[BUG]')
        run: |
          gh issue edit ${{ github.event.issue.number }} \
            --add-label "needs-triage"
```

### PR Template Enforcement

Create a workflow to ensure PRs follow the template:

```yaml
# .github/workflows/pr-validation.yml
name: PR Validation
on:
  pull_request:
    types: [opened, edited]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Check PR description
        run: |
          if [ -z "${{ github.event.pull_request.body }}" ]; then
            echo "PR description is empty"
            exit 1
          fi
```

## 📚 Additional Resources

- [GitHub Issue Templates Documentation](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/configuring-issue-templates-for-your-repository)
- [Pull Request Templates](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository)
- [Keep a Changelog](https://keepachangelog.com/)
- [Semantic Versioning](https://semver.org/)

## ✅ Verification Checklist

After setting up templates:

- [ ] Templates are in `.github/` directory
- [ ] Templates are committed to the repository
- [ ] Issue template chooser appears on GitHub
- [ ] PR template auto-loads when creating PRs
- [ ] Labels are created and configured
- [ ] Team is familiar with template usage
- [ ] Templates are documented in README

## 🆘 Troubleshooting

**Templates not appearing:**

- Ensure templates are in `.github/ISSUE_TEMPLATE/` directory
- Check YAML front matter is valid
- Templates must be committed to default branch

**PR template not loading:**

- File must be named `pull_request_template.md`
- Must be in `.github/` directory
- Clear browser cache and try again

**Labels not applying:**

- Check label names match exactly
- Ensure labels exist in repository
- YAML syntax must be correct

---

**Need Help?** Contact hello@affrentice.com or visit https://docs.aqmrg.org
