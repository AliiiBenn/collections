# Plugin System

## Overview

The plugin system allows you to extend and modify collections, add new field types, create new collections, and inject behavior at any point in the system. Plugins are composable, type-safe, and can be shared as packages.

## What Plugins Can Do

### 1. Add Fields to Collections

Plugins can automatically add fields to existing collections:

```typescript
// plugins/timestamps.ts
import { field, timestamp } from '@deessejs/collections'

export const timestampsPlugin = () => ({
  name: 'timestamps',

  // Add fields to all collections
  fields: {
    createdAt: field({
      type: timestamp(),
      default: () => new Date()
    }),
    updatedAt: field({
      type: timestamp()
    })
  }
})

// Usage
collection('posts', {
  plugins: [timestampsPlugin()]
  // Automatically gains: createdAt, updatedAt
})
```

### 2. Add New Field Types

Plugins can register custom field types:

```typescript
// plugins/custom-fields.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText } from 'drizzle-orm/pg-core'

// Custom field types
export const slug = fieldType({
  schema: z.string().regex(/^[a-z0-9-]+$/),
  database: pgText()
})

export const phone = fieldType({
  schema: z.string().regex(/^\+?[\d\s-()]+$/),
  database: pgText()
})

export const money = fieldType({
  schema: z.number().positive(),
  database: pgNumeric({ precision: 10, scale: 2 })
})

// Plugin that registers field types
export const customFieldsPlugin = () => ({
  name: 'custom-fields',

  // Register new field types
  fieldTypes: {
    slug,
    phone,
    money
  }
})

// Usage in config
defineConfig({
  database: { ... },
  plugins: [customFieldsPlugin()],
  collections: [
    collection('posts', {
      fields: {
        // Now available everywhere!
        slug: slug(),
        price: money()
      }
    })
  ]
})
```

### 3. Add New Collections

Plugins can create entire collections:

```typescript
// plugins/audit-log.ts
import { collection, field } from '@deessejs/collections'
import { text, timestamp, enumField } from '@deessejs/collections/fields'

export const auditLogPlugin = () => ({
  name: 'audit-log',

  // Add new collections
  collections: [
    collection('auditLogs', {
      label: 'Audit Logs',
      fields: {
        action: field({
          type: enumField(['create', 'update', 'delete', 'read'])
        }),
        entity: field({
          type: text()
        }),
        entityId: field({
          type: number()
        }),
        userId: field({
          type: relation('users')
        }),
        changes: field({
          type: json()
        }),
        timestamp: field({
          type: timestamp(),
          default: () => new Date()
        })
      }
    })
  ],

  // Add hooks to track all operations
  hooks: {
    afterCreate: [async ({ result, context }) => {
      await context.collections.auditLogs.create({
        data: {
          action: 'create',
          entity: context.collection.slug,
          entityId: result.id,
          userId: context.currentUser?.id,
          changes: result
        }
      })
    }]
  }
})

// Usage
defineConfig({
  database: { ... },
  plugins: [auditLogPlugin()],
  collections: [
    // auditLogs collection automatically added!
    collection('posts', { ... }),
    collection('users', { ... })
  ]
})
```

## Plugin Structure

### Basic Plugin Shape

```typescript
export const myPlugin = (options?: {
  // Plugin configuration
}) => ({
  // Plugin identifier
  name: 'my-plugin',

  // Add collections
  collections: [],

  // Add field types
  fieldTypes: {},

  // Add fields to existing collections
  fields: {},

  // Add hooks to collections
  hooks: {},

  // Modify operations
  operations: {},

  // Add validation
  validators: {},

  // Add utilities
  utils: {},

  // Plugin initialization
  init: ({ config, collections }) => {
    // Called when plugin is loaded
  }
})
```

## Adding Fields to Collections

### Global Fields (All Collections)

```typescript
export const globalFieldsPlugin = () => ({
  name: 'global-fields',

  // Add to ALL collections
  fields: {
    id: field({
      type: number(),
      primary: true
    }),
    createdAt: field({
      type: timestamp(),
      default: () => new Date()
    }),
    updatedAt: field({
      type: timestamp()
    })
  }
})
```

