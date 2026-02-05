# Architecture

## System Architecture

## Core Components

### Configuration Layer

The entry point for the system is the `defineConfig` function which accepts:

- **Database Configuration**: Drizzle connection details and schema information
- **Collection Definitions**: Array of collection configurations
- **Global Plugins**: Plugins that apply to all collections
- **Global Settings**: Cross-cutting configuration options

This configuration is processed and validated, then returns a runtime object containing:

- **Collections API**: Typed object with methods for each collection
- **Database Client**: Configured Drizzle instance
- **Metadata**: Collection and field information for tooling

### Collection Definition

Collections are the primary organizational unit in the system. Each collection encapsulates:

- **Fields Definition**: The structure and types of data
- **Hooks**: Lifecycle callbacks for operations
- **Plugins**: Collection-specific extensions
- **Validation Rules**: Data integrity constraints
- **Access Control**: Permissions for operations
- **Indexes and Constraints**: Database-level optimizations

Collections are composable and can reference each other through relationship fields.

### Field System

Fields are the building blocks of collections. Each field defines:

- **Type**: The underlying data type and database column type
- **Validation**: Constraints and validation rules
- **Behavior**: How the field behaves in queries and mutations
- **Metadata**: Display names, descriptions, help text
- **Hooks**: Field-specific lifecycle hooks

Fields can be primitive (text, number), structural (array, object), relational (references to other collections), or virtual (computed values).

### Query Engine

The query engine translates collection operations into Drizzle queries. It handles:

- **Query Building**: Constructing Drizzle queries from collection operations
- **Hook Execution**: Running hooks at appropriate lifecycle points
- **Relationship Loading**: Eager loading strategies and join handling
- **Transaction Management**: Wrapping operations in transactions when needed
- **Error Handling**: Converting database errors into application errors

### Plugin System

The plugin system provides extensibility through:

- **Field Extensions**: New field types that compose existing ones
- **Collection Extensions**: Modifications to collection definitions
- **Operation Extensions**: New query methods or behavior modifications
- **Global Extensions**: Cross-cutting concerns applied to all collections

Plugins are executed in a specific order during configuration, with clear rules for composition and conflict resolution.

## Data Flow

### Read Operations

When performing a find operation:

1. **Query Parsing**: The query parameters are parsed and validated
2. **Authorization**: Access control hooks check permissions
3. **Before Read Hooks**: Transform the query or add constraints
4. **Query Execution**: The Drizzle query is built and executed
5. **Result Processing**: Results are mapped and transformed
6. **After Read Hooks**: Modify or augment results
7. **Type Coercion**: Ensure results match expected types
8. **Return**: Typed results are returned to the caller

### Write Operations

When performing a create, update, or delete operation:

1. **Input Validation**: Data is validated against field constraints
2. **Before Operation Hooks**: Transform input or add metadata
3. **Validation Hooks**: Custom validation logic runs
4. **Before Database Hooks**: Last chance to modify data
5. **Query Execution**: The Drizzle mutation is executed
6. **Transaction Management**: Handle rollbacks on errors
7. **After Operation Hooks**: Side effects and notifications
8. **Result Processing**: Map database results to application types
9. **Return**: Typed results are returned to the caller

## Type System Architecture

### Type Inference Pipeline

The type system works through multiple stages:

1. **Field Type Extraction**: Each field definition contributes to the collection type
2. **Optional Handling**: Optional fields are correctly typed as optional
3. **Relation Type Resolution**: Relationship fields reference other collection types
4. **Virtual Field Exclusion**: Computed fields don't appear in database types
5. **Operation Type Derivation**: CRUD operations derive their types from the collection type
6. **Query Builder Types**: Filter, select, and include types are inferred

### Type Safety Layers

Type safety is enforced at multiple levels:

- **Configuration Time**: Invalid configurations produce type errors
- **Operation Time**: Query parameters are typed based on collection definitions
- **Result Time**: Return types accurately reflect the data structure
- **Hook Types**: Hook parameters and return values are fully typed

## Plugin Architecture

### Plugin Types

Plugins operate at different scopes:

- **Global Plugins**: Applied to all collections during configuration
- **Collection Plugins**: Applied to specific collections
- **Field Plugins**: Extend the field type system
- **Operation Plugins**: Add or modify query operations

### Plugin Composition

Multiple plugins compose in predictable ways:

- **Order of Application**: Plugins are applied in a defined order
- **Conflict Resolution**: Clear rules for when plugins modify the same thing
- **Dependency Management**: Plugins can declare dependencies on other plugins
- **Isolation**: Plugins cannot break other plugins accidentally

### Plugin Hooks

Plugins can hook into:

- **Configuration Phase**: Modify collection or field definitions
- **Query Phase**: Intercept or transform queries
- **Result Phase**: Modify query results
- **Validation Phase**: Add custom validation logic

## Integration Architecture

### Drizzle Integration

The integration with Drizzle maintains:

- **Schema Compatibility**: Collection definitions map to Drizzle schemas
- **Query Compatibility**: Can execute raw Drizzle queries when needed
- **Type Compatibility**: Drizzle types work seamlessly with collection types
- **Migration Compatibility**: Use DrizzleKit for schema migrations

### Framework Integration

While framework-agnostic, the system provides integration patterns for:

- **HTTP Frameworks**: Route generation for Hono, Express, Fastify
- **Validation Libraries**: Integration with Zod, Yup, or custom validators
- **Logging**: Structured logging integration
- **Monitoring**: Performance monitoring and metrics collection

## Performance Architecture

### Lazy Initialization

Collections and their metadata are initialized lazily to minimize startup cost.

### Query Optimization

The query engine provides:

- **Select Optimization**: Only query requested fields
- **Join Optimization**: Use efficient join strategies
- **N+1 Prevention**: Batch relationship loading when possible
- **Query Caching**: Cache frequently executed queries (opt-in)

### Memory Management

The system avoids memory leaks through:

- **No Global State**: Configuration is passed explicitly
- **Disposable Connections**: Database connections are properly managed
- **Weak References**: Caches use weak references when appropriate
