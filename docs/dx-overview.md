# Developer Experience: Core Concepts

## Introduction

The developer experience of `@deessejs/collections` is built around three fundamental concepts that work together: **Configuration**, **Collections**, and **Fields**. Understanding how these pieces interact is key to being productive with the library.

## Configuration

### The Single Entry Point

All applications start with a single configuration file that defines the entire data layer. This configuration is not just a settings file—it's a living part of your application that returns a fully-typed API object.

```typescript
// config/database.ts
import { defineConfig, collection, field } from '@deessejs/collections'
import { drizzle } from 'drizzle-orm/postgres-js'

export const { collections, db } = defineConfig({
  database: {
    url: process.env.DATABASE_URL!,
    schema: './schema.ts' // Optional: use existing Drizzle schema
  },

  collections: [
    // Your collection definitions go here
    collection('users', {
      fields: [
        field.text('name'),
        field.email('email').unique(),
        field.date('createdAt').default(() => new Date())
      ]
    })
  ],

  plugins: [
    // Global plugins applied to all collections
    timestampsPlugin(),
    softDeletePlugin()
  ]
})
```

### Configuration as Code

Configuration is written in TypeScript, not JSON or YAML. This means:

- **Autocomplete**: Your IDE helps you discover available options
- **Type Safety**: Mistakes are caught before you run your code
- **Refactoring**: Rename collections or fields safely across your entire codebase
- **Documentation**: Hover over any configuration to see JSDoc explanations
- **Import Anywhere**: Use shared utilities, constants, or helper functions in your config

```typescript
// You can import and use anything in your config
import { slugify } from './utils/string'
import { VALID_EMAIL_REGEX } from './utils/validation'

collection('users', {
  fields: [
    field.text('slug').default(({ name }) => slugify(name)),
    field.email('email', {
      validation: {
        regex: VALID_EMAIL_REGEX
      }
    })
  ]
})
```

### The Return Value

The `defineConfig` function returns a runtime object that becomes your primary interface to the data layer:

```typescript
const { collections, db } = defineConfig({...})

// collections is fully typed - TypeScript knows about 'users'
collections.users.findMany() // ✅ Autocomplete works!

// db is the underlying Drizzle instance for when you need it
import { users } from './schema'
await db.select().from(users) // Direct Drizzle access
```

This object:
- **Is Fully Typed**: All operations benefit from complete type information
- **Has Your Collections**: Each collection becomes a property with typed methods
- **Includes the Database**: Access to the underlying Drizzle instance when needed
- **Provides Metadata**: Information about your schema for tooling and introspection

### Using the Configuration

```typescript
// Anywhere in your app
import { collections } from './config/database'

// Fully typed access to your data
const users = await collections.users.findMany({
  where: {
    email: { contains: '@example.com' }
  }
})
```

## Collections

### Collections as Domain Aggregates

Collections represent coherent domain concepts—like users, posts, or orders. Each collection encapsulates everything about that concept:

```typescript
collection('posts', {
  // What data it contains
  fields: [
    field.text('title'),
    field.richText('content'),
    field.relation('author', { collection: 'users' })
  ],

  // How it behaves
  hooks: {
    beforeCreate: async ({ data }) => {
      // Auto-generate slug from title
      data.slug = slugify(data.title)
    }
  },

  // How it relates to other collections
  // (defined in fields via relation())

  // What extensions it has
  plugins: [
    searchPlugin({ fields: ['title', 'content'] })
  ]
})
```

### Collection Identity

The collection name becomes a property on the returned collections object:

```typescript
// Collection name: 'posts'
collection('posts', { ... })

// Accessible as:
collections.posts.findMany()
collections.posts.create({ data: { ... } })
collections.posts.findUnique({ where: { id: 1 } })
```

### Collection Composition

Collections compose from smaller pieces:

```typescript
collection('products', {
  // Fields define the structure
  fields: [
    field.text('name'),
    field.number('price'),
    field.relation('category', { collection: 'categories' })
  ],

  // Hooks define behavior
  hooks: {
    beforeUpdate: [
      validatePriceChange,
      logPriceChange
    ],
    afterUpdate: [
      updateSearchIndex,
      notifyPriceChange
    ]
  },

  // Plugins add extensions
  plugins: [
    slugPlugin({ from: 'name' }),
    timestampsPlugin()
  ],

  // Validation rules
  validation: {
    // Cross-field validation
    price: {
      min: ({ cost }) => cost * 1.2 // Price must be 20% above cost
    }
  }
})
```

### Collection Operations

After defining a collection, you automatically get typed operations:

```typescript
// Finding data
const posts = await collections.posts.findMany({
  where: {
    status: { equals: 'published' },
    createdAt: { gte: new Date('2024-01-01') }
  },
  orderBy: { createdAt: 'desc' },
  limit: 10
})

// Finding a single record
const post = await collections.posts.findUnique({
  where: { id: 1 }
})

// Creating data
const newPost = await collections.posts.create({
  data: {
    title: 'Hello World',
    content: 'This is my first post',
    authorId: 1
  }
})

// Updating data
const updatedPost = await collections.posts.update({
  where: { id: 1 },
  data: {
    title: 'Updated Title'
  }
})

// Deleting data
await collections.posts.delete({
  where: { id: 1 }
})

// Counting
const count = await collections.posts.count({
  where: { status: { equals: 'published' } }
})
```

### Collection Isolation

Each collection is independent:

```typescript
// collections.users doesn't know about collections.posts
// They can't accidentally affect each other

// Both work independently
const users = await collections.users.findMany()
const posts = await collections.posts.findMany()

// No shared state, no global configuration
```

## Fields

### Fields as Building Blocks

Fields are the atomic units of your data model:

```typescript
field.text('name') // Simple text field
field.number('age') // Simple number field
field.date('birthDate') // Simple date field
```

Each field defines:
- **Its Type**: What kind of data it stores
- **Its Constraints**: Validation rules
- **Its Behavior**: How it behaves in queries
- **Its Metadata**: Labels and descriptions

### Primitive Fields

```typescript
collection('users', {
  fields: [
    // Text fields
    field.text('firstName'),
    field.text('lastName'),
    field.email('email'), // Specialized text field with email validation
    field.url('website'), // Specialized text field with URL validation

    // Number fields
    field.number('age'),
    field.number('salary', { decimal: true }),

    // Boolean fields
    field.boolean('isActive'),
    field.boolean('isAdmin').default(false),

    // Date fields
    field.date('birthDate'),
    field.timestamp('createdAt'),

    // Enum fields
    field.enum('role', {
      options: ['user', 'admin', 'moderator']
    })
  ]
})
```

### Structural Fields

```typescript
collection('posts', {
  fields: [
    // Array fields
    field.array('tags', {
      of: field.text('tag')
    }),

    field.array('ratings', {
      of: field.number('rating', { min: 1, max: 5 })
    }),

    // Object fields (nested structures)
    field.object('metadata', {
      fields: [
        field.text('key'),
        field.text('value')
      ]
    }),

    // JSON fields (flexible data)
    field.json('settings', {
      schema: {
        type: 'object',
        properties: {
          theme: { type: 'string' },
          notifications: { type: 'boolean' }
        }
      }
    })
  ]
})
```

### Relational Fields

```typescript
collection('posts', {
  fields: [
    // One-to-one relation
    field.relation('author', {
      collection: 'users',
      singular: true
    }),

    // One-to-many (from the other side)
    // This field is virtual - it doesn't exist in the database
    field.reverseRelation('posts', {
      collection: 'users',
      via: 'author'
    }),

    // Many-to-many
    field.relation('categories', {
      collection: 'categories',
      many: true,
      through: 'postCategories' // Join table
    })
  ]
})
```

### Virtual/Computed Fields

```typescript
collection('users', {
  fields: [
    field.text('firstName'),
    field.text('lastName'),

    // Virtual field - computed from other fields
    field.virtual('fullName', {
      compute: ({ firstName, lastName }) => `${firstName} ${lastName}`
    }),

    field.relation('posts', {
      collection: 'posts',
      many: true
    }),

    // Aggregate field - computed from related records
    field.aggregate('postCount', {
      collection: 'posts',
      via: 'author',
      compute: 'count'
    })
  ]
})

// Usage in queries
const users = await collections.users.findMany({
  select: {
    fullName: true, // Virtual field works like any other field
    postCount: true
  }
})
```

### Field Modifiers

Fields can be modified with helper functions:

```typescript
collection('users', {
  fields: [
    // Chaining modifiers
    field.text('email')
      .required() // Cannot be null
      .unique() // Must be unique in the database
      .indexed(), // Add database index for performance

    field.text('username')
      .unique()
      .minLength(3)
      .maxLength(30)
      .pattern(/^[a-zA-Z0-9_]+$/), // Custom regex pattern

    field.number('age')
      .min(13)
      .max(120)
      .default(18), // Default value

    field.boolean('isActive')
      .default(true) // Default value can be a function
      .indexed()
  ]
})
```

### Field Reusability

Extract and reuse field definitions:

```typescript
// fields/common.ts
export const timestampFields = [
  field.timestamp('createdAt').default(() => new Date()),
  field.timestamp('updatedAt')
]

export const slugField = (from: string) =>
  field.text('slug').unique().default(({ data }) => slugify(data[from]))

// Usage
collection('posts', {
  fields: [
    field.text('title'),
    slugField('title'),
    ...timestampFields
  ]
})

collection('categories', {
  fields: [
    field.text('name'),
    slugField('name'),
    ...timestampFields
  ]
})
```

