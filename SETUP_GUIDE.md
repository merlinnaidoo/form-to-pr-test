# GitHub Form to PR Setup Guide

Complete guide to set up a form submission system that automatically creates Pull Requests.

## What This Does

Users fill out a GitHub issue form → GitHub Action automatically creates a PR → Maintainers review and merge → Issue closes automatically.

**Perfect for:** Non-technical contributors who want to submit content without knowing Git.

---

## Setup Steps

### 1. Create a Public Repository

GitHub Pages (needed for the form) only works on public repos with free accounts.

```bash
# Create repo via GitHub UI or API
```

### 2. Enable GitHub Pages

1. Go to: `https://github.com/YOUR_USERNAME/YOUR_REPO/settings/pages`
2. Under "Source" → Select **"Deploy from a branch"**
3. Branch: **"main"** → Folder: **"/docs"**
4. Click **"Save"**
5. Wait 1-2 minutes for deployment

Your form will be live at: `https://YOUR_USERNAME.github.io/YOUR_REPO/`

### 3. Enable GitHub Actions Permissions

**CRITICAL STEP** - Without this, the workflow will fail.

1. Go to: `https://github.com/YOUR_USERNAME/YOUR_REPO/settings/actions`
2. Scroll to "Workflow permissions"
3. Select **"Read and write permissions"**
4. Check ✅ **"Allow GitHub Actions to create and approve pull requests"**
5. Click **"Save"**

---

## File Structure

```
your-repo/
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   └── submission-form.yml          # The form definition
│   └── workflows/
│       └── issue-to-pr.yml              # Automation workflow
├── docs/
│   └── index.html                       # Optional: Custom web form
├── submissions/                         # Where submissions are stored
└── README.md                            # With contribution button
```

---

## Key Files Explained

### 1. Issue Template (`.github/ISSUE_TEMPLATE/submission-form.yml`)

Defines the form fields users fill out.

```yaml
name: Submit Entry
description: Submit your entry
title: "[Entry]: "
labels: ["submission"]
body:
  - type: dropdown
    id: category
    attributes:
      label: Category
      options:
        - Option 1
        - Option 2
    validations:
      required: true
  
  - type: input
    id: title
    attributes:
      label: Title
      placeholder: Enter title
    validations:
      required: true
```

### 2. Workflow (`.github/workflows/issue-to-pr.yml`)

Automates the conversion from issue to PR.

```yaml
name: Convert Issue to PR

on:
  issues:
    types: [opened]

jobs:
  create-pr:
    if: contains(github.event.issue.labels.*.name, 'submission')
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Create submission file
        run: |
          mkdir -p submissions
          cat > "submissions/entry-${{ github.event.issue.number }}.md" << 'EOF'
          # Entry from Issue #${{ github.event.issue.number }}

          ${{ github.event.issue.body }}
          EOF

      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v5
        with:
          title: "New Entry: ${{ github.event.issue.title }}"
          body: |
            Auto-generated from issue #${{ github.event.issue.number }}
            
            Closes #${{ github.event.issue.number }}
          branch: "submission-${{ github.event.issue.number }}"
          base: "main"
          commit-message: "Add entry from issue #${{ github.event.issue.number }}"
          labels: submission

      - name: Close issue
        uses: actions/github-script@v6
        with:
          script: |
            await github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: '✅ Your submission has been converted to a Pull Request!'
            });
            
            await github.rest.issues.update({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              state: 'closed'
            });
```

### 3. README Button

Add a prominent button linking to the form:

```markdown
## 🚀 [Submit Your Entry](https://github.com/YOUR_USERNAME/YOUR_REPO/issues/new?assignees=&labels=submission&template=submission-form.yml&title=%5BEntry%5D%3A+) 🚀

No Git knowledge required - just fill out the form!
```

---

## How It Works

### User Flow

1. User clicks "Submit Entry" button in README
2. Fills out the GitHub issue form
3. Submits the form
4. GitHub Action triggers automatically
5. Action creates a file with the submission
6. Action creates a Pull Request
7. Action closes the issue with a comment
8. Maintainer reviews and merges the PR

### Behind the Scenes

1. **Issue Created** → Has `submission` label (from template)
2. **Workflow Triggers** → Only on issues with `submission` label
3. **File Created** → In `submissions/` directory
4. **Branch Created** → Named `submission-{issue-number}`
5. **PR Created** → Links back to original issue with "Closes #X"
6. **Issue Closed** → Automatically, so it doesn't clutter issues list

---

## Customization

### Change Form Fields

Edit `.github/ISSUE_TEMPLATE/submission-form.yml`:

- `dropdown` - Multiple choice
- `input` - Single line text
- `textarea` - Multi-line text
- `checkboxes` - Multiple selections

### Change File Location

In workflow, change:
```yaml
mkdir -p submissions
cat > "submissions/entry-${{ github.event.issue.number }}.md"
```

To your preferred directory.

### Change Label Name

Replace `submission` with your label throughout:
- Issue template: `labels: ["your-label"]`
- Workflow: `if: contains(github.event.issue.labels.*.name, 'your-label')`

---

## Troubleshooting

### Issue: Workflow doesn't trigger

**Solution:** Check that:
- Issue has the correct label (`submission`)
- Workflow file is in `.github/workflows/`
- Workflow file has correct YAML syntax

### Issue: "GitHub Actions is not permitted to create or approve pull requests"

**Solution:** Enable in Settings → Actions → Workflow permissions (see Step 3 above)

### Issue: No PR created but workflow succeeds

**Solution:** Check that files are actually being created:
- Add `ls -la` commands in workflow for debugging
- Check the workflow logs for file creation output

### Issue: Form doesn't show up

**Solution:** 
- Ensure file is at `.github/ISSUE_TEMPLATE/your-form.yml`
- Check YAML syntax is valid
- Try accessing directly: `https://github.com/OWNER/REPO/issues/new/choose`

---

## Testing

1. Click your submission button
2. Fill out the form with test data
3. Submit
4. Check:
   - ✅ Issue created
   - ✅ Workflow runs (Actions tab)
   - ✅ PR created
   - ✅ Issue closed
   - ✅ File appears in PR

---

## Security Notes

- ✅ No tokens in browser code
- ✅ GitHub handles all authentication
- ✅ Workflow runs with `GITHUB_TOKEN` (automatic)
- ✅ Users need GitHub account to submit
- ✅ Maintainers review before merging

---

## Applying to Your Prompt Library

When ready to make your prompt library public:

1. Make repository public
2. Follow steps 2-3 above (GitHub Pages + Actions permissions)
3. Replace form fields with prompt-specific fields:
   - Prompt Title
   - Well-Architected Pillar (dropdown)
   - Description
   - Use Case
   - Prerequisites
   - Prompt Text
   - Example Output
   - Tags
   - Author

4. Update workflow to create files in appropriate pillar directories
5. Add contribution button to README

---

## Resources

- [GitHub Issue Forms](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/syntax-for-issue-forms)
- [GitHub Actions Permissions](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token)
- [peter-evans/create-pull-request](https://github.com/peter-evans/create-pull-request)

---

## Support

For issues with this setup:
1. Check workflow logs in Actions tab
2. Verify all setup steps completed
3. Test with minimal form first
4. Open an issue if stuck
