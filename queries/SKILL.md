---
name: queries
description: Build and execute semantic queries with filters, time dimensions, aggregations, and ordering in drizzle-cube. Use when querying analytics data, filtering results, working with date ranges, aggregating measures, or building reports with drizzle-cube.
---

# Drizzle Cube Queries

This skill helps you build and execute semantic queries using Drizzle Cube's query API. Queries combine measures, dimensions, filters, and time dimensions to retrieve analytics data.

## Core Concept

A **query** in Drizzle Cube specifies:
- **Measures** - What to aggregate (counts, sums, averages, etc.)
- **Dimensions** - How to group/categorize the results
- **TimeDimensions** - Time-based filtering and grouping
- **Filters** - Conditions to filter the data
- **Order** - Sorting of results
- **Limit/Offset** - Pagination

## Basic Query Structure

```typescript
const query = {
  measures: ['CubeName.measureName'],
  dimensions: ['CubeName.dimensionName'],
  timeDimensions: [{
    dimension: 'CubeName.timeDimension',
    granularity: 'day',
    dateRange: ['2024-01-01', '2024-12-31']
  }],
  filters: [{
    member: 'CubeName.field',
    operator: 'equals',
    values: ['value']
  }],
  order: { 'CubeName.field': 'asc' },
  limit: 100,
  offset: 0
}

// Execute the query
const result = await semanticLayer.execute(query, securityContext)
```

## Measures

Measures are aggregated values calculated across your data.

### Single Measure

```typescript
const query = {
  measures: ['Employees.count']
}

// Result: Total employee count
```

### Multiple Measures

```typescript
const query = {
  measures: [
    'Employees.count',
    'Employees.avgSalary',
    'Employees.totalSalary'
  ]
}

// Result: Multiple aggregations in one query
```

### Cross-Cube Measures

```typescript
const query = {
  measures: [
    'Employees.count',
    'Departments.count',
    'Projects.count'
  ]
}

// Result: Aggregations from multiple cubes (automatic JOINs)
```

## Dimensions

Dimensions categorize and group your data.

### Single Dimension

```typescript
const query = {
  measures: ['Employees.count'],
  dimensions: ['Employees.department']
}

// Result: Employee count grouped by department
```

### Multiple Dimensions

```typescript
const query = {
  measures: ['Employees.count'],
  dimensions: [
    'Employees.department',
    'Employees.isActive'
  ]
}

// Result: Employee count grouped by department and active status
```

### Cross-Cube Dimensions

```typescript
const query = {
  measures: ['Employees.count'],
  dimensions: [
    'Employees.name',
    'Departments.name' // Dimension from joined cube
  ]
}

// Result: Uses automatic JOIN to include department name
```

## Time Dimensions

Time dimensions enable temporal filtering and grouping.

### Basic Time Dimension

```typescript
const query = {
  measures: ['Employees.count'],
  timeDimensions: [{
    dimension: 'Employees.createdAt',
    granularity: 'day' // Group by day
  }]
}
```

### Time Granularities

```typescript
// Supported granularities:
const granularities = [
  'second',
  'minute',
  'hour',
  'day',
  'week',
  'month',
  'quarter',
  'year'
]

// Example: Monthly grouping
const query = {
  measures: ['Employees.count'],
  timeDimensions: [{
    dimension: 'Employees.createdAt',
    granularity: 'month'
  }]
}
```

### Date Range Filtering

```typescript
// Absolute date range
const query = {
  measures: ['Employees.count'],
  timeDimensions: [{
    dimension: 'Employees.createdAt',
    dateRange: ['2024-01-01', '2024-12-31']
  }]
}

// Relative date range
const query2 = {
  measures: ['Employees.count'],
  timeDimensions: [{
    dimension: 'Employees.createdAt',
    dateRange: 'last 7 days'
  }]
}

// Supported relative ranges:
const relativeRanges = [
  'today',
  'yesterday',
  'this week',
  'this month',
  'this quarter',
  'this year',
  'last 7 days',
  'last 30 days',
  'last week',
  'last month',
  'last quarter',
  'last year'
]
```

