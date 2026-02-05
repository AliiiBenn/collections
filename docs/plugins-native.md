# Native Plugins

## Overview

`@deessejs/collections` includes several powerful, production-ready plugins that solve common needs. These plugins are built into the framework, fully typed, and optimized for performance.

## Available Native Plugins

1. **Versioning** - Track changes to records over time
2. **Cache** - Intelligent caching for queries
3. **Soft Delete** - Safe deletion with recovery
4. **SEO** - Search engine optimization fields

---

## Versioning Plugin

Track all changes to your records with automatic versioning.

### Basic Usage

```typescript
import { defineConfig } from '@deessejs/collections'
import { versioningPlugin } from '@deessejs/collections/plugins'
import { posts, users } from '../collections'

// Plugins are defined at config level
export const { collections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!
  },

  // Apply plugins globally
  plugins: [
    versioningPlugin({
      // Apply to specific collections
      collections: ['posts', 'pages'],

      // Track specific fields
      fields: ['title', 'content', 'status'],

      // Keep last N versions (null = unlimited)
      maxVersions: 50,

      // Who made the change
      userField: 'authorId',

      // Add notes to versions
      allowNotes: true
    })
  ],

  collections: [
    posts,
    users
  ]
})
```

### What It Adds

**New Collection**: `postsVersions`

```typescript
{
  id: number
  postId: number           // Reference to original record
  version: number          // Version number
  data: json              // Complete snapshot of record
  changes: json           // Only changed fields
  createdAt: timestamp
  createdBy: number       // User who made the change
  note?: string           // Optional version note
}
```

**New Operations on Original Collection**:

```typescript
// Get all versions
const versions = await collections.posts.getVersions({
  where: { id: 1 }
})

// Get specific version
const version = await collections.posts.getVersion({
  id: 1,
  version: 5
})

// Restore a version
const restored = await collections.posts.restoreVersion({
  id: 1,
  version: 5,
  note: 'Restored after accidental deletion'
})

// Compare versions
const diff = await collections.posts.compareVersions({
  id: 1,
  from: 3,
  to: 5
})

// Get version history
const history = await collections.posts.getVersionHistory({
  id: 1,
  limit: 10
})
```

### Automatic Versioning

```typescript
// Create - Automatically creates version 1
const post = await collections.posts.create({
  data: {
    title: 'Hello World',
    content: 'My first post'
  }
  // Version 1 created automatically
})

// Update - Creates version 2
await collections.posts.update({
  where: { id: 1 },
  data: {
    title: 'Hello World!' // Changed
  }
  // Version 2 created (only tracks changed fields)
})

// Restore - Creates version 3
await collections.posts.restoreVersion({
  id: 1,
  version: 1
})
// Version 3 created
```

### Version Notes

```typescript
// Update with a note
await collections.posts.update({
  where: { id: 1 },
  data: {
    title: 'New Title'
  },
  versionNote: 'Fixed typo in title' // Stored with version
})

// Restore with note
await collections.posts.restoreVersion({
  id: 1,
  version: 5,
  note: 'Reverting to previous version after client request'
})
```

### Querying with Versions

```typescript
// Get post with version info
const post = await collections.posts.findUnique({
  where: { id: 1 },
  include: {
    versions: {
      limit: 5,
      orderBy: { createdAt: 'desc' }
    }
  }
})

// Result:
{
  id: 1,
  title: 'Hello World',
  content: '...',
  versions: [
    { version: 5, createdAt: '...', changes: { title: '...' } },
    { version: 4, createdAt: '...', changes: { content: '...' } },
    // ...
  ]
}
```

### Version Diff

```typescript
const diff = await collections.posts.compareVersions({
  id: 1,
  from: 3,
  to: 5
})

// Result:
{
  versionFrom: 3,
  versionTo: 5,
  changes: {
    title: {
      from: 'Old Title',
      to: 'New Title'
    },
    content: {
      from: 'Old content',
      to: 'New content'
    }
  },
  changedAt: '...',
  changedBy: { id: 1, name: 'John' }
}
```

### Selective Versioning

```typescript
// Only version specific fields
versioningPlugin({
  fields: ['title', 'content'], // Only track these
  ignoreFields: ['views', 'lastViewed'] // Never track these
})
```

### Version Cleanup

```typescript
// Auto-cleanup old versions
versioningPlugin({
  maxVersions: 50, // Keep only last 50 versions
  cleanupAfter: '90 days', // Delete versions older than 90 days
  autoCleanup: true // Automatically clean up
})

// Manual cleanup
await collections.posts.cleanupVersions({
  olderThan: '90 days',
  keepLast: 10
})
```

