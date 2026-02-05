# Field Type System

## Overview

The field type system is built around the `fieldType` function, which allows anyone to create custom field types. All built-in field types (text, number, email, etc.) are created using this same function, meaning you have the same power as the framework authors.

## What is a Field Type?

A `fieldType` combines:
- **Zod Schema**: For runtime validation and TypeScript type inference
- **Drizzle Column**: For database schema mapping
- **Configuration Function**: Returns a configurable field definition

## Creating Field Types

### Basic Structure

```typescript
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText, pgInteger, pgTimestamp } from 'drizzle-orm/pg-core'

// fieldType takes an object with schema and database properties
// Returns a function that accepts configuration options
const myFieldType = fieldType({
  schema: z.string().min(3).max(255),  // Zod schema for validation
  database: pgText()                     // Drizzle column for DB mapping
})
```

### Using Field Types

```typescript
// collections/posts.ts
import { collection, field } from '@deessejs/collections'
import { text, email } from '@deessejs/collections/fields'

export const posts = collection({
  slug: 'posts',

  fields: {
    // Simple field with default options
    title: text(),

    // Field with custom options
    slug: text({ unique: true }),

    // Email field with validation
    authorEmail: email({ unique: true })
  }
})
```

## Built-in Field Types

All built-in field types follow this pattern:

### Text Field Type

```typescript
// fields/text.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText } from 'drizzle-orm/pg-core'

export const text = fieldType({
  // Base Zod schema
  schema: z.string(),

  // Drizzle column
  database: pgText(),

  // Configuration options (optional)
  config: {
    // These options can modify the Zod schema and Drizzle column
    min: (value) => ({
      schema: z.string().min(value),
      message: `Must be at least ${value} characters`
    }),
    max: (value) => ({
      schema: z.string().max(value)
    }),
    unique: () => ({
      database: pgText().unique()
    }),
    indexed: () => ({
      database: pgText().index()
    })
  }
})

// Usage
title: text()                                              // Simple text
slug: text({ unique: true })                               // Unique text
name: text({ min: 2, max: 100 })                           // With length constraints
email: text({ unique: true, indexed: true })                // Unique + indexed
```

### Number Field Type

```typescript
// fields/number.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgInteger, pgNumeric } from 'drizzle-orm/pg-core'

export const number = fieldType({
  schema: z.number(),
  database: pgInteger(), // Default: integer

  config: {
    // Option to use decimal instead of integer
    decimal: () => ({
      database: pgNumeric(),
      schema: z.number().or(z.string()) // Accept string for decimals
    }),

    min: (value) => ({
      schema: z.number().min(value)
    }),

    max: (value) => ({
      schema: z.number().max(value)
    }),

    integer: () => ({
      schema: z.number().int(),
      database: pgInteger()
    })
  }
})

// Usage
age: number()                              // Integer
price: number({ decimal: true })           // Decimal
rating: number({ min: 1, max: 5 })         // Range
quantity: number({ integer: true, min: 0 }) // Positive integer
```

### Email Field Type

```typescript
// fields/email.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText } from 'drizzle-orm/pg-core'

export const email = fieldType({
  schema: z.string().email(),
  database: pgText()
})

// Usage
email: email()
workEmail: email({ unique: true })
```

### Boolean Field Type

```typescript
// fields/boolean.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgBoolean } from 'drizzle-orm/pg-core'

export const boolean = fieldType({
  schema: z.boolean(),
  database: pgBoolean()
})

// Usage
isActive: boolean()
isAdmin: boolean({ default: false })
```

### Date/Timestamp Field Types

```typescript
// fields/date.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgDate, pgTimestamp } from 'drizzle-orm/pg-core'

export const date = fieldType({
  schema: z.date(),
  database: pgDate()
})

export const timestamp = fieldType({
  schema: z.date(),
  database: pgTimestamp(),

  config: {
    defaultNow: () => ({
      schema: z.date(),
      database: pgTimestamp().defaultNow()
    })
  }
})

// Usage
birthDate: date()
createdAt: timestamp()
updatedAt: timestamp({ defaultNow: true })
```

### Enum Field Type

```typescript
// fields/enum.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText } from 'drizzle-orm/pg-core'

export const enumField = <T extends readonly [string, ...string[]]>(
  options: T
) => fieldType({
  schema: z.enum(options),
  database: pgText()
})

// Usage - Fully typed!
status: enumField(['draft', 'published', 'archived'])
role: enumField(['user', 'admin', 'moderator'])
```

## Creating Custom Field Types

### Simple Custom Field Type

```typescript
// fields/slug.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText } from 'drizzle-orm/pg-core'

export const slug = fieldType({
  schema: z.string()
    .regex(/^[a-z0-9-]+$/, 'Must contain only lowercase letters, numbers, and hyphens')
    .min(3, 'Must be at least 3 characters')
    .max(100, 'Must be at most 100 characters'),

  database: pgText(),

  config: {
    unique: () => ({
      database: pgText().unique()
    })
  }
})

// Usage
slug: slug()
handle: slug({ unique: true })
```