### Time Dimension with Granularity and Range

```typescript
const query = {
  measures: ['Employees.count'],
  timeDimensions: [{
    dimension: 'Employees.createdAt',
    granularity: 'week',
    dateRange: 'last 90 days'
  }]
}

// Result: Weekly employee counts for the last 90 days
```

## Filters

Filters restrict which data is included in the query.

### Simple Filters

#### Equals Filter

```typescript
const query = {
  measures: ['Employees.count'],
  filters: [{
    member: 'Employees.department',
    operator: 'equals',
    values: ['Engineering']
  }]
}
```

#### Multiple Values (IN clause)

```typescript
const query = {
  measures: ['Employees.count'],
  filters: [{
    member: 'Employees.department',
    operator: 'equals',
    values: ['Engineering', 'Sales', 'Marketing']
  }]
}
```

### String Operators

```typescript
// Contains
{
  member: 'Employees.name',
  operator: 'contains',
  values: ['John']
}

// Not contains
{
  member: 'Employees.email',
  operator: 'notContains',
  values: ['@competitor.com']
}

// Starts with
{
  member: 'Employees.name',
  operator: 'startsWith',
  values: ['Dr.']
}

// Ends with
{
  member: 'Employees.email',
  operator: 'endsWith',
  values: ['@company.com']
}

// Not equals
{
  member: 'Employees.status',
  operator: 'notEquals',
  values: ['terminated']
}
```

### Numeric Operators

```typescript
// Greater than
{
  member: 'Employees.salary',
  operator: 'gt',
  values: [100000]
}

// Greater than or equal
{
  member: 'Employees.salary',
  operator: 'gte',
  values: [50000]
}

// Less than
{
  member: 'Employees.yearsOfService',
  operator: 'lt',
  values: [5]
}

// Less than or equal
{
  member: 'Employees.age',
  operator: 'lte',
  values: [65]
}

// Between (range)
{
  member: 'Employees.salary',
  operator: 'between',
  values: [50000, 150000]
}
```

### Null/Empty Operators

```typescript
// Is set (not null)
{
  member: 'Employees.departmentId',
  operator: 'set'
}

// Not set (is null)
{
  member: 'Employees.terminationDate',
  operator: 'notSet'
}

// Is empty (null or empty string)
{
  member: 'Employees.middleName',
  operator: 'isEmpty'
}

// Is not empty
{
  member: 'Employees.email',
  operator: 'isNotEmpty'
}
```

### Date Operators

```typescript
// In date range
{
  member: 'Employees.createdAt',
  operator: 'inDateRange',
  values: ['2024-01-01', '2024-12-31']
}

// Before date
{
  member: 'Employees.createdAt',
  operator: 'beforeDate',
  values: ['2024-06-01']
}

// After date
{
  member: 'Employees.createdAt',
  operator: 'afterDate',
  values: ['2023-01-01']
}
```

### Compound Filters (AND)

```typescript
const query = {
  measures: ['Employees.count'],
  filters: [
    {
      and: [
        {
          member: 'Employees.department',
          operator: 'equals',
          values: ['Engineering']
        },
        {
          member: 'Employees.isActive',
          operator: 'equals',
          values: [true]
        },
        {
          member: 'Employees.salary',
          operator: 'gte',
          values: [100000]
        }
      ]
    }
  ]
}

// Result: Active engineers making >= $100k
```

### Compound Filters (OR)

```typescript
const query = {
  measures: ['Employees.count'],
  filters: [
    {
      or: [
        {
          member: 'Employees.department',
          operator: 'equals',
          values: ['Engineering']
        },
        {
          member: 'Employees.department',
          operator: 'equals',
          values: ['Data Science']
        }
      ]
    }
  ]
}

// Result: Employees in either Engineering or Data Science
```

### Nested Compound Filters

