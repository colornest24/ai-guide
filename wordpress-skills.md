# OpenAI Codex: WordPress Agent Skills & Live Site MCP Guide

This guide covers installing official WordPress coding skills (`SKILL.md`), connecting OpenAI Codex to a live WordPress site via the Model Context Protocol (MCP), and building custom `SKILL.md` templates for your WordPress workflows.

---

## Part 1: Installing Official WordPress Agent Skills

WordPress maintains an official [WordPress Agent Skills repository](https://github.com/WordPress/agent-skills) containing 20+ specialized `SKILL.md` modules for block development, themes, WP-CLI, performance profiling, and WordPress Playground.

### Prerequisites
* **OpenAI Codex:** Codex CLI, IDE extension, or ChatGPT Desktop App installed.
* **Node.js:** Version **20.18+** (required for `npx` CLI installation).
* **Git:** Required if cloning the repository manually.

### Method 1: Quick Install via CLI (Recommended)
Run the `skills` CLI in your terminal to automatically download and register skills for Codex:

```bash
# Install a specific WordPress skill
npx skills add wordpress/agent-skills --skill wp-playground

# Install additional skills as needed
npx skills add wordpress/agent-skills --skill wp-block-development
npx skills add wordpress/agent-skills --skill wp-plugin-development
```

#### Alternative: Project-Scoped Install via Skills Hub
To lock a skill inside a specific project folder (tracked via `.skills.json`):

```bash
npx @skills-hub-ai/cli install <skill-slug>
```

### Method 2: Manual Global Installation
To make all official WordPress skills available globally across every project:

1. **Create the global Codex skills directory** (if it does not exist):
   ```bash
   mkdir -p ~/.codex/skills
   ```
   *(On Windows PowerShell, use `New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.codex\skills"`.)*

2. **Clone the repository and copy the skill folders:**
   ```bash
   git clone https://github.com/WordPress/agent-skills.git /tmp/wp-skills
   cp -r /tmp/wp-skills/skills/* ~/.codex/skills/
   rm -rf /tmp/wp-skills
   ```
   *(Note: If you use a custom `CODEX_HOME` environment variable, copy the folders into `$CODEX_HOME/skills/` instead.)*

3. **Restart Codex** (CLI, IDE extension, or desktop app) to load the skills.

---

## Part 2: Step-by-Step MCP Server Configuration for a Live WordPress Site

Local skills help Codex write code, whereas an **MCP Server** connects Codex directly to a live or staging WordPress site to create posts, edit pages, manage media, and inspect site data.

OpenAI Codex reads MCP server definitions from **`~/.codex/config.toml`** (macOS/Linux) or **`%USERPROFILE%\.codex\config.toml`** (Windows). All Codex clients (CLI, IDE extension, and Desktop App) share this file.

### Step 1: Prepare Your WordPress Site
Choose one of the following WordPress-side setups:

* **Option A: Official WordPress MCP Adapter Plugin (Recommended for full native Abilities)**
  1. Install and activate the [WordPress MCP Adapter](https://github.com/WordPress/mcp-adapter) plugin on your WordPress site.
  2. In your WordPress Admin dashboard, go to **Users -> Profile**.
  3. Scroll to **Application Passwords**, enter a name (e.g., `Codex MCP`), and click **Add New Application Password**.
  4. Copy the generated password immediately (spaces are optional when pasting into environment variables).

* **Option B: REST API Bridge (No WordPress Plugin Required)**
  * If you cannot install plugins on the target site, you only need a standard WordPress **Application Password** (created under **Users -> Profile**). You will connect via a local stdio wrapper such as `@leonardobora/mcp-wordpress` or `mcp-wordpress`.

### Step 2: Register the MCP Server in Codex

#### Using Option A (Official WordPress MCP Adapter via CLI)
Run the `codex mcp add` command in your terminal, replacing the URL, username, and password with your details:

```bash
codex mcp add wordpress-live \
  --env WP_API_URL=https://your-site.com/wp-json/mcp/mcp-adapter-default-server \
  --env WP_API_USERNAME=your-wp-username \
  --env WP_API_PASSWORD="xxxx xxxx xxxx xxxx xxxx xxxx" \
  -- npx -y @automattic/mcp-wordpress-remote@latest
```

#### Manual Configuration in `~/.codex/config.toml` (Recommended for Security)
To avoid storing your WordPress Application Password in plain text inside `config.toml`, pass it via a shell environment variable using `env_vars`:

1. **Export your password in your shell profile** (`~/.zshrc`, `~/.bashrc`, or Windows Environment Variables):
   ```bash
   export WP_API_PASSWORD="xxxx xxxx xxxx xxxx xxxx xxxx"
   ```

2. **Add the server block to `~/.codex/config.toml`:**
   ```toml
   [mcp_servers.wordpress-live]
   command = "npx"
   args = ["-y", "@automattic/mcp-wordpress-remote@latest"]
   env_vars = ["WP_API_PASSWORD"]

   [mcp_servers.wordpress-live.env]
   WP_API_URL = "https://your-site.com/wp-json/mcp/mcp-adapter-default-server"
   WP_API_USERNAME = "your-wp-username"
   ```

#### Alternative `config.toml` Block for Option B (No-Plugin REST API Server)
If you did not install the MCP Adapter plugin on WordPress, configure the REST API wrapper instead:

```toml
[mcp_servers.wordpress-rest]
command = "npx"
args = ["-y", "@leonardobora/mcp-wordpress"]
env_vars = ["WP_APP_PASSWORD"]

[mcp_servers.wordpress-rest.env]
WP_SITE_URL = "https://your-site.com"
WP_USERNAME = "your-wp-username"
```

### Step 3: Verify and Test the Connection
1. **Check registered servers in terminal:**
   ```bash
   codex mcp list
   ```
2. **Start a new Codex session** (`codex` in terminal, or restart the Desktop App/IDE extension).
3. Inside the Codex prompt, type `/mcp` to verify that `wordpress-live` is connected and inspect its available tools.
4. **Run a safe read-only test prompt:**
   > *"List the 5 most recent draft posts on my WordPress site using the wordpress-live MCP server."*

*(Tip: To restrict a WordPress MCP server to a single project instead of every Codex session, place the `[mcp_servers.*]` block inside `<project-root>/.codex/config.toml` instead of your global config file.)*

---

## Part 3: Custom `SKILL.md` Template Examples

Codex discovers custom skills stored in `~/.codex/skills/<skill-name>/SKILL.md` (global) or `<project-root>/.codex/skills/<skill-name>/SKILL.md` (project-specific). Every `SKILL.md` file requires YAML frontmatter (`name` and `description`) followed by clear Markdown instructions.

### Template 1: Content Publishing & Editorial Standards (`wp-editorial-workflow`)
Save as `~/.codex/skills/wp-editorial-workflow/SKILL.md` to enforce brand voice, SEO metadata, and safety rules whenever Codex publishes or updates posts via MCP.

```markdown
---
name: wp-editorial-workflow
description: Enforces editorial formatting, SEO metadata checks, and safe draft-first publishing rules when managing live WordPress content via MCP.
---

# WordPress Editorial & MCP Publishing Workflow

Use this skill whenever creating, editing, or auditing posts and pages on a live WordPress site.

## Safety & Publishing Rules
1. **Draft First:** Always create or update content with `status: "draft"` unless the user explicitly instructs you to publish immediately.
2. **No Destructive Deletes:** Never delete existing posts, pages, or media attachments without asking for explicit confirmation first.
3. **Slug Preservation:** Never modify the `slug` of an already published post to prevent broken URLs and SEO loss.

## Content Structure Standards
* **Headings:** Use a single `H1` (handled by post title) and structure body sections logically with `H2` and `H3` blocks.
* **Gutenberg Blocks:** Format content using valid WordPress block markup (`<!-- wp:paragraph -->`, `<!-- wp:heading -->`, `<!-- wp:list -->`) rather than raw unformatted HTML.
* **Internal Links:** Check existing published posts via MCP and include 2-3 relevant internal links where natural.

## Pre-Submission Checklist
Before finishing any content task, verify and report:
- [ ] Post saved as `draft` with a clean, lowercase hyphenated slug.
- [ ] Excerpt populated (140-160 characters).
- [ ] Appropriate category and tags assigned (do not create duplicate tags).
- [ ] Direct preview URL provided to the user.
```

### Template 2: WooCommerce Custom Development (`wp-woocommerce-dev`)
Save as `<project-root>/.codex/skills/wp-woocommerce-dev/SKILL.md` for custom WooCommerce plugins, checkout hooks, and HPOS compatibility.

```markdown
---
name: wp-woocommerce-dev
description: Guidelines for developing WooCommerce extensions, hooks, and custom queries compatible with High-Performance Order Storage (HPOS).
---

# WooCommerce Development Standards

Follow these rules when writing or modifying WooCommerce code in this repository.

## High-Performance Order Storage (HPOS)
* **Never query `wp_posts` or `wp_postmeta` directly** for order data (`shop_order`).
* Always use WooCommerce CRUD classes (`wc_get_order()`, `WC_Order`, `wc_get_orders()`) to read and update order properties and metadata.
* Declare HPOS compatibility in the main plugin file using `\Automattic\WooCommerce\Utilities\FeaturesUtil::declare_compatibility( 'custom_order_tables', __FILE__, true )`.

## Hooks & Security
* Prefix all custom functions, hooks, and option names with the plugin slug.
* Validate and sanitize all `$_POST` and `$_GET` inputs (`sanitize_text_field()`, `absint()`) and verify nonces (`wp_verify_nonce()`) on form submissions and AJAX handlers.
* Escape all output using `esc_html()`, `esc_attr()`, `esc_url()`, or `wp_kses_post()`.

## Verification
* Check all modified PHP files for syntax errors (`php -l`).
* Ensure any new templates override `woocommerce/templates/` cleanly with version docblocks intact.
```

### Template 3: WP-CLI Staging & Deployment Checklist (`wp-staging-deploy`)
Save as `~/.codex/skills/wp-staging-deploy/SKILL.md` to guide safe database search-replace and cache flushing operations.

```markdown
---
name: wp-staging-deploy
description: Safe WP-CLI workflow for syncing staging environments, running search-replace, and flushing caches without corrupting serialized data.
---

# Safe WP-CLI & Staging Operations

Use this skill when running WP-CLI commands for site migrations, database updates, or troubleshooting.

## Mandatory Pre-Flight Step
* Always run `wp db export backup-$(date +%F-%H%M).sql` before executing any database write, plugin update, or search-replace command.

## Search-Replace Rules
* Always run with `--dry-run` first and show the affected table counts to the user.
* Include `--all-tables` and `--precise` so PHP serialized arrays and JSON blocks are updated safely without corruption.
* Example:
  ```bash
  wp search-replace 'https://staging.example.com' 'https://example.com' --all-tables --precise --dry-run
  ```

## Post-Operation Cleanup
After completing database or plugin changes, run:
1. `wp cache flush`
2. `wp rewrite flush --hard`
3. `wp transient delete --all`
```

### Links:
> https://mcpservers.org/agent-skills/author/wordpress
> https://developer.wordpress.org/playground/handbook/guides/agent-skill-wp-playground/
https://skills-hub.ai/sources/wordpress