### URL Field Type

```typescript
// fields/url.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText } from 'drizzle-orm/pg-core'

export const url = fieldType({
  schema: z.string().url(),
  database: pgText()
})

// Usage
website: url()
avatar: url()
```

### Phone Number Field Type

```typescript
// fields/phone.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText } from 'drizzle-orm/pg-core'

const phoneRegex = /^\+?[\d\s-()]+$/

export const phone = fieldType({
  schema: z.string().regex(phoneRegex, 'Invalid phone number format'),
  database: pgText()
})

// Usage
phoneNumber: phone()
mobile: phone()
```

## Complex Field Types

### Array Field Type

```typescript
// fields/array.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgArray, pgText } from 'drizzle-orm/pg-core'

export const array = <T extends z.ZodType>(
  itemType: T,
  drizzleColumn = pgText()
) => fieldType({
  schema: z.array(itemType),
  database: pgArray(itemType, drizzleColumn)
})

// Usage
tags: array(z.string())
categories: array(z.enum(['tech', 'design', 'business']))
scores: array(z.number().int().min(0).max(100))
```

### JSON Field Type

```typescript
// fields/json.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgJsonb } from 'drizzle-orm/pg-core'

export const json = <T extends z.ZodType = z.ZodAny>(
  schema?: T
) => fieldType({
  schema: schema || z.any(),
  database: pgJsonb()
})

// Usage
metadata: json()
settings: json(z.object({
  theme: z.enum(['light', 'dark']),
  notifications: z.boolean()
}))
```

### Object Field Type

```typescript
// fields/object.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgJsonb } from 'drizzle-orm/pg-core'

export const object = <T extends z.ZodType>(
  schema: T
) => fieldType({
  schema: schema,
  database: pgJsonb()
})

// Usage
metadata: object(z.object({
  key: z.string(),
  value: z.string()
}))

address: object(z.object({
  street: z.string(),
  city: z.string(),
  zipCode: z.string().regex(/^\d{5}$/)
}))
```

## Relation Field Type

### Basic Relation

```typescript
// fields/relation.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgInteger } from 'drizzle-orm/pg-core'

export const relation = (collectionName: string) => fieldType({
  schema: z.number(), // Foreign key ID
  database: pgInteger().references(`.${collectionName}.id`)
})

// Usage
authorId: relation('users')
postId: relation('posts')
```

### Many-to-Many Relation

```typescript
// fields/manyToMany.ts
import { field } from '@deessejs/collections'
import { pgInteger, pgTable, serial, primaryKey } from 'drizzle-orm/pg-core'

// This helper creates the join table automatically
export const manyToMany = (
  collectionA: string,
  collectionB: string
) => {
  const joinTableName = `${collectionA}_${collectionB}`

  // Create the join table
  const joinTable = pgTable(joinTableName, {
    [`${collectionA}Id`]: pgInteger(`${collectionA}_id`)
      .references(`${collectionA}.id`)
      .notNull(),
    [`${collectionB}Id`]: pgInteger(`${collectionB}_id`)
      .references(`${collectionB}.id`)
      .notNull()
  }, (table) => ({
    pk: primaryKey({ columns: [table[`${collectionA}Id`], table[`${collectionB}Id`]] })
  }))

  return {
    // Field for the first collection
    fieldA: field({
      type: relation(collectionB, { many: true, through: joinTableName })
    }),
    // Field for the second collection
    fieldB: field({
      type: relation(collectionA, { many: true, through: joinTableName })
    }),
    // The join table for schema export
    joinTable
  }
}

// Usage
const { fieldA: categories, fieldB: posts, joinTable } = manyToMany('posts', 'categories')

// In posts collection
categories: categories

// In categories collection
posts: posts

// Export the join table in schema
export const postCategories = joinTable
```

## Composing Field Types

### Wrapping Existing Field Types

```typescript
// fields/verified-email.ts
import { email } from './email'
import { z } from 'zod'

export const verifiedEmail = email.extend({
  // Add additional validation
  verified: () => ({
    zod: z.string().email().refine(
      async (email) => await checkEmailVerified(email),
      'Email must be verified'
    )
  })
})

// Or compose from scratch
export const verifiedEmail = fieldType({
  schema: z.string().email().refine(
    async (email) => await checkEmailVerified(email),
    'Email must be verified'
  ),
  database: pgText()
})

// Usage
email: verifiedEmail()
```

### Adding Default Values

```typescript
// fields/with-default.ts
import { text } from './text'

export const autoSlug = (fromField: string) =>
  text.extend({
    default: () => ({
      transform: (data) => slugify(data[fromField])
    })
  })

// Usage
title: text(),
slug: autoSlug('title')({ unique: true })
```

## Parameterized Field Types

### Range Field Type