```typescript
const query = {
  measures: ['Employees.count'],
  filters: [
    {
      and: [
        {
          member: 'Employees.isActive',
          operator: 'equals',
          values: [true]
        },
        {
          or: [
            {
              member: 'Employees.salary',
              operator: 'gte',
              values: [100000]
            },
            {
              member: 'Employees.yearsOfService',
              operator: 'gte',
              values: [10]
            }
          ]
        }
      ]
    }
  ]
}

// Result: Active employees who either earn >= $100k OR have >= 10 years service
```

## Ordering

Sort query results by dimensions or measures.

### Order by Dimension

```typescript
const query = {
  measures: ['Employees.count'],
  dimensions: ['Employees.department'],
  order: {
    'Employees.department': 'asc'
  }
}
```

### Order by Measure

```typescript
const query = {
  measures: ['Employees.count', 'Employees.avgSalary'],
  dimensions: ['Employees.department'],
  order: {
    'Employees.count': 'desc' // Sort by employee count descending
  }
}
```

### Multiple Order Fields

```typescript
const query = {
  measures: ['Employees.count'],
  dimensions: ['Employees.department', 'Employees.isActive'],
  order: {
    'Employees.department': 'asc',
    'Employees.count': 'desc'
  }
}

// Result: Sorted by department (A-Z), then by count (high to low)
```

## Pagination

Limit results and implement pagination.

### Limit Results

```typescript
const query = {
  measures: ['Employees.count'],
  dimensions: ['Employees.name'],
  order: { 'Employees.count': 'desc' },
  limit: 10 // Top 10 only
}
```

### Pagination with Offset

```typescript
// Page 1 (first 20 results)
const page1 = {
  measures: ['Employees.count'],
  dimensions: ['Employees.name'],
  order: { 'Employees.name': 'asc' },
  limit: 20,
  offset: 0
}

// Page 2 (results 21-40)
const page2 = {
  measures: ['Employees.count'],
  dimensions: ['Employees.name'],
  order: { 'Employees.name': 'asc' },
  limit: 20,
  offset: 20
}

// Page 3 (results 41-60)
const page3 = {
  measures: ['Employees.count'],
  dimensions: ['Employees.name'],
  order: { 'Employees.name': 'asc' },
  limit: 20,
  offset: 40
}
```

## Complete Query Examples

### Example 1: Sales Report

```typescript
const salesReport = {
  measures: [
    'Orders.count',
    'Orders.totalRevenue',
    'Orders.avgOrderValue'
  ],
  dimensions: [
    'Orders.productCategory',
    'Customers.region'
  ],
  timeDimensions: [{
    dimension: 'Orders.createdAt',
    granularity: 'month',
    dateRange: 'this year'
  }],
  filters: [{
    and: [
      {
        member: 'Orders.status',
        operator: 'equals',
        values: ['completed']
      },
      {
        member: 'Orders.totalAmount',
        operator: 'gte',
        values: [100]
      }
    ]
  }],
  order: {
    'Orders.totalRevenue': 'desc'
  },
  limit: 100
}

const result = await semanticLayer.execute(salesReport, securityContext)
```

### Example 2: Employee Analytics

```typescript
const employeeAnalytics = {
  measures: [
    'Employees.count',
    'Employees.avgSalary',
    'Employees.totalSalary'
  ],
  dimensions: [
    'Departments.name',
    'Employees.seniorityLevel'
  ],
  filters: [{
    and: [
      {
        member: 'Employees.isActive',
        operator: 'equals',
        values: [true]
      },
      {
        member: 'Employees.createdAt',
        operator: 'afterDate',
        values: ['2020-01-01']
      }
    ]
  }],
  order: {
    'Departments.name': 'asc',
    'Employees.avgSalary': 'desc'
  }
}

const result = await semanticLayer.execute(employeeAnalytics, securityContext)
```

### Example 3: Growth Metrics

```typescript
const growthMetrics = {
  measures: [
    'Users.count',
    'Users.newSignups',
    'Users.activeUsers'
  ],
  timeDimensions: [{
    dimension: 'Users.createdAt',
    granularity: 'week',
    dateRange: 'last 90 days'
  }],
  filters: [{
    member: 'Users.isVerified',
    operator: 'equals',
    values: [true]
  }],
  order: {
    'Users.createdAt': 'asc'
  }
}

const result = await semanticLayer.execute(growthMetrics, securityContext)
```

