# Features

## Core Features

### Collection Definition System

Define data models using a declarative collection function that encapsulates fields, behaviors, and relationships. Collections serve as the central unit of organization in your application's data layer.

### Rich Field System

A comprehensive field system supporting various data types beyond basic database columns:

- **Primitive Fields**: Text, number, boolean, date fields with built-in validation
- **Structural Fields**: Arrays, objects, and JSON fields for nested data
- **Relational Fields**: One-to-one, one-to-many, and many-to-many relationships
- **Computed Fields**: Virtual fields derived from other data
- **Custom Fields**: Extensible field type system for domain-specific needs

### Dynamic Field Generation

Fields can be generated programmatically based on context, configuration, or other fields. This enables patterns like:

- Auto-generated slug fields from title fields
- Timestamp fields automatically added to collections
- Fields that modify themselves based on collection configuration
- Computed fields that aggregate data from relationships

### Type-Safe Query API

A fluent query interface that provides full type safety and autocomplete:

- Find operations with filtering, sorting, and pagination
- Create, update, and delete operations with hooks support
- Relationship loading and eager loading strategies
- Transaction support for complex operations
- Automatic type inference from collection definitions

### Hooks System

Lifecycle hooks that intercept operations at specific points:

- **Operation Hooks**: Before/after read, create, update, delete operations
- **Validation Hooks**: Custom validation logic before persistence
- **Transformation Hooks**: Modify data on read or write
- **Async Hooks**: Support for async operations in hooks

### Plugin Architecture

An extensible plugin system that allows cross-cutting concerns:

- **Collection Plugins**: Add fields and behaviors to specific collections
- **Field Plugins**: Extend the available field types
- **Global Plugins**: Apply transformations to all collections
- **Operation Plugins**: Add new query operations or modify existing ones
- **Plugin Composition**: Combine multiple plugins with predictable behavior

## Advanced Features

### Relationship Management

Sophisticated relationship handling beyond basic foreign keys:

- Bidirectional relationship synchronization
- Cascade operations (delete, update)
- Relationship validation and constraints
- Polymorphic relationships
- Through tables for many-to-many with metadata

### Validation System

Multi-layer validation approach:

- Field-level validation constraints
- Cross-field validation
- Custom validation functions
- Async validation (database uniqueness, API calls)
- Validation error collection and reporting

### Migration Integration

Seamless integration with DrizzleKit for schema management:

- Automatic schema generation from collection definitions
- Migration generation for collection changes
- Schema diffing and validation
- Rollback support

### Performance Optimization

Built-in performance features:

- Query optimization hints
- Selective field loading
- N+1 query prevention through proper relationship loading
- Caching hooks for frequently accessed data
- Database query logging and analysis

### Extensibility Points

Multiple ways to extend the framework:

- Custom field types with full editor support
- Custom operations on collections
- Virtual collections (views, aggregated data)
- Custom middleware in the query pipeline
- Integration points for external services

## Developer Experience

### TypeScript Integration

First-class TypeScript support throughout:

- Full type inference from collection definitions
- Autocomplete for all operations
- Type-safe query builders
- Generic type parameters for advanced scenarios
- Strict mode compatibility

### Error Handling

Comprehensive error types and handling:

- Specific error types for different failure modes
- Error context and suggestions
- Hook error handling strategies
- Transaction rollback on errors

### Documentation and Tooling

Support for development workflow:

- JSDoc comments for autocomplete documentation
- Collection introspection and metadata
- Development mode with additional validation
- Debugging hooks and logging