### Configuration Options

```typescript
versioningPlugin({
  // Fields to track (all if not specified)
  fields?: string[],

  // Maximum versions to keep
  maxVersions?: number | null,

  // User field for tracking who made changes
  userField?: string,

  // Allow adding notes to versions
  allowNotes?: boolean,

  // Fields to never track
  ignoreFields?: string[],

  // Auto-cleanup settings
  autoCleanup?: boolean,
  cleanupAfter?: string,

  // Custom version collection name
  versionsCollection?: string,

  // Store full snapshot or only changes
  snapshotMode?: 'full' | 'changes'
})
```

---

## Cache Plugin

Intelligent caching layer for improved query performance.

### Basic Usage

```typescript
import { defineConfig } from '@deessejs/collections'
import { cachePlugin } from '@deessejs/collections/plugins'
import { posts, comments, users } from '../collections'

export const { collections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!
  },

  plugins: [
    cachePlugin({
      // Apply to all collections or specific ones
      collections: ['posts', 'pages', 'users'],

      // Cache strategy
      strategy: 'lru', // 'lru' | 'ttl' | 'smart'

      // TTL in seconds
      ttl: 300, // 5 minutes

      // Max cache size
      maxSize: 1000,

      // What to cache
      cacheFindMany: true,
      cacheFindUnique: true,
      cacheCount: true,

      // Cache invalidation
      invalidateOnUpdate: true,
      invalidateOnDelete: true
    })
  ],

  collections: [
    posts,
    comments, // Not cached
    users
  ]
})
```

### Cache Strategies

**LRU (Least Recently Used)**

```typescript
cachePlugin({
  strategy: 'lru',
  maxSize: 1000 // Keep 1000 most recent queries
})
```

**TTL (Time To Live)**

```typescript
cachePlugin({
  strategy: 'ttl',
  ttl: 300 // Cache for 5 minutes
})
```

**Smart (Adaptive)**

```typescript
cachePlugin({
  strategy: 'smart',

  // Cache frequently accessed data longer
  hotDataTTL: 600,     // 10 minutes

  // Cache rarely accessed data shorter
  coldDataTTL: 60,     // 1 minute

  // Learn from access patterns
  adaptive: true
})
```

### Selective Caching

```typescript
cachePlugin({
  // Cache different operations differently
  cacheFindMany: {
    enabled: true,
    ttl: 300
  },

  cacheFindUnique: {
    enabled: true,
    ttl: 600 // Longer TTL for single records
  },

  cacheCount: {
    enabled: false // Don't cache counts (change frequently)
  }
})
```

### Cache Keys

```typescript
// Automatic cache key generation
await collections.posts.findMany({
  where: { status: 'published' },
  orderBy: { createdAt: 'desc' },
  limit: 10
})

// Cache key automatically generated:
// "posts:findMany:status=published,orderBy=createdAt:desc,limit=10"
```

### Cache Invalidation

```typescript
cachePlugin({
  // Invalidate on writes
  invalidateOnCreate: true,
  invalidateOnUpdate: true,
  invalidateOnDelete: true,

  // Invalidation strategy
  invalidationStrategy: 'exact' | 'pattern' | 'all',

  // Invalidate related collections
  invalidateRelated: ['comments', 'likes']
})

// Manual invalidation
await collections.posts.cache.invalidate({
  // Invalidate specific query
  key: 'posts:findMany:status=published',

  // Or invalidate pattern
  pattern: 'posts:findMany:*',

  // Or invalidate all
  all: true
})
```

### Cache Warming

```typescript
// Pre-populate cache
await collections.posts.cache.warmup([
  { where: { status: 'published' }, limit: 10 },
  { where: { featured: true }, limit: 5 }
])

// Or warmup on startup
cachePlugin({
  warmupOnStartup: true,
  warmupQueries: [
    { where: { status: 'published' }, limit: 10 }
  ]
})
```

### Cache Statistics

```typescript
// Get cache statistics
const stats = await collections.posts.cache.getStats()

// Result:
{
  hits: 1500,           // Cache hits
  misses: 300,          // Cache misses
  hitRate: 0.83,        // 83% hit rate
  size: 250,            // Current cache size
  memoryUsage: '15MB',  // Memory used
  keys: [ /* ... */ ]    // All cache keys
}

// Reset stats
await collections.posts.cache.resetStats()
```

