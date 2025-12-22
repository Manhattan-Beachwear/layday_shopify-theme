# The Leisure Collective Theme - Based on Shopify's Horizon theme


### Theme Customizations

**File**: `theme-customizations.md`

Comprehensive documentation for all customizations made to the Horizon theme, including their purpose, strategy, implementation details, and affected files.

**Key Topics**:

- Combined Listing Variant Picker
- Custom Accordion Component
- Custom Disclosure Component
- Product Custom Property Block
- Product Title Processing
- Variant Picker Utilities
- Cross-references and integration points
- Maintenance notes and testing considerations


[Architecture Overview](#architecture-overview) |
[Core Maintenance](#section-a-core-maintenance-updating-the-parent) |
[Store Deployment](#section-b-store-deployment-updating-children) |
[Troubleshooting](#troubleshooting) |
[Developer Tools](#developer-tools) |
[Contributing](#contributing) |
[License](#license)

Horizon is the flagship of a new generation of first party Shopify themes. It incorporates the latest Liquid Storefronts features, including [theme blocks](https://shopify.dev/docs/storefronts/themes/architecture/blocks/theme-blocks/quick-start?framework=liquid).

- **Web-native in its purest form:** Themes run on the [evergreen web](https://www.w3.org/2001/tag/doc/evergreen-web/). We leverage the latest web browsers to their fullest, while maintaining support for the older ones through progressive enhancement—not polyfills.
- **Lean, fast, and reliable:** Functionality and design defaults to “no” until it meets this requirement. Code ships on quality. Themes must be built with purpose. They shouldn’t support each and every feature in Shopify.
- **Server-rendered:** HTML must be rendered by Shopify servers using Liquid. Business logic and platform primitives such as translations and money formatting don’t belong on the client. Async and on-demand rendering of parts of the page is OK, but we do it sparingly as a progressive enhancement.
- **Functional, not pixel-perfect:** The Web doesn’t require each page to be rendered pixel-perfect by each browser engine. Using semantic markup, progressive enhancement, and clever design, we ensure that themes remain functional regardless of the browser.

## Architecture Overview

The Leisure Collective theme system follows a three-tier deployment architecture:

```
┌─────────────────────────────────────────┐
│   Shopify Horizon (Grandparent)        │
│   Base theme managed by Shopify         │
│   https://github.com/Shopify/horizon-   │
│   private.git                           │
└─────────────────┬───────────────────────┘
                  │
                  │ Updates pulled by Core Maintainers
                  │
┌─────────────────▼───────────────────────┐
│   TLC Core (Parent)                     │
│   tlc_shopify_theme                     │
│   Our customized version of Horizon     │
│   This repository                       │
└─────────────────┬───────────────────────┘
                  │
                  │ Updates pulled by Store Developers
                  │
      ┌───────────┼───────────┐
      │           │           │
┌─────▼─────┐ ┌──▼───┐ ┌─────▼─────┐
│ creatures │ │ otis │ │   sito    │
│ of leisure│ │      │ │           │
│           │ │      │ │           │
└───────────┘ └──────┘ └───────────┘
  (Children)    (Children)  (Children)
```

### Code Flow

**Grandparent → Parent (Core)**
- Shopify releases updates to Horizon
- Core maintainers merge Horizon updates into `tlc_shopify_theme`
- Customizations and shared components are maintained in the Parent

**Parent → Children (Stores)**
- Store developers merge Parent updates into store-specific repos
- Store-specific templates and settings are protected from overwrite
- Each store maintains its own configuration while receiving shared updates

### File Ownership

**Parent (Core) Owns:**
- `assets/` - CSS, JavaScript, and images shared across stores
- `snippets/` - Shared components and partials
- `sections/*.liquid` - Shared sections and section schemas
- `locales/` - Shared translations
- `config/settings_schema.json` - Shared settings schema
- `layout/password.liquid` - Shared password page layout

**Store (Child) Owns:**
- `templates/*.json` - Store-specific page layouts and template configurations
- `config/settings_data.json` - Store-specific theme settings (colors, fonts, etc.)
- `layout/theme.liquid` - Store-specific scripts and meta tags

**Hard Rule:** Parent updates must never overwrite Child-owned files.

---

## Section A: Core Maintenance (Updating the Parent)

**Target Audience:** Senior developers managing `tlc_shopify_theme`

**Goal:** Pull updates from Shopify's Horizon into our Core theme while preserving customizations.

### One-Time Setup

Add the Horizon repository as a remote:

```bash
git remote add horizon https://github.com/Shopify/horizon-private.git
```

Verify the remote was added:

```bash
git remote -v
```

You should see both `origin` (pointing to `tlc_shopify_theme`) and `horizon` (pointing to Shopify's Horizon repository).

### Update Routine

1. **Fetch the latest changes from Horizon:**

```bash
git fetch horizon
```

2. **Check what's changed (optional but recommended):**

```bash
git log horizon/main..HEAD --oneline  # See our commits not in Horizon
git log HEAD..horizon/main --oneline  # See Horizon commits we don't have
```

3. **Merge Horizon's main branch into your local main:**

```bash
git checkout main
git pull origin main  # Ensure you're up to date
git merge horizon/main
```

4. **Resolve merge conflicts manually:**

**Crucial Note:** Handle merge conflicts carefully:
- **Accept Horizon's changes for:** Structural updates, new features, bug fixes
- **Keep our customizations for:** Custom sections, snippets, and modifications
- **Review carefully:** Changes to files we've customized

Common conflict areas:
- `sections/` - We may have custom sections; keep both when possible
- `snippets/` - Our custom snippets should be preserved
- `assets/` - Merge carefully, preserving our customizations

5. **Test thoroughly:**

After resolving conflicts, test the theme locally:

```bash
shopify theme dev
```

6. **Commit and push:**

```bash
git add .
git commit -m "Merge Horizon updates from [date or version]"
git push origin main
```

### Best Practices for Core Maintainers

- **Review changes before merging:** Use `git log` and `git diff` to understand what's changing
- **Test in development:** Always test merged changes before pushing to main
- **Document customizations:** Keep `theme-customizations.md` updated when making changes
- **Preserve backwards compatibility:** When adding features, ensure existing store themes continue to work
- **Avoid template changes:** Don't modify `templates/` in the Parent; these belong to Child repos

---

## Section B: Store Deployment (Updating Children)

**Target Audience:** Developers managing specific storefronts (e.g., Otis, Creatures of Leisure)

**Goal:** Pull updates from `tlc_shopify_theme` into a specific store theme while protecting store-specific files.

### One-Time Setup

1. **Add the Parent repository as upstream:**

From inside your Child repo (e.g., `otis`):

```bash
git remote add upstream [Parent_Repo_URL]
```

Example:

```bash
git remote add upstream git@github.com:the-leisure-collective/tlc_shopify_theme.git
```

2. **Configure the merge driver:**

Enable Git's `merge=ours` driver globally (one-time setup per machine):

```bash
git config --global merge.ours.driver true
```

3. **Create `.gitattributes` file:**

Create a `.gitattributes` file in the root of your Child repo with the following content:

```gitattributes
# Store-owned files - always keep our version during merges
templates/**                 merge=ours
config/settings_data.json    merge=ours
layout/theme.liquid          merge=ours
```

This ensures that when merging from the Parent, your store-specific templates and settings are never overwritten.

4. **Verify setup:**

```bash
git remote -v
```

You should see:
- `origin` - Your Child repo (e.g., `otis`)
- `upstream` - The Parent repo (`tlc_shopify_theme`)

### Update Routine

1. **Ensure you're on main and up to date:**

```bash
git checkout main
git pull origin main
```

2. **Fetch latest changes from Parent:**

```bash
git fetch upstream
```

3. **Create an update branch (recommended):**

```bash
git checkout -b update-parent-YYYY-MM-DD
```

Replace `YYYY-MM-DD` with today's date (e.g., `update-parent-2025-01-15`).

4. **Merge Parent's main into your update branch:**

```bash
git merge upstream/main
```

5. **Handle any conflicts:**

The `.gitattributes` file will automatically resolve conflicts for `templates/**` and `config/settings_data.json` by keeping your version. For other conflicts:

- **Core files (assets/, snippets/, sections/):** Generally accept Parent's version
- **Custom store files:** Keep your version
- **Uncertain conflicts:** Review carefully and test

6. **Test locally:**

```bash
shopify theme dev
```

Test key pages and functionality to ensure nothing broke.

7. **Push and create a Pull Request:**

```bash
git push origin update-parent-YYYY-MM-DD
```

Create a PR in your Child repo, review the changes, then merge to `main`.

8. **Deploy to Shopify:**

After merging to `main`, Shopify's GitHub integration will automatically sync the updated theme to your store.

### Best Practices for Store Developers

- **Always use update branches:** Never merge directly to `main`; use a branch and PR for review
- **Test before merging:** Use `shopify theme dev` to test locally before creating PRs
- **Review changes:** Understand what's being updated from the Parent
- **Keep `.gitattributes` in place:** Never remove or modify the `.gitattributes` file
- **Document store-specific changes:** If you make customizations, document them in your store repo

---

## Troubleshooting

### Why didn't my new section appear after updating?

**Problem:** You pulled updates from the Parent, but a new section isn't showing up in your store.

**Possible Causes:**
1. The section exists in `sections/` but isn't added to any templates
2. The section requires new settings in `config/settings_data.json`
3. The section has dependencies (snippets/assets) that weren't included

**Solution:**
- Check if the section file exists: `ls sections/your-section-name.liquid`
- Review the section's schema to see if it needs settings configured
- Check if the section requires specific snippets or assets
- Manually add the section to a template if needed

### Merge conflicts in files that should be protected

**Problem:** You're getting merge conflicts in `templates/**`, `config/settings_data.json`, or `layout/theme.liquid` even though you have `.gitattributes` set up.

**Possible Causes:**
1. The `merge=ours` driver isn't configured
2. The `.gitattributes` file isn't in the repo root
3. The file paths in `.gitattributes` don't match exactly

**Solution:**
```bash
# Verify merge driver is configured
git config --global merge.ours.driver

# Should output: true

# If not set, configure it:
git config --global merge.ours.driver true

# Verify .gitattributes exists and is correct
cat .gitattributes

# Should include:
# templates/**                 merge=ours
# config/settings_data.json    merge=ours
# layout/theme.liquid          merge=ours

# If missing, create it with the correct paths
```

### Parent updates overwrote my store settings or layout

**Problem:** After merging from Parent, your `config/settings_data.json` or `layout/theme.liquid` was reset.

**Possible Causes:**
1. `.gitattributes` file is missing or incorrect
2. `merge=ours` driver isn't configured
3. The file was manually resolved incorrectly during a conflict

**Solution:**
1. Restore from git history:
```bash
# For settings_data.json
git log --oneline config/settings_data.json
git checkout [commit-hash] -- config/settings_data.json

# For layout/theme.liquid
git log --oneline layout/theme.liquid
git checkout [commit-hash] -- layout/theme.liquid
```

2. Fix `.gitattributes` and merge driver (see above)

3. Re-apply your settings manually if needed

### Can't fetch from upstream

**Problem:** `git fetch upstream` fails with authentication or URL errors.

**Possible Causes:**
1. Remote URL is incorrect
2. SSH keys aren't set up
3. Repository URL changed

**Solution:**
```bash
# Check current remote URL
git remote get-url upstream

# Update if needed (use SSH for consistency)
git remote set-url upstream git@github.com:the-leisure-collective/tlc_shopify_theme.git

# Test connection
git fetch upstream
```

### Merge conflicts in core files (assets/, snippets/, sections/)

**Problem:** You're getting conflicts in files that should come from the Parent.

**Solution:**
- **Generally accept Parent's version** for core files
- Review the conflict to understand what changed
- If you have store-specific customizations in core files, consider:
  - Moving customizations to store-specific files
  - Documenting why you need to keep your version
  - Discussing with Core maintainers if the customization should be in the Parent

### Theme Check fails after update

**Problem:** Running `shopify theme check` shows errors after updating.

**Possible Causes:**
1. New Theme Check rules in updated Parent
2. Deprecated Liquid syntax
3. Missing required files

**Solution:**
```bash
# Run theme check to see specific errors
shopify theme check

# Fix errors one by one
# Common fixes:
# - Update deprecated Liquid syntax
# - Add missing required attributes
# - Fix schema issues in sections
```

---

## Developer tools

There are a number of really useful tools that the Shopify Themes team uses during development. Horizon is already set up to work with these tools.

### Shopify CLI

[Shopify CLI](https://shopify.dev/docs/storefronts/themes/tools/cli) helps you build Shopify themes faster and is used to automate and enhance your local development workflow. It comes bundled with a suite of commands for developing Shopify themes—everything from working with themes on a Shopify store (e.g. creating, publishing, deleting themes) or launching a development server for local theme development.

You can follow this [quick start guide for theme developers](https://shopify.dev/docs/themes/tools/cli) to get started.

### Theme Check

We recommend using [Theme Check](https://github.com/shopify/theme-check) as a way to validate and lint your Shopify themes.

We've added Theme Check to Horizon's [list of VS Code extensions](/.vscode/extensions.json) so if you're using Visual Studio Code as your code editor of choice, you'll be prompted to install the [Theme Check VS Code](https://marketplace.visualstudio.com/items?itemName=Shopify.theme-check-vscode) extension upon opening VS Code after you've forked and cloned Horizon.

You can also run it from a terminal with the following Shopify CLI command:

```bash
shopify theme check
```

You can follow the [theme check documentation](https://shopify.dev/docs/storefronts/themes/tools/theme-check) for more details.

### Continuous Integration

Horizon uses [GitHub Actions](https://github.com/features/actions) to maintain the quality of the theme. [This is a starting point](https://github.com/Shopify/horizon-private/blob/main/.github/workflows/ci.yml) and what we suggest to use in order to ensure you're building better themes. Feel free to build off of it!

#### Shopify/theme-check-action

Horizon runs [Theme Check](#theme-check) on every commit via [Shopify/theme-check-action](https://github.com/Shopify/theme-check-action).

## Contributing

We are not accepting contributions to Horizon at this time.

## License

Copyright (c) 2025-present Shopify Inc. See [LICENSE](/LICENSE.md) for further details.