### Conditional Fields (Specific Collections)

```typescript
export const softDeletePlugin = () => ({
  name: 'soft-delete',

  // Add fields only to collections that opt in
  extend: (collection) => {
    if (collection.options?.softDelete !== true) return {}

    return {
      fields: {
        deletedAt: field({
          type: timestamp().nullable(),
          label: {
            en: 'Deleted At',
            fr: 'Supprimé le'
          }
        }),
        deletedBy: field({
          type: relation('users').nullable()
        })
      },

      hooks: {
        beforeDelete: [async ({ where, context }) => {
          // Soft delete instead
          await context.db.update(collection.slug)
            .set({ deletedAt: new Date() })
            .where(where)
          return false // Prevent actual delete
        }]
      }
    }
  }
})

// Usage
collection('posts', {
  softDelete: true, // Enable soft delete
  plugins: [softDeletePlugin()]
})
```

### Computed Fields

```typescript
export const computedFieldsPlugin = () => ({
  name: 'computed-fields',

  extend: (collection) => {
    const computedFields = {}

    // Auto-add slug field if title exists
    if (collection.fields.title) {
      computedFields.slug = field({
        type: text(),
        unique: true,
        computed: {
          from: ['title'],
          compute: ({ title }) => slugify(title)
        }
      })
    }

    // Auto-add fullName field if firstName and lastName exist
    if (collection.fields.firstName && collection.fields.lastName) {
      computedFields.fullName = field({
        type: text(),
        computed: {
          from: ['firstName', 'lastName'],
          compute: ({ firstName, lastName }) => `${firstName} ${lastName}`
        }
      })
    }

    return { fields: computedFields }
  }
})
```

## Adding Field Types

### Field Type Registration

```typescript
// plugins/specialized-fields.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText, pgJsonb } from 'drizzle-orm/pg-core'

// Rich text field
export const richText = fieldType({
  schema: z.object({
    raw: z.string(),
    html: z.string(),
    blocks: z.array(z.any())
  }),
  database: pgJsonb()
})

// Color field
export const color = fieldType({
  schema: z.string().regex(/^#[0-9A-F]{6}$/i),
  database: pgText()
})

// Coordinates field
export const coordinates = fieldType({
  schema: z.object({
    lat: z.number().min(-90).max(90),
    lng: z.number().min(-180).max(180)
  }),
  database: pgJsonb()
})

export const specializedFieldsPlugin = () => ({
  name: 'specialized-fields',

  fieldTypes: {
    richText,
    color,
    coordinates
  }
})
```

### Using Custom Field Types

```typescript
defineConfig({
  plugins: [specializedFieldsPlugin()],

  collections: [
    collection('posts', {
      fields: {
        // Custom field types are now available
        content: field({ type: richText() }),
        accentColor: field({ type: color() })
      }
    }),

    collection('locations', {
      fields: {
        name: field({ type: text() }),
        coordinates: field({ type: coordinates() })
      }
    })
  ]
})
```

### Parameterized Field Types

```typescript
// plugins/rating-field.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgInteger } from 'drizzle-orm/pg-core'

export const rating = (max: number = 5) => fieldType({
  schema: z.number().int().min(1).max(max),
  database: pgInteger()
})

export const ratingPlugin = () => ({
  name: 'rating-fields',

  fieldTypes: {
    rating5: rating(5),
    rating10: rating(10),
    rating100: rating(100)
  }
})

// Usage
collection('products', {
  plugins: [ratingPlugin()],
  fields: {
    quality: field({ type: rating5() }),
    price: field({ type: rating10() })
  }
})
```

## Adding Collections

### Single Collection Plugin

```typescript
// plugins/settings.ts
import { collection, field } from '@deessejs/collections'
import { text, json } from '@deessejs/collections/fields'

export const settingsPlugin = () => ({
  name: 'settings',

  collections: [
    collection('settings', {
      label: {
        en: 'Settings',
        fr: 'Paramètres'
      },
      fields: {
        key: field({
          type: text(),
          unique: true
        }),
        value: field({
          type: json()
        }),
        category: field({
          type: text()
        })
      }
    })
  ]
})
```

