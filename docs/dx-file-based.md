# Developer Experience: File-Based Collections

## Overview

While you can define collections inline in `defineConfig`, the recommended approach is to define each collection in its own file. This promotes better organization, reusability, and maintainability as your application grows.

## Collection Files

### Basic Structure

Each collection lives in its own file and exports a collection definition:

```typescript
// collections/posts.ts
import { collection, field, text } from '@deessejs/collections'

export const posts = collection({
  slug: 'posts',

  fields: {
    title: field({
      type: text({ min: 3, max: 255 })
    }),

    content: field({
      type: text()
    }),

    status: field({
      type: enumField({
        options: ['draft', 'published', 'archived']
      }),
      defaultValue: 'draft'
    })
  }
})
```

### Collection in Separate Files

```typescript
// collections/users.ts
import { collection, field, text, email, boolean } from '@deessejs/collections'

export const users = collection({
  slug: 'users',

  fields: {
    name: field({
      type: text({ min: 2, max: 100 })
    }),

    email: field({
      type: email(),
      unique: true
    }),

    isActive: field({
      type: boolean(),
      defaultValue: true
    })
  }
})

// collections/comments.ts
import { collection, field, text } from '@deessejs/collections'

export const comments = collection({
  slug: 'comments',

  fields: {
    content: field({
      type: text({ min: 1, max: 1000 })
    }),

    authorId: field({
      type: relation({ collection: 'users' })
    }),

    postId: field({
      type: relation({ collection: 'posts' })
    })
  }
})
```

## Field Definition Syntax

### Type-Based Field Definition

Instead of chaining methods, fields are defined with a type object:

```typescript
// Old approach (not used)
field.text('title').minLength(3).maxLength(255).required()

// New approach
title: field({
  type: text({ min: 3, max: 255 }),
  required: true
})
```

### Primitive Field Types

```typescript
fields: {
  // Text with constraints
  title: field({
    type: text({ min: 3, max: 255 }),
    required: true
  }),

  // Email field
  email: field({
    type: email(),
    unique: true
  }),

  // Number field
  age: field({
    type: number({ min: 0, max: 120, integer: true })
  }),

  // Boolean field
  isActive: field({
    type: boolean(),
    defaultValue: true
  }),

  // Date field
  birthDate: field({
    type: date()
  }),

  // Timestamp field
  createdAt: field({
    type: timestamp(),
    defaultValue: () => new Date()
  }),

  // Enum field
  role: field({
    type: enumField({
      options: ['user', 'admin', 'moderator']
    }),
    defaultValue: 'user'
  })
}
```

### Structural Field Types

```typescript
fields: {
  // Array field
  tags: field({
    type: array({
      of: text()
    })
  }),

  // Object field (nested structure)
  metadata: field({
    type: object({
      fields: {
        key: field({ type: text() }),
        value: field({ type: text() })
      }
    })
  }),

  // JSON field
  settings: field({
    type: json({
      schema: {
        type: 'object',
        properties: {
          theme: { type: 'string' },
          notifications: { type: 'boolean' }
        }
      }
    })
  })
}
```

### Relational Field Types

```typescript
fields: {
  // One-to-one relation
  author: field({
    type: relation({
      collection: 'users'
    })
  }),

  // One-to-many (on the owning side)
  posts: field({
    type: relation({
      collection: 'posts',
      many: true
    })
  }),

  // Many-to-many with join table
  categories: field({
    type: relation({
      collection: 'categories',
      many: true,
      through: 'postCategories'
    })
  })
}
```

### Virtual/Computed Fields

```typescript
fields: {
  firstName: field({
    type: text()
  }),

  lastName: field({
    type: text()
  }),

  // Computed field
  fullName: field({
    type: computed({
      compute: ({ firstName, lastName }) => `${firstName} ${lastName}`
    })
  }),

  // Aggregate field
  postCount: field({
    type: computed({
      dependsOn: ['posts'],
      compute: async (user) => {
        return await collections.posts.count({
          where: { authorId: user.id }
        })
      }
    })
  })
}
```

## Field Modifiers

### Required/Optional

```typescript
fields: {
  // Required field
  email: field({
    type: email(),
    required: true
  }),

  // Optional field (explicit)
  bio: field({
    type: text(),
    required: false
  })
}
```

### Default Values

```typescript
fields: {
  // Static default
  status: field({
    type: enumField({ options: ['draft', 'published'] }),
    defaultValue: 'draft'
  }),

  // Function default
  createdAt: field({
    type: timestamp(),
    defaultValue: () => new Date()
  }),

  // Dynamic default based on other fields
  slug: field({
    type: text(),
    defaultValue: ({ title }) => slugify(title)
  })
}
```