### Distributed Cache

```typescript
// Use Redis for distributed cache
cachePlugin({
  strategy: 'redis',

  redis: {
    host: 'localhost',
    port: 6379,
    password: process.env.REDIS_PASSWORD,
    db: 0
  },

  // Cache key prefix
  keyPrefix: 'deesse:cache:',

  // Serialization
  serialize: JSON.stringify,
  deserialize: JSON.parse
})
```

### Cache Hooks

```typescript
cachePlugin({
  hooks: {
    beforeCache: async ({ key, value }) => {
      // Transform before caching
      return value
    },

    afterCache: async ({ key, value }) => {
      // Post-cache hook
    },

    onHit: async ({ key, value }) => {
      // Called on cache hit
    },

    onMiss: async ({ key }) => {
      // Called on cache miss
    }
  }
})
```

### Tag-Based Caching

```typescript
cachePlugin({
  tags: ['posts', 'content'],

  // Invalidate by tag
  invalidateTag: async (tag) => {
    // Invalidate all entries with this tag
  }
})

// Tag queries
await collections.posts.findMany({
  where: { featured: true },
  cache: {
    tags: ['featured', 'homepage']
  }
})

// Invalidate all homepage cache
await collections.posts.cache.invalidateTag('homepage')
```

---

## Soft Delete Plugin

Safe deletion with recovery options.

### Basic Usage

```typescript
import { defineConfig } from '@deessejs/collections'
import { softDeletePlugin } from '@deessejs/collections/plugins'
import { posts, comments, users } from '../collections'

export const { collections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!
  },

  plugins: [
    softDeletePlugin({
      // Apply to specific collections
      collections: ['posts', 'pages', 'comments'],

      // Field names
      deletedAtField: 'deletedAt',
      deletedByField: 'deletedBy',

      // Cascade soft delete to relations
      cascade: ['comments', 'likes'],

      // Allow permanent delete
      allowPermanentDelete: true,

      // Default behavior
      defaultToSoftDelete: true
    })
  ],

  collections: [
    posts,      // Has soft delete
    comments,   // Has soft delete
    users       // No soft delete
  ]
})
```

### What It Adds

**New Fields**:

```typescript
{
  deletedAt: timestamp | null   // When it was soft deleted
  deletedBy: relation('users') | null  // Who deleted it
}
```

**Modified Operations**:

```typescript
// Delete now sets deletedAt
await collections.posts.delete({
  where: { id: 1 }
})
// Sets: { deletedAt: new Date(), deletedBy: currentUser.id }

// Queries automatically filter out soft-deleted records
const posts = await collections.posts.findMany()
// Does NOT include deleted posts

// Include soft-deleted records
const allPosts = await collections.posts.findMany({
  includeDeleted: true
})
```

### Restore Operations

```typescript
// Restore a soft-deleted record
await collections.posts.restore({
  where: { id: 1 }
})

// Restore with modifications
await collections.posts.restore({
  where: { id: 1 },
  data: {
    title: 'Restored Post'
  }
})

// Bulk restore
await collections.posts.restoreMany({
  where: {
    deletedAt: { not: null }
  }
})
```

### Permanent Delete

```typescript
// Permanently delete (bypass soft delete)
await collections.posts.permanentDelete({
  where: { id: 1 }
})

// Or force delete
await collections.posts.delete({
  where: { id: 1 },
  permanent: true
})
```

### Querying Soft-Deleted Records

```typescript
// Only soft-deleted records
const deletedPosts = await collections.posts.findMany({
  where: {
    deletedAt: { not: null }
  }
})

// Mixed queries
const posts = await collections.posts.findMany({
  includeDeleted: true,
  where: {
    deletedAt: { not: null } // Only deleted
  }
})
```

### Cascade Soft Delete

```typescript
softDeletePlugin({
  // Automatically soft delete related records
  cascade: ['comments', 'likes', 'shares'],

  // Cascade behavior
  cascadeBehavior: 'delete' | 'nullify' | 'check'
})

// When you delete a post, its comments are also soft deleted
await collections.posts.delete({ where: { id: 1 } })
// All comments for post 1 are also soft deleted
```

### Soft Delete with Relations

```typescript
collection('posts', {
  plugins: [softDeletePlugin()],
  fields: {
    comments: field({
      type: relation('comments', { many: true }),
      // Automatically handle soft deletes in relations
      softDeleteRelations: true
    })
  }
})

// Doesn't include comments from deleted posts
const posts = await collections.posts.findMany({
  include: {
    comments: true
  }
})
```

