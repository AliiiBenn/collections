---
title: MDX Syntax Guide
description: Writing content with MDX components
---

# MDX Syntax Guide

Fumadocs uses MDX (Markdown + JSX) with powerful extensions.

## Frontmatter

YAML-based frontmatter at the top of files:

```yaml
---
title: This is a document
description: Document description
---

# Content here
```

**Note**: `title` becomes the page h1. Don't use `# Heading` unless you have custom logic.

## Auto Links

- **Internal links**: Use framework's `<Link />` for prefetching
- **External links**: Get `rel="noreferrer noopener" target="_blank"`

```md
[My Link](https://example.com)
https://example.com
```

## Cards

Useful for linking to other pages:

```mdx
import { HomeIcon } from 'lucide-react';

<Cards>
  <Card href="https://example.com" title="Link Title">
    Description here
  </Card>
  <Card title="No href">Content here</Card>
  <Card icon={<HomeIcon />} href="/" title="With Icon">
    Icon included
  </Card>
</Cards>
```

### Auto-Generate from Peers

Show other pages in the same folder:

```tsx
import { getPageTreePeers } from 'fumadocs-core/page-tree';
import { source } from '@/lib/source';

<Cards>
  {getPageTreePeers(source.getPageTree(), '/docs/my-page').map((peer) => (
    <Card key={peer.url} title={peer.name} href={peer.url}>
      {peer.description}
    </Card>
  ))}
</Cards>
```

## Callouts

Add tips, warnings, and notes:

```mdx
<Callout>Hello World</Callout>
<Callout title="Title">Hello World</Callout>
<Callout type="error" title="Error">
  Error message
</Callout>
<Callout type="idea" title="Idea">
  Great idea
</Callout>
```

**Types:**

- `info` (default)
- `warn` / `warning`
- `error`
- `success`
- `idea`

## Headings

Anchors automatically applied to each heading.

### Custom Anchor

```md
# heading [#my-heading-id]
```

### TOC Control

```md
# Heading [!toc]     // Hidden from TOC
# Heading [toc]      // Only visible in TOC
```

### Chain Settings

```md
# heading [toc] [#my-id]
```

## Codeblocks

### Basic

````
```js
console.log('Hello World');
```
````

### With Title

````
```js title="My Title"
console.log('Hello World');
```
````

### Line Numbers

````
```js lineNumbers
const a = 'Hello World';
console.log(a);
```
````

### Custom Start

````
```js lineNumbers=4
function main() {
  console.log('starts from 4');
  return 0;
}
```
````

### Shiki Transformers

Highlight specific lines:

````
```tsx
// highlight a line
<div>Hello World</div> // [!code highlight]

// highlight a word
// [!code word:Fumadocs]
<div>Fumadocs</div>

// diff styles
console.log('hewwo'); // [!code --]
console.log('hello'); // [!code ++]

// focus
return new ResizeObserver(() => {}) // [!code focus]
```
````

## Tab Groups

````
```ts tab="Tab 1"
console.log('A');
```

```ts tab="Tab 2"
console.log('B');
```
````

### Enable MDX in Tabs

Configure remark plugin:

```ts
import { defineConfig } from 'fumadocs-mdx/config';

export default defineConfig({
  mdxOptions: {
    remarkCodeTabOptions: {
      parseMdx: true,
    },
  },
});
```

Then use JSX in tab values:

````
```ts tab="<Component /> Tab 1"
console.log('A');
```
````

## Include

Reference another file (MDX/Markdown):

```mdx
<include>./another.mdx</include>
```

Path is relative to the current MDX file.

## NPM Commands

Generate installation commands for different package managers:

````
```npm
npm i next -D
```
````

Renders as: `npm install next -D` with dropdown for yarn/pnpm/bun.

## Other Components

Built-in components available:

- **Tabs**: `import { Tabs, Tab } from 'fumadocs-ui'`
- **Accordions**: `import { Accordion, AccordionItem } from 'fumadocs-ui'`
- **Zoomable Image**: `import { ZoomableImage } from 'fumadocs-ui'`