### Multiple Collections Plugin

```typescript
// plugins/user-management.ts
import { collection, field } from '@deessejs/collections'
import { text, email, boolean, enumField } from '@deessejs/collections/fields'

export const userManagementPlugin = () => ({
  name: 'user-management',

  collections: [
    collection('users', {
      fields: {
        name: field({ type: text() }),
        email: field({ type: email() }),
        isActive: field({ type: boolean() })
      }
    }),

    collection('roles', {
      fields: {
        name: field({ type: text() }),
        permissions: field({ type: array(text()) })
      }
    }),

    collection('permissions', {
      fields: {
        resource: field({ type: text() }),
        action: field({
          type: enumField(['create', 'read', 'update', 'delete'])
        })
      }
    }),

    collection('userRoles', {
      fields: {
        userId: field({ type: relation('users') }),
        roleId: field({ type: relation('roles') })
      }
    })
  ]
})
```

### Dynamic Collections

```typescript
// plugins/workflow.ts
export const workflowPlugin = (workflows: WorkflowDefinition[]) => ({
  name: 'workflow',

  collections: workflows.map(workflow =>
    collection(`${workflow.slug}WorkflowStates`, {
      fields: {
        status: field({
          type: enumField(workflow.states)
        }),
        entityId: field({
          type: number()
        }),
        assignedTo: field({
          type: relation('users')
        }),
        history: field({
          type: json()
        })
      }
    })
  )
})

// Usage
defineConfig({
  plugins: [
    workflowPlugin([
      { slug: 'article', states: ['draft', 'review', 'published'] },
      { slug: 'order', states: ['pending', 'processing', 'shipped'] }
    ])
  ]
})
```

## Plugin Hooks

### Collection-Level Hooks

```typescript
export const searchIndexPlugin = () => ({
  name: 'search-index',

  hooks: {
    afterCreate: [async ({ result, collection }) => {
      await searchEngine.index({
        collection: collection.slug,
        id: result.id,
        data: result
      })
    }],

    afterUpdate: [async ({ result, collection }) => {
      await searchEngine.update({
        collection: collection.slug,
        id: result.id,
        data: result
      })
    }],

    afterDelete: [async ({ id, collection }) => {
      await searchEngine.remove({
        collection: collection.slug,
        id
      })
    }]
  }
})
```

### Operation Modification

```typescript
export const cachePlugin = () => ({
  name: 'cache',

  operations: {
    findMany: {
      before: async ({ query }) => {
        // Check cache
        const cached = await cache.get(query.hash)
        if (cached) return cached
      },

      after: async ({ result, query }) => {
        // Store in cache
        await cache.set(query.hash, result, { ttl: 300 })
        return result
      }
    },

    findUnique: {
      before: async ({ query }) => {
        const cached = await cache.get(query.hash)
        if (cached) return cached
      },

      after: async ({ result, query }) => {
        await cache.set(query.hash, result, { ttl: 300 })
        return result
      }
    }
  }
})
```

### Validation Hooks

```typescript
export const validationPlugin = () => ({
  name: 'validation',

  validators: {
    beforeCreate: async ({ data, collection }) => {
      // Custom validation before create
      const errors = []

      if (collection.uniqueFields) {
        for (const field of collection.uniqueFields) {
          const exists = await db.select()
            .from(collection.slug)
            .where(eq(collection.slug[field], data[field]))
            .limit(1)

          if (exists.length > 0) {
            errors.push({
              field,
              message: `${field} must be unique`
            })
          }
        }
      }

      if (errors.length > 0) {
        throw new ValidationError(errors)
      }
    }
  }
})
```

## Plugin Composition

### Combining Multiple Plugins

```typescript
defineConfig({
  plugins: [
    timestampsPlugin(),
    softDeletePlugin(),
    searchIndexPlugin(),
    auditLogPlugin()
  ]
})
```

