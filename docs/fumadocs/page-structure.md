---
title: Page Structure & Organization
description: How Fumadocs organizes pages and generates slugs
---

# Page Structure & Organization

## File System to Slugs

Fumadocs generates page slugs and page tree from your content directory using `loader()`.

### Basic Rules

| File Path | Slugs |
|-----------|-------|
| `./dir/page.mdx` | `['dir', 'page']` |
| `./dir/index.mdx` | `['dir']` |

### Frontmatter

Customize page info from frontmatter:

```yaml
---
title: My Page
description: Best document ever
icon: HomeIcon
---
```

**Properties:**

- `title`: Page title
- `description`: Page description
- `icon`: Icon name (see Icons section)

## Folders

Organize multiple pages with folders. Create a `meta.json` to customize:

```json
{
  "title": "Display Name",
  "icon": "MyIcon",
  "pages": ["index", "getting-started"],
  "defaultOpen": true,
  "collapsible": true
}
```

**Properties:**

- `title`: Display name
- `icon`: Icon name
- `defaultOpen`: Open folder by default
- `collapsible`: Allow collapsing (default: true)
- `pages`: Control item order

### Folder Groups

Wrap folder name in parentheses to avoid impacting slugs:

| Path | Slugs |
|------|-------|
| `./(group-name)/page.mdx` | `['page']` |

### Root Folders

Mark as root to show only that folder's content:

```json
{
  "title": "Framework",
  "root": true
}
```

Root folders render as **Sidebar Tabs** - only items in the opened tab are visible.

## Pages Property

Control folder item order with `pages` in `meta.json`:

```json
{
  "pages": ["index", "getting-started"]
}
```

When specified, items NOT listed are excluded.

### Syntax Types

| Type | Syntax | Description |
|------|--------|-------------|
| Path | `./path/to/page` | Path to page or folder |
| Separator | `---Label---` | Section separator |
| Link | `[Text](url)` | Internal/external link |
| External Link | `external:[Text](url)` | Mark as external |
| Rest | `...` | Include remaining pages |
| Reversed Rest | `z...a` | Reversed rest |
| Extract | `...folder` | Extract items from folder |
| Except | `!item` | Exclude item |

### Example

```json
{
  "pages": [
    "components",
    "---My Separator---",
    "...folder",
    "...",
    "!file",
    "!otherFolder",
    "[Vercel](https://vercel.com)",
    "[Triangle][Vercel](https://vercel.com)"
  ]
}
```

## i18n Routing

Add locale suffix to files:

```
get-started.mdx
get-started.cn.mdx

meta.json
meta.cn.json
```

For default locale, the locale code is optional.

**Parser types:**

```ts
// Default
parser: 'dot'

// Or directory-based
parser: 'dir'
```