### Deleted By Tracking

```typescript
// Track who deleted what
await collections.posts.delete({
  where: { id: 1 },
  context: {
    currentUser: { id: 5, name: 'John' }
  }
})
// Sets: deletedAt: timestamp, deletedBy: 5

// Get deletion info
const post = await collections.posts.findUnique({
  where: { id: 1 },
  includeDeleted: true,
  include: {
    deletedBy: true // Include user who deleted it
  }
})
```

### Trash/Recycle Bin

```typescript
// Get all soft-deleted records
const trash = await collections.posts.trash()

// Empty trash (permanent delete)
await collections.posts.emptyTrash({
  olderThan: '30 days' // Only delete items older than 30 days
})

// Restore all from trash
await collections.posts.restoreTrash({
  where: {
    deletedAt: { gte: new Date('2024-01-01') }
  }
})
```

### Configuration Options

```typescript
softDeletePlugin({
  // Field configuration
  deletedAtField?: string,
  deletedByField?: string,

  // Cascade settings
  cascade?: string[],
  cascadeBehavior?: 'delete' | 'nullify' | 'check',

  // Delete behavior
  allowPermanentDelete?: boolean,
  defaultToSoftDelete?: boolean,

  // Query behavior
  includeDeletedByDefault?: boolean,

  // Index deletedAt for performance
  indexDeletedAt?: boolean
})
```

---

## SEO Plugin

Complete SEO meta tags and optimization fields.

### Basic Usage

```typescript
import { defineConfig } from '@deessejs/collections'
import { seoPlugin } from '@deessejs/collections/plugins'
import { posts, pages, products } from '../collections'

export const { collections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!
  },

  plugins: [
    seoPlugin({
      // Apply to collections that need SEO
      collections: ['posts', 'pages', 'products'],

      // Preset: 'basic' | 'advanced' | 'full'
      preset: 'advanced',

      // Generate from fields
      generateFrom: {
        metaTitle: 'title',
        metaDescription: 'content',
        ogImage: 'featuredImage'
      },

      // Character limits
      limits: {
        metaTitle: 60,
        metaDescription: 160,
        twitterTitle: 70
      },

      // Validation
      validateOnPublish: true
    })
  ],

  collections: [
    posts,     // Has SEO
    pages,     // Has SEO
    products,  // Has SEO
    comments   // No SEO
  ]
})
```

### What It Adds

**Basic Preset**:

```typescript
{
  // Meta tags
  metaTitle: text,
  metaDescription: text,

  // Open Graph
  ogTitle: text,
  ogDescription: text,
  ogImage: text,

  // Twitter Card
  twitterCard: enumField(['summary', 'summary_large_image', 'app', 'player']),
  twitterTitle: text,
  twitterDescription: text,
  twitterImage: text
}
```

**Advanced Preset** (adds):

```typescript
{
  // Additional meta
  canonicalUrl: text,
  noindex: boolean,
  nofollow: boolean,

  // Structured data
  structuredData: json,

  // Alternates
  alternateLanguages: json, // { fr: '/fr/page', es: '/es/page' }

  // SEO scores
  seoScore: number, // 0-100
  keywordDensity: number,
  readabilityScore: number
}
```

**Full Preset** (adds):

```typescript
{
  // Advanced tags
  robots: text, // "index, follow" etc
  googlebot: text,
  bingbot: text,

  // Social preview
  socialPreview: json,

  // Keywords
  focusKeyword: text,
  keywords: array(text()),

  // Analytics
  tracking: json,

  // Sitemap
  sitemapPriority: enumField(['0.0', '0.1', ..., '1.0']),
  sitemapChangeFreq: enumField(['always', 'hourly', 'daily', 'weekly', 'monthly', 'yearly', 'never'])
}
```

### Auto-Generation

```typescript
seoPlugin({
  generateFrom: {
    metaTitle: 'title',
    metaDescription: 'content',
    ogTitle: 'title',
    ogDescription: 'content',
    twitterTitle: 'title',
    twitterDescription: 'content'
  },

  // Truncate to limits
  truncateToLimits: true,

  // Strip HTML
  stripHtml: true,

  // Remove duplicates
  removeDuplicates: true
})

// On create/update, auto-generates from content
const post = await collections.posts.create({
  data: {
    title: 'My Amazing Blog Post About SEO',
    content: 'This is a long article about SEO optimization...'
  }
})

// Auto-generated:
// metaTitle: "My Amazing Blog Post About SEO"
// metaDescription: "This is a long article about SEO..." (truncated to 160 chars)
```