### Example 4: Top Performers

```typescript
const topPerformers = {
  measures: [
    'Employees.count',
    'Productivity.avgLinesOfCode',
    'Productivity.totalDeployments'
  ],
  dimensions: [
    'Employees.name',
    'Departments.name'
  ],
  timeDimensions: [{
    dimension: 'Productivity.date',
    dateRange: 'last 30 days'
  }],
  filters: [{
    and: [
      {
        member: 'Employees.isActive',
        operator: 'equals',
        values: [true]
      },
      {
        member: 'Productivity.totalDeployments',
        operator: 'gt',
        values: [0]
      }
    ]
  }],
  order: {
    'Productivity.totalDeployments': 'desc'
  },
  limit: 10
}

const result = await semanticLayer.execute(topPerformers, securityContext)
```

## Query Execution

### Basic Execution

```typescript
import { SemanticLayerCompiler } from 'drizzle-cube'

const semanticLayer = new SemanticLayerCompiler({ drizzle: db, schema })
semanticLayer.registerCube(employeesCube)

const result = await semanticLayer.execute(query, {
  organisationId: 'org-123',
  userId: 'user-456'
})

console.log(result.data) // Array of result rows
console.log(result.annotation) // Metadata about measures/dimensions
```

### Result Structure

```typescript
interface QueryResult {
  data: Array<Record<string, any>> // Result rows
  annotation: {
    measures: Record<string, MeasureAnnotation>
    dimensions: Record<string, DimensionAnnotation>
    timeDimensions: Record<string, TimeDimensionAnnotation>
  }
  requestId: string
  slowQuery: boolean
  query: any // Transformed query
}

// Example result
const result = {
  data: [
    {
      'Employees.department': 'Engineering',
      'Employees.count': 50,
      'Employees.avgSalary': 125000
    },
    {
      'Employees.department': 'Sales',
      'Employees.count': 30,
      'Employees.avgSalary': 95000
    }
  ],
  annotation: {
    measures: {
      'Employees.count': { title: 'Employee Count', type: 'count' },
      'Employees.avgSalary': { title: 'Average Salary', type: 'avg' }
    },
    dimensions: {
      'Employees.department': { title: 'Department', type: 'string' }
    }
  },
  requestId: 'req-abc123',
  slowQuery: false
}
```

## Best Practices

1. **Specify only needed fields** - Don't request all measures/dimensions unnecessarily
2. **Use filters early** - Apply filters to reduce data processed
3. **Limit large result sets** - Use `limit` and `offset` for pagination
4. **Order strategically** - Order by measures for "top N" queries
5. **Combine filters efficiently** - Use AND/OR appropriately to minimize data scanned
6. **Use time dimensions** - Leverage granularity for time-series analysis
7. **Test security context** - Always validate multi-tenant isolation

## Common Pitfalls

- **Missing security context** - Every query requires a security context
- **Wrong operator** - Use `equals` not `=`, `gte` not `>=`
- **Invalid date format** - Use ISO format: `2024-01-01`
- **Mixing AND/OR incorrectly** - Nested compound filters need careful structure
- **Performance issues** - Large queries without limits can be slow

## All Available Filter Operators

```typescript
// String operators
'equals', 'notEquals', 'contains', 'notContains', 'startsWith', 'endsWith'

// Numeric operators
'gt', 'gte', 'lt', 'lte', 'between'

// Array operators
'in', 'notIn'

// Null operators
'set', 'notSet', 'isEmpty', 'isNotEmpty'

// Date operators
'inDateRange', 'beforeDate', 'afterDate'

// Pattern operators
'like', 'ilike', 'regex'
```

## Next Steps

- Create **charts** from your queries with the chart skills
- Build **dashboards** with the `dashboard` skill
- Learn about **cube definitions** with the `cube-definition` skill
- Set up **server APIs** with the `server-setup` skill
