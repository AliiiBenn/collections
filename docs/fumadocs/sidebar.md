---
title: Sidebar Configuration
description: Customizing sidebar navigation
---

# Sidebar Configuration

## Basics

Sidebar items are rendered from page tree passed to `<DocsLayout />`:

```tsx
import { DocsLayout } from 'fumadocs-ui/layouts/docs';
import { source } from '@/lib/source';

export default function Layout({ children }) {
  return (
    <DocsLayout tree={source.getPageTree()}>
      {children}
    </DocsLayout>
  );
}
```

## Sidebar Tabs (Root Folders)

Root folders render as tabs/dropdowns. Only content of opened tab is visible.

### Using meta.json

Mark folder as root:

```json
// content/docs/my-folder/meta.json
{
  "title": "Name of Folder",
  "description": "Optional description",
  "root": true
}
```

### Using Layout Props

Specify tabs explicitly:

```tsx
<DocsLayout
  sidebar={{
    tabs: [
      {
        title: 'Components',
        description: 'Hello World!',
        url: '/docs/components', // Active for this URL and sub-routes
        // urls: new Set(['/docs/test', '/docs/components']), // Optional
      },
    ],
  }}
/>
```

### Disable Tabs

```tsx
<DocsLayout sidebar={{ tabs: false }} />
```

## Banner

Add banner to sidebar:

```tsx
<DocsLayout
  sidebar={{
    banner: <div>Hello World</div>
  }}
/>
```

## Tab Decoration

Customize tab icons/styles:

```tsx
<DocsLayout
  sidebar={{
    tabs: {
      transform: (option, node) => ({
        ...option,
        icon: <MyIcon />,
      }),
    },
  }}
/>
```

## Custom Components

Replace sidebar rendering components:

```tsx
import { SidebarSeparator } from './layout.client';

<DocsLayout
  sidebar={{
    enabled: true,
    components: {
      Separator: SidebarSeparator,
    },
  }}
/>
```

## Prefetching

Fumadocs uses framework's `<Link />` with default prefetch behavior.

**Disable to reduce serverless usage:**

```tsx
<DocsLayout sidebar={{ prefetch: false }} />
```

## Layout System

Fumadocs uses CSS Grid for layout:

```css
#nd-docs-layout {
  grid-template:
    'sidebar header toc'
    'sidebar toc-popover toc'
    'sidebar main toc' 1fr / minmax(var(--fd-sidebar-col), 1fr) minmax(0, var(--fd-page-col))
    minmax(min-content, 1fr);

  --fd-docs-row-1: var(--fd-banner-height, 0px);
  --fd-docs-row-2: calc(var(--fd-docs-row-1) + var(--fd-header-height));
  --fd-docs-row-3: calc(var(--fd-docs-row-2) + var(--fd-toc-popover-height));

  --fd-sidebar-col: var(--fd-sidebar-width);
  --fd-page-col: calc(
    var(--fd-layout-width, 97rem) - var(--fd-sidebar-width) - var(--fd-toc-width)
  );

  --fd-sidebar-width: 0px;
  --fd-toc-width: 0px;
  --fd-header-height: 0px;
  --fd-toc-popover-height: 0px;
}
```

**Key CSS Variables:**

- `--fd-docs-row-*`: Top offset for each row (for sticky positioning)
- `--fd-*-width`: Dynamic widths updated by state
- `--fd-*-col`: Calculated column widths

This system is:
- **Composable**: Components manage position in place
- **Flexible**: No fixed heights/values
- **Cohesive**: Components respond to each other
- **Predictable**: Centralized layout properties
- **Compatible**: Uses baseline CSS features
