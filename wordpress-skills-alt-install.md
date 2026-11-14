# WordPress Agent Skills Guide for Codex

This guide explains how to install, configure, and use standardized WordPress Agent Skills (`SKILL.md` files) within OpenAI Codex CLI and compatible AI coding environments.

---

## 1. Overview

WordPress Agent Skills provide AI coding agents with structured domain knowledge, coding standards, security baselines, and local testing workflows tailored specifically for WordPress core, plugins, and block themes.

### Key Benefits
* **Automated Project Routing:** Detects whether your repository is a plugin, classic theme, block theme, or full site setup.
* **Security & Standards Enforcement:** Enforces WordPress Coding Standards (WPCS), proper `$wpdb` preparation, capability checks, and nonce verification.
* **Rapid Local Testing:** Integrates with WordPress Playground for instant browser or CLI-based environment spins without heavy Docker or local server configuration.

---

## 2. Installation Methods

Choose the installation method that best fits your development setup.

### Method A: Project-Level Installation via OpenSkills (Recommended)
Run the following commands in the root directory of your WordPress project to install the official WordPress agent skills and sync them with Codex:

```bash
# Install the official WordPress agent skills package
npx openskills install WordPress/agent-skills

# Sync skills to generate local configuration mappings for Codex
npx openskills sync
```

### Method B: Global Configuration (`~/.codex/skills`)
If you work across multiple WordPress repositories and want the skills available globally across all Codex sessions:

```bash
# Create the global Codex skills directory if it does not exist
mkdir -p ~/.codex/skills

# Clone the WordPress agent skills repository into a temporary folder
git clone https://github.com/WordPress/agent-skills.git /tmp/wp-agent-skills

# Copy skill definitions into your Codex home directory
cp -r /tmp/wp-agent-skills/skills/* ~/.codex/skills/

# Clean up temporary files
rm -rf /tmp/wp-agent-skills
```

*(Note: If you use a custom `CODEX_HOME` environment variable, replace `~/.codex/` with `$CODEX_HOME/`.)*

---

## 3. Core WordPress Skills Reference

Once installed, Codex can automatically invoke or be explicitly prompted to use the following core skills:

| Skill Slug | Purpose & Functionality | Best Used When... |
| :--- | :--- | :--- |
| `wordpress-router` | Inspects directory structure (`style.css`, plugin headers, `theme.json`, `composer.json`) to classify the project. | Starting a new Codex session in an unfamiliar repository. |
| `wp-plugin-development` | Applies best practices for hooks, actions, filters, custom post types, settings APIs, and data sanitization. | Scaffolding a new plugin or writing backend PHP logic. |
| `wp-block-development` | Guides `block.json` configuration, React/JSX `edit.js` and `save.js` components, and `@wordpress/scripts` builds. | Building custom Gutenberg blocks or interactive blocks. |
| `wp-theme-development` | Enforces `theme.json` v3 schemas, template parts, patterns, and Full Site Editing (FSE) conventions. | Creating or modifying modern WordPress block themes. |
| `wp-playground` | Launches ephemeral WordPress instances using `@wp-playground/cli` to test code changes immediately. | Running integration checks or verifying plugin activation without manual setup. |
| `wp-abilities-audit` | Audits REST API endpoints, custom routes, permissions callbacks, and exposed capabilities. | Performing security reviews or documenting plugin APIs. |

---

## 4. Writing Custom `SKILL.md` Files

You can create custom project-specific skills (for example, for Elementor widgets, WooCommerce extensions, or agency boilerplate rules) by adding a `SKILL.md` file inside `.codex/skills/<skill-name>/SKILL.md`.

### Example Custom Skill Template

```markdown
---
name: wp-custom-architecture
description: Enforces agency coding standards, custom post type prefixes, and WooCommerce hook conventions.
---

# Custom WordPress Architecture Rules

## Naming & Prefixing
- Prefix all PHP functions, classes, and global variables with `acme_`.
- Use the text domain `acme-core` for all internationalization functions (`__()`, `_e()`, `esc_html__()`).

## Security Checklist
- Always sanitize input using `sanitize_text_field()`, `absint()`, or `wp_kses_post()`.
- Always escape output using `esc_html()`, `esc_attr()`, or `esc_url()`.
- Verify nonces via `wp_verify_nonce()` before processing `$_POST` requests.

## Performance Rules
- Avoid `posts_per_page => -1` in `WP_Query`; always set an explicit upper bound.
- Cache expensive external API responses using the Transients API (`set_transient()`).
```

---

## 5. Effective Prompting Workflow with Codex

To get the best results when working with WordPress skills in Codex CLI:

1. **Initialize Context First:**
   > "Use the `wordpress-router` skill to inspect this repository and summarize its architecture before making changes."
2. **Combine Skills for Feature Work:**
   > "Following `wp-plugin-development` rules, create a custom REST endpoint under `/acme/v1/status` with proper `permission_callback` checks."
3. **Verify Before Committing:**
   > "Run `wp-abilities-audit` on the newly added endpoints and test activation using `wp-playground`."

---

## 6. Troubleshooting

* **Skills Not Detected:** Ensure `AGENTS.md` or your `.codex/skills/` directory contains valid YAML frontmatter (`name` and `description`) at the top of each `SKILL.md` file.
* **Outdated Skill Definitions:** Re-run `npx openskills update` or pull the latest changes from the upstream repository and re-sync.
* **Permission Errors on Sync:** Verify write permissions on `~/.codex/` and your local project root directory.

