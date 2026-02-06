---
title: Twoslash Integration
description: Type hovering in code blocks
---

# Twoslash Integration

Add type hovering to TypeScript code blocks using Twoslash.

## Installation

```bash
npm install fumadocs-twoslash twoslash
```

## Next.js Setup

Externalize deps in `next.config.mjs`:

```js
const config = {
  reactStrictMode: true,
  serverExternalPackages: ['typescript', 'twoslash'],
};
```

## Configure Transformer

Add to Shiki transformers in `source.config.ts`:

```ts
import { defineConfig } from 'fumadocs-mdx/config';
import { transformerTwoslash } from 'fumadocs-twoslash';
import { rehypeCodeDefaultOptions } from 'fumadocs-core/mdx-plugins';

export default defineConfig({
  mdxOptions: {
    rehypeCodeOptions: {
      themes: {
        light: 'github-light',
        dark: 'github-dark',
      },
      transformers: [
        ...(rehypeCodeDefaultOptions.transformers ?? []),
        transformerTwoslash()
      ],

      // Important: Define common languages
      langs: ['js', 'jsx', 'ts', 'tsx'],
    },
  },
});
```

## Add Styles

Tailwind CSS v4 required:

```css
@import 'fumadocs-twoslash/twoslash.css';
```

## Add MDX Components

In `mdx-components.tsx`:

```tsx
import * as Twoslash from 'fumadocs-twoslash/ui';
import defaultComponents from 'fumadocs-ui/mdx';
import type { MDXComponents } from 'mdx/types';

export function getMDXComponents(components?: MDXComponents): MDXComponents {
  return {
    ...defaultComponents,
    ...Twoslash,
    ...components,
  };
}
```

## Usage

Add `twoslash` meta to code blocks:

````
```ts twoslash
console.log('Hello World');
```
````

### Type Errors

````
```ts twoslash
const a = '123';
a = 132; // Error: Cannot assign to 'a' because it is a constant
```
````

### Hover Types

````
```ts twoslash
const player: Player = { name: 'Hello World' };
```
````

Users can hover over `Player` to see type definition.

## Filesystem Cache (Optional)

Enable caching with `typesCache`:

```ts
import { transformerTwoslash } from 'fumadocs-twoslash';
import { createFileSystemTypesCache } from 'fumadocs-twoslash/cache-fs';

transformerTwoslash({
  typesCache: createFileSystemTypesCache(),
});
```

## Important Notes

1. **Pre-define languages**: Shiki doesn't support lazy loading for Twoslash popups
2. **Common langs**: Define `js`, `jsx`, `ts`, `tsx` at minimum
3. **Performance**: Cache improves rebuild times