### Field Factories

Create parameterized field templates:

```typescript
// fields/audit.ts
export function auditFields(userIdField: string = 'userId') {
  return [
    field.timestamp('createdAt').default(() => new Date()),
    field.timestamp('updatedAt'),
    field.relation('createdBy', {
      collection: 'users',
      singular: true
    }),
    field.relation('updatedBy', {
      collection: 'users',
      singular: true
    })
  ]
}

// Usage
collection('posts', {
  fields: [
    field.text('title'),
    ...auditFields() // Uses defaults
  ]
})

collection('orders', {
  fields: [
    field.text('orderNumber'),
    ...auditFields('customerId') // Custom user field
  ]
})
```

## How They Work Together

### Configuration Contains Collections

```typescript
defineConfig({
  collections: [
    collection('users', { ... }),
    collection('posts', { ... }),
    collection('comments', { ... })
  ]
})
```

### Collections Contain Fields

```typescript
collection('posts', {
  fields: [
    field.text('title'),
    field.text('content'),
    field.relation('author', { collection: 'users' })
  ]
})
```

### Fields Generate Types

```typescript
// The fields above automatically generate this type:
type Post = {
  id: number
  title: string
  content: string
  authorId: number
  author?: User // From the relation
  createdAt: Date
  updatedAt: Date
}

// And query types:
collections.posts.findMany({
  select: {
    title: true, // ✅ Autocomplete knows these fields
    content: true,
    author: { // Relation loading
      select: {
        name: true // Nested autocomplete
      }
    }
  }
})
```

### The Result: A Typed API

```typescript
// Everything is fully typed - no manual type annotations needed!

const post = await collections.posts.findUnique({
  where: { id: 1 },
  include: { author: true }
})

// TypeScript knows:
// - post is of type Post | null
// - post.title is a string
// - post.author is of type User | undefined
// - All properties are autocompleted

if (post) {
  console.log(post.title) // ✅ Type-safe!
  console.log(post.author?.name) // ✅ Type-safe!
}
```

## Developer Experience Benefits

### Zero Configuration Type Safety

You rarely write type annotations manually:

```typescript
// ❌ Not needed - types are inferred!
// interface Post {
//   title: string
//   content: string
// }

// ✅ Just define the collection, get types for free
collection('posts', {
  fields: [
    field.text('title'),
    field.text('content')
  ]
})
```

### Excellent Autocomplete

Your IDE knows everything about your data model:

```typescript
collections.posts.f // 💡 Autocomplete shows:
// - findMany
// - findUnique
// - findFirst
// - create
// - update
// - delete
// - count

collections.posts.findMany({
  where: {
    // 💡 Autocomplete shows all fields:
    // - title
    // - content
    // - author
    // - createdAt
    title: {
      // 💡 Autocomplete shows all operators:
      // - equals
      // - contains
      // - startsWith
      // - in
      // - etc.
    }
  }
})
```

### Refactoring Confidence

Rename a field, and TypeScript tells you every place that needs updating:

```typescript
// Rename 'title' to 'name' in the collection definition
field.text('name') // Was 'title'

// TypeScript now shows errors everywhere 'title' is used:
collections.posts.findMany({
  where: {
    // ❌ Error: Property 'title' does not exist
    // 💡 Quick fix: Rename to 'name'
    title: { contains: 'hello' }
  }
})
```

### Clear Error Messages

Validation errors are specific and helpful:

```typescript
// Runtime validation error:
// ❌ ValidationError in collection 'posts', field 'title':
//    Value is too short. Minimum length is 10, got 3.

// Type error at compile time:
// ❌ Type 'number' is not assignable to type 'string'.
//    Field 'title' expects a string.
```

### Progressive Complexity

Start simple, add complexity as needed:

```typescript
// Step 1: Simple collection
collection('posts', {
  fields: [
    field.text('title'),
    field.text('content')
  ]
})

// Step 2: Add validation
collection('posts', {
  fields: [
    field.text('title').minLength(10).required(),
    field.text('content').minLength(50)
  ]
})

// Step 3: Add hooks
collection('posts', {
  fields: [
    field.text('title').minLength(10),
    field.text('content')
  ],
  hooks: {
    beforeCreate: [{ data: { title } => {
      data.slug = slugify(title)
    }}]
  }
})

// Step 4: Add plugins
collection('posts', {
  fields: [
    field.text('title'),
    field.text('content')
  ],
  plugins: [
    slugPlugin(),
    searchPlugin(),
    timestampsPlugin()
  ]
})
```