### Validation

```typescript
seoPlugin({
  validation: {
    // Required for publish
    requiredForPublish: ['metaTitle', 'metaDescription'],

    // Length limits
    metaTitle: { min: 30, max: 60 },
    metaDescription: { min: 120, max: 160 },

    // Custom validation
    validate: (data) => {
      if (data.focusKeyword && !data.content.includes(data.focusKeyword)) {
        throw new Error('Content should include focus keyword')
      }
    }
  },

  // Validate on publish
  validateOnPublish: true
})

// Fails validation if SEO incomplete
await collections.posts.update({
  where: { id: 1 },
  data: {
    status: 'published', // Will fail if metaTitle/metaDescription missing
    seo: { /* ... */ }
  }
})
```

### SEO Score

```typescript
// Auto-calculate SEO score
seoPlugin({
  scoring: {
    enabled: true,
    factors: {
      titleLength: 10,
      descriptionLength: 10,
      keywordInTitle: 15,
      keywordInContent: 20,
      contentLength: 15,
      internalLinks: 10,
      imagesWithAlt: 10,
      readability: 10
    }
  }
})

// Get SEO score
const post = await collections.posts.findUnique({
  where: { id: 1 },
  select: {
    seoScore: true,
    seoIssues: true // Array of issues
  }
})

// Result:
{
  seoScore: 75,
  seoIssues: [
    'Meta description is too short (120 chars, recommended 155)',
    'Content missing internal links',
    'Focus keyword not in first paragraph'
  ]
}
```

### Structured Data

```typescript
// Auto-generate structured data
seoPlugin({
  structuredData: {
    enabled: true,
    types: ['Article', 'BlogPosting', 'NewsArticle'],
    generateFrom: 'content'
  }
})

// Access structured data
const post = await collections.posts.findUnique({
  where: { id: 1 },
  select: {
    title: true,
    structuredData: true
  }
})

// Result:
{
  title: 'My Post',
  structuredData: {
    '@context': 'https://schema.org',
    '@type': 'BlogPosting',
    'headline': 'My Post',
    'datePublished': '2024-01-01',
    'author': {
      '@type': 'Person',
      'name': 'John'
    }
  }
}
```

### Social Preview

```typescript
// Generate social preview data
seoPlugin({
  socialPreview: {
    enabled: true,
    templates: {
      twitter: '📝 {title} - {siteName}',
      facebook: '{title} | {siteName}',
      linkedin: '{title} - {description}'
    }
  }
})

// Get preview
const preview = await collections.posts.getSocialPreview({
  where: { id: 1 }
})

// Result:
{
  twitter: {
    title: '📝 My Post - My Blog',
    description: '...',
    image: 'https://...',
    cardType: 'summary_large_image'
  },
  facebook: {
    title: 'My Post | My Blog',
    description: '...',
    image: 'https://...'
  }
}
```

### Sitemap Integration

```typescript
seoPlugin({
  sitemap: {
    enabled: true,
    priority: 'dynamic', // Or fixed value
    changeFreq: 'weekly',
    excludeField: 'noindex'
  }
})

// Generate sitemap
const sitemap = await collections.posts.generateSitemap()

// Result:
[
  {
    url: 'https://example.com/posts/my-post',
    lastmod: '2024-01-01',
    changefreq: 'weekly',
    priority: 0.8
  },
  // ...
]
```

### Canonical URLs

```typescript
// Auto-generate canonical URLs
seoPlugin({
  canonicalUrl: {
    enabled: true,
    pattern: '/posts/{slug}',
    trailingSlash: false,
    lowercase: true
  }
})

// Access canonical URL
const post = await collections.posts.findUnique({
  where: { id: 1 },
  select: {
    slug: true,
    canonicalUrl: true
  }
})

// Result:
{
  slug: 'my-post',
  canonicalUrl: 'https://example.com/posts/my-post'
}
```

### Multi-Language SEO

```typescript
seoPlugin({
  i18n: {
    enabled: true,
    locales: ['en', 'fr', 'es'],
    defaultLocale: 'en',
    alternateUrls: true
  }
})

// Access alternate URLs
const post = await collections.posts.findUnique({
  where: { id: 1 },
  select: {
    seoAlternateLanguages: true
  }
})

// Result:
{
  seoAlternateLanguages: {
    en: 'https://example.com/en/my-post',
    fr: 'https://example.com/fr/mon-article',
    es: 'https://example.com/es/mi-articulo'
  }
}
```