```typescript
// fields/range.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgNumeric } from 'drizzle-orm/pg-core'

export const range = (min: number, max: number, step = 1) => fieldType({
  schema: z.number()
    .min(min, `Must be at least ${min}`)
    .max(max, `Must be at most ${max}`)
    .refine(
      (n) => n % step === 0,
      `Must be a multiple of ${step}`
    ),
  database: pgNumeric()
})

// Usage
price: range(0, 1000, 0.01)    // Price between 0-1000, increments of 0.01
quantity: range(1, 100, 1)      // Quantity 1-100, whole numbers
rating: range(1, 5, 0.5)        // Rating 1-5, increments of 0.5
```

### Country Field Type

```typescript
// fields/country.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText } from 'drizzle-orm/pg-core'
import { countryList } from './countries'

export const country = fieldType({
  schema: z.enum(countryList.map(c => c.code) as [string, ...string[]]),
  database: pgText()
})

// Usage
country: country()
billingCountry: country({ default: 'US' })
```

## Field Type Options

### Available Options

All field types support these options:

```typescript
// Modify the Zod schema
myField({
  required: true,           // Add .nonempty() or similar
  default: defaultValue,     // Add .default()
  transform: fn,            // Add .transform()
  refine: fn,               // Add .refine()
})

// Modify the Drizzle column
myField({
  unique: true,             // Add .unique()
  indexed: true,            // Add .index()
  notNull: true,            // Add .notNull()
  defaultNow: true,         // Add .defaultNow()
})

// Field metadata
myField({
  label: 'Display Name',
  description: 'Field description',
  hidden: false,
  readOnly: false
})
```

### Combining Options

```typescript
// Simple field
username: text()

// With options
email: email({
  unique: true,
  indexed: true,
  label: 'Email Address',
  description: 'We will never share your email'
})

// Complex field
password: text({
  required: true,
  min: 8,
  refine: (value) => {
    if (!/[A-Z]/.test(value)) {
      throw new Error('Must contain at least one uppercase letter')
    }
    if (!/[0-9]/.test(value)) {
      throw new Error('Must contain at least one number')
    }
    return value
  }
})
```

## Publishing Custom Field Types

### Creating a Field Type Package

```typescript
// packages/my-fields/src/index.ts
import { fieldType } from '@deessejs/collections'
import { z } from 'zod'
import { pgText } from 'drizzle-orm/pg-core'

// Phone field with country-specific validation
export const phone = (country: string = 'US') => fieldType({
  schema: z.string().regex(getPhonePattern(country), `Invalid ${country} phone number`),
  database: pgText()
})

// URL slug with custom separator
export const urlSlug = (separator: string = '-') => fieldType({
  schema: z.string().regex(
    new RegExp(`^[a-z0-9${separator}]+$`),
    `Must contain only lowercase letters, numbers, and ${separator}`
  ),
  database: pgText(),

  config: {
    unique: () => ({ database: pgText().unique() })
  }
})

// Money field with currency
export const money = (currency: string = 'USD', precision: number = 2) => fieldType({
  schema: z.number().positive(),
  database: pgNumeric({ precision: 10, scale: precision })
})

export * from '@deessejs/collections/fields' // Re-export built-ins
```

### Using Custom Field Type Packages

```typescript
// collections/users.ts
import { phone, urlSlug, money } from '@my-org/custom-fields'

export const users = collection({
  slug: 'users',

  fields: {
    phoneNumber: phone('FR'),
    handle: urlSlug('_'),
    balance: money('EUR', 2)
  }
})
```

## Type Inference

### Automatic Type Inference

```typescript
// All field types automatically infer TypeScript types from their Zod schema
export const posts = collection({
  slug: 'posts',

  fields: {
    title: text(),              // string
    views: number(),            // number
    published: boolean(),       // boolean
    status: enumField(['draft', 'published', 'archived']) // 'draft' | 'published' | 'archived'
  }
})

// TypeScript automatically infers:
// type Post = {
//   id: number
//   title: string
//   views: number
//   published: boolean
//   status: 'draft' | 'published' | 'archived'
//   createdAt: Date
//   updatedAt: Date
// }
```

### Complex Type Inference

```typescript
export const users = collection({
  slug: 'users',

  fields: {
    name: text(),
    metadata: json(z.object({
      theme: z.enum(['light', 'dark']),
      notifications: z.boolean()
    })), // { theme: 'light' | 'dark', notifications: boolean }
    tags: array(z.string()), // string[]
    settings: object(z.object({
      key: z.string(),
      value: z.number()
    })) // { key: string, value: number }
  }
})

// All types are automatically inferred!
```

## Benefits

### Simplicity

Field types are just Zod + Drizzle:
- No complex validation logic to learn
- Leverage existing Zod ecosystem
- Direct mapping to database columns

### Type Safety

Full TypeScript support:
- Zod provides runtime validation
- Zod provides compile-time types
- Drizzle provides database schema

### Extensibility

Anyone can create field types:
- Same power as built-in types
- Published as npm packages
- Shared across teams

### Composability

Field types compose naturally:
- Wrap existing types
- Combine validations
- Share schemas
