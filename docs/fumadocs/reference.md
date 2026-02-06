---
title: Fumadocs Quick Reference
description: Essential concepts and syntax
---

import { BookOpen, FileText, Layers, Zap } from 'lucide-react';

# Fumadocs Quick Reference

Quick reference for Fumadocs concepts and syntax.

<Cards>
  <Card icon={<FileText />} href="#frontmatter" title="Frontmatter">
    Page metadata and configuration
  </Card>
  <Card icon={<Layers />} href="#folder-structure" title="Folder Structure">
    Organizing content with meta.json
  </Card>
  <Card icon={<BookOpen />} href="#mdx-components" title="MDX Components">
    Cards, Callouts, Codeblocks, Tabs
  </Card>
  <Card icon={<Zap />} href="#sidebar" title="Sidebar & Tabs">
    Navigation configuration
  </Card>
</Cards>

## Frontmatter

```yaml
---
title: Page Title
description: Page description
icon: IconName
---
```

## Folder Structure

### Basic Rules

| File Path | Generated Slug |
|-----------|----------------|
| `./guide/page.mdx` | `/guide/page` |
| `./guide/index.mdx` | `/guide` |

### meta.json

```json
{
  "title": "Folder Name",
  "icon": "IconName",
  "pages": ["index", "page1", "page2"],
  "defaultOpen": true,
  "collapsible": true,
  "root": false
}
```

**Special values:**

- `"root": true` - Makes folder a sidebar tab
- `"pages": ["..."]` - Include all remaining pages
- `"pages": ["!file"]` - Exclude specific file

## MDX Components

### Callout

```mdx
<Callout type="info">Default info</Callout>
<Callout type="warning">Warning message</Callout>
<Callout type="error">Error message</Callout>
<Callout type="success">Success message</Callout>
<Callout type="idea">Idea / tip</Callout>
```

### Card

```mdx
<Cards>
  <Card href="/link" title="Title">Description</Card>
  <Card icon={<Icon />} title="No Link">Content</Card>
</Cards>
```

### Codeblock

````
```js title="Title" lineNumbers
console.log('Hello');
```
````

### Tab Group

````
```ts tab="Tab 1"
console.log('A');
```

```ts tab="Tab 2"
console.log('B');
```
````

### Twoslash

````
```ts twoslash
const a: string = 'hello';
//    ^?
```
````

## Heading Control

```md
# Regular heading
# Custom Anchor [#custom-id]
# Hide from TOC [!toc]
# TOC Only [toc]
# Combined [toc] [#custom-id]
```

## Pages Property Syntax

```json
{
  "pages": [
    "page",
    "---Separator---",
    "...folder",
    "...",
    "!exclude",
    "[Link](https://example.com)",
    "external:[External](https://example.com)"
  ]
}
```

## i18n Files

```
page.mdx           // Default locale
page.fr.mdx        // French
page.es.mdx        // Spanish

meta.json          // Default
meta.fr.json       // French
```

## Sidebar Tabs

### Using meta.json

```json
{
  "title": "Framework",
  "root": true
}
```

### Using Layout

```tsx
<DocsLayout
  sidebar={{
    tabs: [
      {
        title: 'Components',
        url: '/docs/components'
      }
    ]
  }}
/>
```

## Common Patterns

### Auto-generate cards from folder

```tsx
import { getPageTreePeers } from 'fumadocs-core/page-tree';

<Cards>
  {getPageTreePeers(source.getPageTree(), '/docs/page').map(peer => (
    <Card key={peer.url} title={peer.name} href={peer.url}>
      {peer.description}
    </Card>
  ))}
</Cards>
```

### Include another file

```mdx
<include>./other-file.mdx</include>
```

### NPM command with dropdown

````
```npm
npm i package
```
````

## Layout CSS Variables

```css
--fd-sidebar-width: 250px
--fd-toc-width: 200px
--fd-layout-width: 97rem
--fd-banner-height: 0px
--fd-header-height: 60px
```