### SEO Helpers

```typescript
// Analyze content for SEO
const analysis = await collections.posts.analyzeSEO({
  content: 'My blog post content...',
  focusKeyword: 'SEO tips'
})

// Result:
{
  score: 75,
  titleLength: 25,
  descriptionLength: 145,
  keywordDensity: 1.5,
  readabilityScore: 80,
  suggestions: [
    'Add focus keyword to first paragraph',
    'Increase meta description to 155 characters',
    'Add more internal links'
  ]
}

// Get keyword suggestions
const suggestions = await collections.posts.getKeywordSuggestions({
  content: '...',
  limit: 10
})
```

## Using Multiple Native Plugins

```typescript
// config/database.ts
import { defineConfig } from '@deessejs/collections'
import {
  versioningPlugin,
  cachePlugin,
  softDeletePlugin,
  seoPlugin
} from '@deessejs/collections/plugins'

import { posts, comments, users, pages } from '../collections'

export const { collections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!
  },

  // All plugins defined at config level
  plugins: [
    // Versioning for posts and pages
    versioningPlugin({
      collections: ['posts', 'pages'],
      maxVersions: 50,
      userField: 'authorId'
    }),

    // Cache for read-heavy collections
    cachePlugin({
      collections: ['posts', 'pages', 'users'],
      strategy: 'smart',
      ttl: 600
    }),

    // Soft delete for main content
    softDeletePlugin({
      collections: ['posts', 'pages', 'comments'],
      cascade: ['comments'],
      allowPermanentDelete: true
    }),

    // SEO for public content
    seoPlugin({
      collections: ['posts', 'pages'],
      preset: 'advanced',
      validateOnPublish: true
    })
  ],

  collections: [
    posts,    // Has: versioning, cache, soft delete, SEO
    pages,    // Has: versioning, cache, soft delete, SEO
    comments, // Has: cache, soft delete
    users     // Has: cache only
  ]
})
```

## Plugin Configuration Best Practices

### Environment-Specific Config

```typescript
const isProduction = process.env.NODE_ENV === 'production'

export const { collections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!
  },

  plugins: [
    cachePlugin({
      collections: ['posts', 'users'],
      strategy: isProduction ? 'redis' : 'memory',
      ttl: isProduction ? 600 : 60,
      redis: isProduction ? {
        host: process.env.REDIS_HOST,
        port: 6379
      } : undefined
    }),

    seoPlugin({
      collections: ['posts', 'pages'],
      validation: isProduction
        ? { validateOnPublish: true }
        : { validateOnPublish: false }
    })
  ],

  collections: [/* ... */]
})
```

### Per-Collection Configuration

```typescript
// Configure different cache settings per collection
cachePlugin({
  collections: ['posts', 'comments', 'users'],

  // Per-collection overrides
  config: {
    posts: {
      ttl: 600, // Cache longer
      strategy: 'smart'
    },
    comments: {
      ttl: 60, // Cache shorter (changes frequently)
      strategy: 'lru'
    },
    users: {
      ttl: 300,
      strategy: 'ttl'
    }
  }
})
```

### Selective Plugin Application

```typescript
// Apply plugins to specific collections only
export const { collections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!
  },

  plugins: [
    // Only version posts and pages, not comments
    versioningPlugin({
      collections: ['posts', 'pages'],
      maxVersions: 50
    }),

    // SEO only for public content
    seoPlugin({
      collections: ['posts', 'pages', 'products'],
      preset: 'advanced'
    }),

    // Comments get soft delete but no versioning
    softDeletePlugin({
      collections: ['posts', 'pages', 'comments']
    })
  ],

  collections: [
    posts,    // versioning + soft delete + SEO
    pages,    // versioning + soft delete + SEO
    comments, // soft delete only
    users     // none of the above
  ]
})
```

### Plugin Ordering

When multiple plugins affect the same collection, order matters:

```typescript
export const { collections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!
  },

  plugins: [
    // 1. Timestamps runs first (adds createdAt/updatedAt)
    timestampsPlugin(),

    // 2. Soft delete adds deletedAt/deletedBy
    softDeletePlugin({
      collections: ['posts', 'comments']
    }),

    // 3. Versioning tracks all changes including soft delete
    versioningPlugin({
      collections: ['posts']
    }),

    // 4. Cache is last (caches the final results)
    cachePlugin({
      collections: ['posts', 'users']
    })
  ],

  collections: [/* ... */]
})
```