### Unique and Indexed

```typescript
fields: {
  email: field({
    type: email(),
    unique: true,
    indexed: true
  }),

  username: field({
    type: text(),
    unique: true
  })
}
```

### Validation

```typescript
fields: {
  username: field({
    type: text({ min: 3, max: 30 }),
    validate: {
      pattern: /^[a-zA-Z0-9_]+$/,
      message: 'Username must contain only letters, numbers, and underscores'
    }
  }),

  age: field({
    type: number({ integer: true }),
    validate: {
      min: 13,
      max: 120,
      custom: (value) => {
        if (value < 18) {
          throw new Error('You must be 18 or older')
        }
      }
    }
  })
}
```

## Collection Hooks

### Defining Hooks in Collection

```typescript
// collections/posts.ts
export const posts = collection({
  slug: 'posts',

  fields: {
    title: field({ type: text() }),
    slug: field({ type: text() })
  },

  hooks: {
    beforeCreate: [
      async ({ data }) => {
        // Auto-generate slug from title
        if (!data.slug) {
          data.slug = slugify(data.title)
        }
      }
    ],

    afterCreate: [
      async ({ result }) => {
        // Send notification
        await notifySubscribers(result)
      }
    ],

    beforeUpdate: [
      async ({ data, where }) => {
        // Log changes
        await logUpdate('posts', where.id, data)
      }
    ]
  }
})
```

### Multiple Hooks

```typescript
hooks: {
  beforeCreate: [
    validateTitle,
    generateSlug,
    setDefaults
  ],

  afterCreate: [
    notifySubscribers,
    updateSearchIndex,
    invalidateCache
  ]
}

// Defined as separate functions
function validateTitle({ data }) {
  if (data.title.length < 3) {
    throw new Error('Title is too short')
  }
}

function generateSlug({ data }) {
  data.slug = slugify(data.title)
}

function setDefaults({ data }) {
  data.status = data.status || 'draft'
}
```

## Collection Plugins

### Applying Plugins to Collections

```typescript
// collections/posts.ts
import { slugPlugin, timestampsPlugin, searchPlugin } from '@deessejs/collections/plugins'

export const posts = collection({
  slug: 'posts',

  fields: {
    title: field({ type: text() }),
    content: field({ type: text() })
  },

  plugins: [
    slugPlugin({ from: 'title' }),
    timestampsPlugin(),
    searchPlugin({ fields: ['title', 'content'] })
  ]
})
```

## Relationships Between Collections

### Defining Related Collections

```typescript
// collections/users.ts
export const users = collection({
  slug: 'users',
  fields: {
    name: field({ type: text() }),
    email: field({ type: email() })
  }
})

// collections/posts.ts
import { users } from './users'

export const posts = collection({
  slug: 'posts',
  fields: {
    title: field({ type: text() }),
    author: field({
      type: relation({
        collection: 'users' // or users.slug
      })
    })
  }
})
```

### Bidirectional Relationships

```typescript
// collections/users.ts
export const users = collection({
  slug: 'users',
  fields: {
    name: field({ type: text() }),
    posts: field({
      type: reverseRelation({
        collection: 'posts',
        via: 'author'
      })
    })
  }
})

// collections/posts.ts
export const posts = collection({
  slug: 'posts',
  fields: {
    title: field({ type: text() }),
    author: field({
      type: relation({
        collection: 'users'
      })
    })
  }
})
```

## Collection Metadata

### Adding Metadata

```typescript
export const posts = collection({
  slug: 'posts',

  // Human-readable label
  label: 'Blog Posts',

  // Plural label
  plural: 'Posts',

  // Description
  description: 'Blog posts and articles',

  // Custom metadata
  admin: {
    icon: 'document',
    defaultSort: 'createdAt',
    defaultSortDirection: 'desc'
  },

  fields: {
    title: field({ type: text() })
  }
})
```

## Configuration File

### Importing Collections

```typescript
// config/database.ts
import { defineConfig } from '@deessejs/collections'
import { users } from '../collections/users'
import { posts } from '../collections/posts'
import { comments } from '../collections/comments'

export const { collections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!
  },

  collections: [
    users,
    posts,
    comments
  ],

  plugins: [
    timestampsPlugin(),
    softDeletePlugin()
  ]
})
```

### Automatic Collection Discovery (Optional)

```typescript
// config/database.ts
import { defineConfig } from '@deessejs/collections'
import { importCollections } from '@deessejs/collections/utils'

// Automatically import all collections from the collections directory
const collections = await importCollections('./collections/*.ts')

export const { collections: typedCollections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!
  },
  collections
})
```

## Field Reusability

