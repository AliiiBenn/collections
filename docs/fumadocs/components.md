---
title: Additional Components
description: Accordion, Files, Steps, and more
---

import { FileText, FolderTree, List, Github } from 'lucide-react';

# Additional Components

More Fumadocs UI components for your documentation.

<Cards>
  <Card icon={<List />} href="#accordion" title="Accordion">
    FAQ and collapsible content
  </Card>
  <Card icon={<FolderTree />} href="#files" title="Files">
    Display file structures
  </Card>
  <Card icon={<FileText />} href="#auto-type-table" title="Auto Type Table">
    Generate TypeScript type tables
  </Card>
  <Card icon={<Github />} href="#github-info" title="GitHub Info">
    Repository information
  </Card>
</Cards>

## Accordion

Based on Radix UI Accordion. Perfect for FAQ sections.

### Basic Usage

```mdx
import { Accordion, Accordions } from 'fumadocs-ui/components/accordion';

<Accordions type="single">
  <Accordion title="My Title">My Content</Accordion>
  <Accordion title="Another Title">More Content</Accordion>
</Accordions>
```

### Linking to Accordion

Specify an `id` to auto-open when navigating with hash:

```mdx
<Accordion title="My Title" id="my-title">
  My Content
</Accordion>
```

Navigating to `#my-title` automatically opens this accordion.

## Files

Display file structures in documentation.

### Basic Usage

```mdx
import { File, Folder, Files } from 'fumadocs-ui/components/files';

<Files>
  <Folder name="app" defaultOpen>
    <File name="layout.tsx" />
    <File name="page.tsx" />
    <File name="global.css" />
  </Folder>
  <Folder name="components">
    <File name="button.tsx" />
    <File name="tabs.tsx" />
  </Folder>
  <File name="package.json" />
</Files>
```

### Remark Plugin

Enable syntax with `remark-mdx-files`:

```ts
import { remarkMdxFiles } from 'fumadocs-core/mdx-plugins/remark-mdx-files';
import { defineConfig } from 'fumadocs-mdx/config';

export default defineConfig({
  mdxOptions: {
    remarkPlugins: [remarkMdxFiles],
  },
});
```

### CodeBlock Syntax

```mdx
```files
project
├── src
│   ├── index.js
│   └── utils
│       └── helper.js
├── package.json
```
````

### Auto-Generate from Glob

```mdx
<auto-files dir="./my-dir" pattern="**/*.{ts,tsx}" />
<auto-files dir="./my-dir" pattern="**/*.{ts,tsx}" defaultOpenAll />
```

## Auto Type Table

Server component only - generates tables from TypeScript definitions.

### Installation

```bash
npm i fumadocs-typescript
```

### Setup

```tsx
import defaultComponents from 'fumadocs-ui/mdx';
import { createGenerator, createFileSystemGeneratorCache } from 'fumadocs-typescript';
import { AutoTypeTable } from 'fumadocs-typescript/ui';

const generator = createGenerator({
  cache: createFileSystemGeneratorCache('.next/fumadocs-typescript'),
});

export function getMDXComponents(components) {
  return {
    ...defaultComponents,
    AutoTypeTable: (props) => <AutoTypeTable {...props} generator={generator} />,
    ...components,
  };
}
```

### From File

```tsx
// path/to/file.ts
export interface MyInterface {
  name: string;
}
```

```mdx
<AutoTypeTable path="./path/to/file.ts" name="MyInterface" />
```

### From Type

```mdx
<AutoTypeTable type="{ hello: string }" />
```

With path context:

```mdx
<AutoTypeTable path="file.ts" type="A & { world: string }" />
```

### Multi-line Types

```mdx
<AutoTypeTable
  path="file.ts"
  name="B"
  type={`
import { ReactNode } from "react"
export type B = ReactNode | { world: string }
`}
/>
```

### Compiler Options

```ts
const generator = createGenerator({
  tsconfigPath: './tsconfig.json',
  basePath: './',
  // other options...
});
```

<Callout type="warning">
  Only object types allowed. For functions, wrap them in an object interface.
</Callout>

## GitHub Info

Display GitHub repository information (stars, forks).

### Usage

```tsx
import { GithubInfo } from 'fumadocs-ui/components/github-info';

<GithubInfo
  owner="fuma-nama"
  repo="fumadocs"
  token={process.env.GITHUB_TOKEN} // Optional
/>
```

### Add to Layout

```tsx
import { DocsLayout } from 'fumadocs-ui/layouts/notebook';
import { GithubInfo } from 'fumadocs-ui/components/github-info';

function docsOptions() {
  return {
    tree: source.getPageTree(),
    links: [
      {
        type: 'custom',
        children: <GithubInfo owner="fuma-nama" repo="fumadocs" />,
      },
    ],
  };
}

export default function Layout({ children }) {
  return <DocsLayout {...docsOptions()}>{children}</DocsLayout>;
}
```

## Steps

Numbered step indicators for processes.

### With Components

```mdx
import { Step, Steps } from 'fumadocs-ui/components/steps';

<Steps>
  <Step>
    ### Step 1
    Description here
  </Step>
  <Step>
    ### Step 2
    More content
  </Step>
</Steps>
```

### Without Imports (Tailwind)

```mdx
<div className="fd-steps">
  <div className="fd-step">
    ### Step 1
    Content here
  </div>
  <div className="fd-step">
    ### Step 2
    More content
  </div>
</div>
```

### Arbitrary Variants

Apply step styles only to headings:

```mdx
<div className='fd-steps [&_h3]:fd-step'>
  ### Step 1
  Content
  ### Step 2
  Content
</div>
```

## Code Block

Wrapper for Shiki-highlighted code blocks.

### Basic Setup

```tsx
import { CodeBlock, Pre } from 'fumadocs-ui/components/codeblock';

export function getMDXComponents(components) {
  return {
    ...defaultComponents,
    pre: ({ ref: _ref, ...props }) => (
      <CodeBlock {...props}>
        <Pre>{props.children}</Pre>
      </CodeBlock>
    ),
    ...components,
  };
}
```

### Keep Background

Use Shiki's background color:

```tsx
<CodeBlock keepBackground {...props}>
  <Pre>{props.children}</Pre>
</CodeBlock>
```

### Custom Icon

```tsx
<CodeBlock icon={<MyIcon />} {...props}>
  <Pre>{props.children}</Pre>
</CodeBlock>
```

## Component Props Reference

### Accordion

| Prop | Type |
|------|------|
| title | string |
| id | string |
| children | ReactNode |

### Files / Folder / File

| Component | Prop | Type |
|-----------|------|------|
| Files | - | - |
| Folder | name | string |
| Folder | defaultOpen | boolean |
| File | name | string |

### AutoTypeTable

| Prop | Type |
|------|------|
| path | string (relative to cwd) |
| name | string (export name) |
| type | string (inline type) |
| generator | Generator (required) |

### GithubInfo

| Prop | Type |
|------|------|
| owner | string |
| repo | string |
| token | string (optional) |

### Steps / Step

| Component | Prop | Type |
|-----------|------|------|
| Steps | - | - |
| Step | - | - |

### CodeBlock

| Prop | Type |
|------|------|
| children | ReactNode |
| keepBackground | boolean |
| icon | ReactNode |