### Plugin Dependencies

```typescript
export const advancedAuditPlugin = () => ({
  name: 'advanced-audit',

  // This plugin requires timestampsPlugin
  requires: ['timestamps'],

  init: ({ config }) => {
    // Verify required plugins are loaded
    if (!config.plugins.timestamps) {
      throw new Error('timestampsPlugin is required')
    }
  },

  hooks: {
    afterCreate: [async ({ result }) => {
      // Can rely on createdAt field existing
      console.log(`Created at ${result.createdAt}`)
    }]
  }
})
```

### Plugin Ordering

```typescript
defineConfig({
  plugins: [
    timestampsPlugin(),        // Runs first
    searchIndexPlugin(),       // Runs second
    notificationPlugin(),      // Runs third
    analyticsPlugin()          // Runs last
  ]
})

// Or explicit order
defineConfig({
  plugins: [
    {
      plugin: timestampsPlugin(),
      order: 1
    },
    {
      plugin: searchIndexPlugin(),
      order: 2
    }
  ]
})
```

## Conditional Plugin Application

### Per-Collection Plugins

```typescript
export const versioningPlugin = () => ({
  name: 'versioning',

  extend: (collection) => {
    // Only add to collections with versioning enabled
    if (collection.options?.versioning !== true) {
      return {}
    }

    const versionCollectionName = `${collection.slug}Versions`

    return {
      collections: [
        collection(versionCollectionName, {
          fields: {
            versionId: field({ type: number() }),
            data: field({ type: json() }),
            createdAt: field({ type: timestamp() }),
            createdBy: field({ type: relation('users') })
          }
        })
      ],

      hooks: {
        beforeUpdate: [async ({ result, context }) => {
          // Save version before updating
          await context.collections[versionCollectionName].create({
            data: {
              versionId: result.id,
              data: result,
              createdAt: new Date(),
              createdBy: context.currentUser
            }
          })
        }]
      }
    }
  }
})

// Usage
collection('posts', {
  versioning: true, // Enable versioning
  plugins: [versioningPlugin()]
})

collection('comments', {
  versioning: false, // No versioning
  plugins: [versioningPlugin()]
})
```

### Environment-Based Plugins

```typescript
export const monitoringPlugin = () => ({
  name: 'monitoring',

  init: ({ config }) => {
    // Only enable in production
    if (process.env.NODE_ENV !== 'production') {
      return { disabled: true }
    }
  },

  hooks: {
    afterCreate: [async ({ result }) => {
      analytics.track('entity.created', {
        collection: result.collection,
        id: result.id
      })
    }]
  }
})
```

## Plugin Options

### Configurable Plugins

```typescript
export const slugPlugin = (options: {
  from: string
  separator?: string
  lowercase?: boolean
  unique?: boolean
} = {
  from: 'title',
  separator: '-',
  lowercase: true,
  unique: false
}) => ({
  name: 'slug',

  fields: {
    slug: field({
      type: text({
        unique: options.unique
      }),
      computed: {
        from: [options.from],
        compute: ({ [options.from]: value }) => {
          let slug = value
            .toString()
            .split(options.separator)
            .join('-')

          if (options.lowercase) {
            slug = slug.toLowerCase()
          }

          return slug
        }
      }
    })
  }
})

// Usage
collection('posts', {
  plugins: [
    slugPlugin({
      from: 'title',
      separator: '_',
      unique: true
    })
  ]
})
```

### Plugin with Presets

```typescript
export const seoPlugin = (preset: 'basic' | 'advanced' = 'basic') => ({
  name: 'seo',

  fields: preset === 'advanced' ? {
    metaTitle: field({ type: text() }),
    metaDescription: field({ type: text() }),
    ogImage: field({ type: text() }),
    twitterCard: field({ type: enumField(['summary', 'summary_large_image']) }),
    canonicalUrl: field({ type: text() }),
    robots: field({ type: text() }),
    structuredData: field({ type: json() })
  } : {
    metaTitle: field({ type: text() }),
    metaDescription: field({ type: text() })
  }
})

// Usage
collection('posts', {
  plugins: [
    seoPlugin('basic')    // or 'advanced'
  ]
})
```