### Common Field Definitions

```typescript
// collections/fields/common.ts
import { field, text, timestamp } from '@deessejs/collections'

export const timestampFields = {
  createdAt: field({
    type: timestamp(),
    defaultValue: () => new Date()
  }),
  updatedAt: field({
    type: timestamp()
  })
}

export const slugField = (from: string) =>
  field({
    name: 'slug',
    type: text(),
    unique: true,
    defaultValue: ({ data }) => slugify(data[from])
  })

// Usage
// collections/posts.ts
export const posts = collection({
  slug: 'posts',
  fields: {
    title: field({ type: text() }),
    content: field({ type: text() }),
    ...timestampFields,
    slug: slugField('title')
  }
})
```

### Field Factories

```typescript
// collections/fields/audit.ts
import { field, timestamp, relation } from '@deessejs/collections'

export function auditFields(options: { userField?: string } = {}) {
  const { userField = 'userId' } = options

  return {
    createdAt: field({
      type: timestamp(),
      defaultValue: () => new Date()
    }),
    updatedAt: field({
      type: timestamp()
    }),
    createdBy: field({
      type: relation({
        collection: 'users'
      })
    }),
    updatedBy: field({
      type: relation({
        collection: 'users'
      })
    })
  }
}

// Usage
// collections/posts.ts
export const posts = collection({
  slug: 'posts',
  fields: {
    title: field({ type: text() }),
    ...auditFields()
  }
})

// collections/orders.ts
export const orders = collection({
  slug: 'orders',
  fields: {
    orderNumber: field({ type: text() }),
    ...auditFields({ userField: 'customerId' })
  }
})
```

## Type Inference

### Automatic Typing

```typescript
// collections/posts.ts
export const posts = collection({
  slug: 'posts',
  fields: {
    title: field({ type: text() }),
    content: field({ type: text() })
  }
})

// Type is automatically inferred:
// type Post = {
//   id: number
//   title: string
//   content: string
//   createdAt: Date
//   updatedAt: Date
// }
```

### Using Collection Types

```typescript
// app/routes/posts.ts
import { collections } from '../config/database'
import type { Post } from '../collections/posts' // Exported automatically

// Fully typed operations
const post: Post = await collections.posts.findUnique({
  where: { id: 1 }
})

// Type is inferred in queries
const posts = await collections.posts.findMany({
  select: {
    title: true, // Autocompleted
    content: true // Autocompleted
  }
})
```

## Benefits of File-Based Collections

### Organization

```typescript
collections/
  ├── users.ts
  ├── posts.ts
  ├── comments.ts
  ├── categories.ts
  └── fields/
      ├── common.ts
      └── audit.ts
```

### Reusability

```typescript
// collections/shared/admin-fields.ts
export const adminOnlyFields = {
  isAdmin: field({
    type: boolean(),
    defaultValue: false
  }),
  role: field({
    type: enumField({
      options: ['admin', 'moderator', 'user']
    }),
    defaultValue: 'user'
  })
}

// collections/users.ts
export const users = collection({
  slug: 'users',
  fields: {
    name: field({ type: text() }),
    email: field({ type: email() }),
    ...adminOnlyFields
  }
})
```

### Clear Dependencies

```typescript
// collections/posts.ts
// Clearly shows what this collection depends on
import { users } from './users'
import { categories } from './categories'

export const posts = collection({
  slug: 'posts',
  fields: {
    title: field({ type: text() }),
    author: field({ type: relation({ collection: users.slug }) }),
    category: field({ type: relation({ collection: categories.slug }) })
  }
})
```

### Easy Navigation

- Each collection in its own file
- Clear file structure mirrors domain model
- Easy to find and modify specific collections
- Better for team collaboration (less merge conflicts)

## Comparison: Inline vs File-Based

### Inline (for small apps)

```typescript
// config/database.ts
export const { collections } = defineConfig({
  database: { url: process.env.DATABASE_URL! },
  collections: [
    collection({
      slug: 'users',
      fields: { /* ... */ }
    }),
    collection({
      slug: 'posts',
      fields: { /* ... */ }
    })
  ]
})
```

### File-Based (recommended for most apps)

```typescript
// collections/users.ts
export const users = collection({
  slug: 'users',
  fields: { /* ... */ }
})

// config/database.ts
import { users } from '../collections/users'
import { posts } from '../collections/posts'

export const { collections } = defineConfig({
  database: { url: process.env.DATABASE_URL! },
  collections: [users, posts]
})
```

File-based collections are recommended when:
- You have more than 3 collections
- Multiple developers work on the codebase
- Collections have complex logic (hooks, plugins)
- You want better code organization
- You plan to scale your application