## Plugin Utilities

### Helper Functions

```typescript
export const validationHelpersPlugin = () => ({
  name: 'validation-helpers',

  utils: {
    isEmail: (value: string) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    isUrl: (value: string) => /^https?:\/\//.test(value),
    isPhone: (value: string) => /^\+?[\d\s-()]+$/.test(value),

    slugify: (value: string) => {
      return value
        .toLowerCase()
        .trim()
        .replace(/[^\w\s-]/g, '')
        .replace(/[\s_-]+/g, '-')
        .replace(/^-+|-+$/g, '')
    },

    truncate: (text: string, length: number) => {
      return text.length > length ? text.substring(0, length) + '...' : text
    }
  }
})

// Usage in hooks
collection('posts', {
  plugins: [validationHelpersPlugin()],

  hooks: {
    beforeCreate: [async ({ data, utils }) => {
      // Access plugin utilities
      if (!utils.isEmail(data.email)) {
        throw new Error('Invalid email')
      }

      data.slug = utils.slugify(data.title)
    }]
  }
})
```

## Publishing Plugins

### Plugin Package Structure

```typescript
// packages/deesse-cms-plugins/src/index.ts
export { timestampsPlugin } from './timestamps'
export { softDeletePlugin } from './soft-delete'
export { searchPlugin } from './search'
export { seoPlugin } from './seo'
export { auditLogPlugin } from './audit-log'

// Re-export field types
export * from './field-types'

// packages/deesse-cms-plugins/package.json
{
  "name": "@deessejs/cms-plugins",
  "version": "1.0.0",
  "main": "./dist/index.js",
  "types": "./dist/index.d.ts",
  "peerDependencies": {
    "@deessejs/collections": "^1.0.0"
  }
}
```

### Using External Plugins

```typescript
// app.ts
import { timestampsPlugin, seoPlugin } from '@deessejs/cms-plugins'
import { customPlugin } from '@my-org/custom-plugins'

defineConfig({
  plugins: [
    timestampsPlugin(),
    seoPlugin('advanced'),
    customPlugin({ option: 'value' })
  ]
})
```

## Best Practices

### Plugin Naming

```typescript
// Good: Descriptive names
export const timestampsPlugin = () => ({ name: 'timestamps' })
export const fullTextSearchPlugin = () => ({ name: 'full-text-search' })

// Bad: Generic names
export const plugin = () => ({ name: 'plugin' })
export const helper = () => ({ name: 'helper' })
```

### Plugin Isolation

```typescript
// Good: Namespaced fields
export const seoPlugin = () => ({
  fields: {
    seoTitle: field({ type: text() }),
    seoDescription: field({ type: text() })
  }
})

// Bad: Risk of collision
export const seoPlugin = () => ({
  fields: {
    title: field({ type: text() }),  // Might collide!
    description: field({ type: text() })
  }
})
```

### Plugin Documentation

```typescript
/**
 * Adds soft delete functionality to collections.
 *
 * @example
 * collection('posts', {
 *   plugins: [softDeletePlugin()]
 * })
 *
 * Adds fields:
 * - deletedAt: Timestamp | null
 * - deletedBy: Relation to users
 *
 * Modifies behavior:
 * - Delete operations set deletedAt instead of removing records
 * - Queries filter out soft-deleted records by default
 */
export const softDeletePlugin = () => ({ ... })
```

## Benefits of the Plugin System

### Composability

- Combine multiple plugins without conflicts
- Each plugin is independent
- Clear ordering and dependencies

### Type Safety

- All plugin extensions are fully typed
- Autocomplete for plugin-added fields
- Compile-time validation

### Reusability

- Share plugins as npm packages
- Community plugin ecosystem
- Easy to vendor/customize

### Performance

- Plugins loaded at startup
- Zero runtime overhead
- Tree-shake unused plugins
