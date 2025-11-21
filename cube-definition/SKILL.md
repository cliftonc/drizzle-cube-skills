---
name: cube-definition
description: Define semantic layer cubes with Drizzle ORM tables, including dimensions, measures, time dimensions, and security context. Use when creating analytics cubes, defining data models, setting up multi-tenant filtering, or working with drizzle-cube semantic layers.
---

# Drizzle Cube Definition

This skill helps you create semantic layer cubes using Drizzle Cube's `defineCube` function. Cubes provide a business-friendly abstraction over database tables with type-safe dimensions, measures, and built-in security.

## Core Concept

A **cube** in Drizzle Cube is:
- A semantic layer over one or more database tables
- Defined using Drizzle ORM table references
- **Always filtered by security context** (mandatory for multi-tenant isolation)
- Type-safe with full TypeScript support

## Basic Cube Structure

```typescript
import { defineCube } from 'drizzle-cube'
import { eq } from 'drizzle-orm'
import { employees } from './schema' // Your Drizzle schema

export const employeesCube = defineCube({
  name: 'Employees',
  title: 'Employee Analytics', // Optional human-readable title
  description: 'Analytics for employee data', // Optional description

  // MANDATORY: Security context filtering for multi-tenant isolation
  sql: (ctx) => ({
    from: employees,
    where: eq(employees.organisationId, ctx.securityContext.organisationId)
  }),

  // Define dimensions (categorical/time fields for grouping/filtering)
  dimensions: {
    id: {
      name: 'id',
      title: 'Employee ID',
      type: 'number',
      sql: employees.id,
      primaryKey: true // Mark the primary key
    },
    name: {
      name: 'name',
      title: 'Employee Name',
      type: 'string',
      sql: employees.name
    },
    email: {
      name: 'email',
      title: 'Email Address',
      type: 'string',
      sql: employees.email
    },
    departmentId: {
      name: 'departmentId',
      title: 'Department',
      type: 'number',
      sql: employees.departmentId
    },
    isActive: {
      name: 'isActive',
      title: 'Active Status',
      type: 'boolean',
      sql: employees.isActive
    },
    createdAt: {
      name: 'createdAt',
      title: 'Created Date',
      type: 'time',
      sql: employees.createdAt
    }
  },

  // Define measures (aggregated numeric values)
  measures: {
    count: {
      name: 'count',
      title: 'Total Employees',
      type: 'count',
      sql: employees.id
    },
    totalSalary: {
      name: 'totalSalary',
      title: 'Total Salary',
      type: 'sum',
      sql: employees.salary
    },
    avgSalary: {
      name: 'avgSalary',
      title: 'Average Salary',
      type: 'avg',
      sql: employees.salary
    },
    minSalary: {
      name: 'minSalary',
      title: 'Minimum Salary',
      type: 'min',
      sql: employees.salary
    },
    maxSalary: {
      name: 'maxSalary',
      title: 'Maximum Salary',
      type: 'max',
      sql: employees.salary
    }
  }
})
```

## Dimension Types

Drizzle Cube supports four dimension types:

### 1. String Dimensions
```typescript
dimensions: {
  name: {
    name: 'name',
    title: 'Full Name',
    type: 'string',
    sql: employees.name
  },
  email: {
    name: 'email',
    type: 'string',
    sql: employees.email
  }
}
```

### 2. Number Dimensions
```typescript
dimensions: {
  id: {
    name: 'id',
    type: 'number',
    sql: employees.id,
    primaryKey: true
  },
  departmentId: {
    name: 'departmentId',
    type: 'number',
    sql: employees.departmentId
  }
}
```

### 3. Time Dimensions
```typescript
dimensions: {
  createdAt: {
    name: 'createdAt',
    title: 'Created Date',
    type: 'time',
    sql: employees.createdAt
  },
  updatedAt: {
    name: 'updatedAt',
    type: 'time',
    sql: employees.updatedAt
  }
}
```

### 4. Boolean Dimensions
```typescript
dimensions: {
  isActive: {
    name: 'isActive',
    title: 'Active',
    type: 'boolean',
    sql: employees.isActive
  },
  isRemote: {
    name: 'isRemote',
    title: 'Remote Worker',
    type: 'boolean',
    sql: employees.isRemote
  }
}
```

## Measure Types

Drizzle Cube supports several aggregation types:

### 1. Count Measures
```typescript
measures: {
  count: {
    name: 'count',
    title: 'Total Count',
    type: 'count',
    sql: employees.id // Column to count
  },
  activeCount: {
    name: 'activeCount',
    title: 'Active Employees',
    type: 'count',
    sql: employees.id,
    filters: [(ctx) => eq(employees.isActive, true)] // Filtered count
  }
}
```

### 2. Count Distinct Measures
```typescript
measures: {
  uniqueDepartments: {
    name: 'uniqueDepartments',
    title: 'Unique Departments',
    type: 'countDistinct',
    sql: employees.departmentId
  }
}
```

### 3. Sum Measures
```typescript
measures: {
  totalSalary: {
    name: 'totalSalary',
    title: 'Total Salary',
    type: 'sum',
    sql: employees.salary
  }
}
```

### 4. Average Measures
```typescript
measures: {
  avgSalary: {
    name: 'avgSalary',
    title: 'Average Salary',
    type: 'avg',
    sql: employees.salary
  }
}
```

### 5. Min/Max Measures
```typescript
measures: {
  minSalary: {
    name: 'minSalary',
    title: 'Minimum Salary',
    type: 'min',
    sql: employees.salary
  },
  maxSalary: {
    name: 'maxSalary',
    title: 'Maximum Salary',
    type: 'max',
    sql: employees.salary
  }
}
```

### 6. Calculated Measures
```typescript
measures: {
  salaryPercentage: {
    name: 'salaryPercentage',
    title: 'Salary as Percentage',
    type: 'calculated',
    calculatedSql: '{totalSalary} / NULLIF({departmentBudget}, 0) * 100'
  }
}
```

## Advanced Patterns

### Filtered Measures

Add filters to measures for conditional aggregation:

```typescript
measures: {
  activeEmployees: {
    name: 'activeEmployees',
    title: 'Active Employees',
    type: 'count',
    sql: employees.id,
    filters: [
      (ctx) => eq(employees.isActive, true)
    ]
  },
  seniorEmployees: {
    name: 'seniorEmployees',
    title: 'Senior Employees',
    type: 'count',
    sql: employees.id,
    filters: [
      (ctx) => {
        const { sql, gte } = ctx.imports
        return gte(employees.yearsOfService, 5)
      }
    ]
  },
  highEarners: {
    name: 'highEarners',
    title: 'High Earners',
    type: 'count',
    sql: employees.id,
    filters: [
      (ctx) => {
        const { gt } = ctx.imports
        return gt(employees.salary, 100000)
      }
    ]
  }
}
```

### Computed Dimensions

Use SQL expressions for computed values:

```typescript
import { sql } from 'drizzle-orm'

dimensions: {
  fullName: {
    name: 'fullName',
    title: 'Full Name',
    type: 'string',
    sql: sql`${employees.firstName} || ' ' || ${employees.lastName}`
  },
  seniorityLevel: {
    name: 'seniorityLevel',
    title: 'Seniority',
    type: 'string',
    sql: sql`CASE
      WHEN ${employees.yearsOfService} < 2 THEN 'Junior'
      WHEN ${employees.yearsOfService} < 5 THEN 'Mid-level'
      ELSE 'Senior'
    END`
  }
}
```

## Security Context (MANDATORY)

**Every cube MUST filter by security context** to ensure multi-tenant data isolation:

```typescript
// ✅ CORRECT - Security context filtering
sql: (ctx) => ({
  from: employees,
  where: eq(employees.organisationId, ctx.securityContext.organisationId)
})

// ✅ CORRECT - Multiple security conditions
sql: (ctx) => ({
  from: employees,
  where: and(
    eq(employees.organisationId, ctx.securityContext.organisationId),
    eq(employees.tenantId, ctx.securityContext.tenantId)
  )
})

// ❌ WRONG - No security filtering (data leak!)
sql: (ctx) => ({
  from: employees
  // Missing where clause - SECURITY VIOLATION
})
```

## Complete Example

```typescript
import { defineCube } from 'drizzle-cube'
import { eq, sql, and, gte } from 'drizzle-orm'
import { employees } from './schema'

export const employeesCube = defineCube({
  name: 'Employees',
  title: 'Employee Analytics',
  description: 'Comprehensive employee data and metrics',

  // Security context filtering (MANDATORY)
  sql: (ctx) => ({
    from: employees,
    where: eq(employees.organisationId, ctx.securityContext.organisationId)
  }),

  dimensions: {
    id: {
      name: 'id',
      title: 'Employee ID',
      type: 'number',
      sql: employees.id,
      primaryKey: true
    },
    name: {
      name: 'name',
      title: 'Name',
      type: 'string',
      sql: employees.name
    },
    email: {
      name: 'email',
      title: 'Email',
      type: 'string',
      sql: employees.email
    },
    department: {
      name: 'department',
      title: 'Department',
      type: 'string',
      sql: employees.departmentName
    },
    isActive: {
      name: 'isActive',
      title: 'Active',
      type: 'boolean',
      sql: employees.isActive
    },
    createdAt: {
      name: 'createdAt',
      title: 'Hire Date',
      type: 'time',
      sql: employees.createdAt
    },
    // Computed dimension
    seniorityLevel: {
      name: 'seniorityLevel',
      title: 'Seniority Level',
      type: 'string',
      sql: sql`CASE
        WHEN ${employees.yearsOfService} < 2 THEN 'Junior'
        WHEN ${employees.yearsOfService} < 5 THEN 'Mid-level'
        ELSE 'Senior'
      END`
    }
  },

  measures: {
    count: {
      name: 'count',
      title: 'Total Employees',
      type: 'count',
      sql: employees.id
    },
    activeCount: {
      name: 'activeCount',
      title: 'Active Employees',
      type: 'count',
      sql: employees.id,
      filters: [(ctx) => eq(employees.isActive, true)]
    },
    totalSalary: {
      name: 'totalSalary',
      title: 'Total Salary',
      type: 'sum',
      sql: employees.salary
    },
    avgSalary: {
      name: 'avgSalary',
      title: 'Average Salary',
      type: 'avg',
      sql: employees.salary
    },
    minSalary: {
      name: 'minSalary',
      title: 'Minimum Salary',
      type: 'min',
      sql: employees.salary
    },
    maxSalary: {
      name: 'maxSalary',
      title: 'Maximum Salary',
      type: 'max',
      sql: employees.salary
    },
    uniqueDepartments: {
      name: 'uniqueDepartments',
      title: 'Unique Departments',
      type: 'countDistinct',
      sql: employees.departmentId
    },
    // Filtered measure
    seniorEmployees: {
      name: 'seniorEmployees',
      title: 'Senior Employees',
      type: 'count',
      sql: employees.id,
      filters: [(ctx) => gte(employees.yearsOfService, 5)]
    }
  }
})
```

## Registering Cubes

Once defined, register cubes with the semantic layer compiler:

```typescript
import { SemanticLayerCompiler } from 'drizzle-cube'
import { drizzle } from 'drizzle-orm/postgres-js'
import { employeesCube } from './cubes/employees'

const db = drizzle(process.env.DATABASE_URL)

const compiler = new SemanticLayerCompiler({
  drizzle: db,
  schema: schema
})

// Register your cube
compiler.registerCube(employeesCube)
```

## Best Practices

1. **Always include security context filtering** - This is mandatory for multi-tenant isolation
2. **Use meaningful names** - Dimension and measure names should be clear and descriptive
3. **Add titles** - Provide human-readable titles for UI display
4. **Mark primary keys** - Set `primaryKey: true` on ID dimensions
5. **Type safety** - Use Drizzle ORM table references for compile-time validation
6. **Filtered measures** - Use filters for conditional aggregations instead of creating separate cubes

## Common Pitfalls

- **Missing security context** - Every cube must filter by security context
- **Wrong SQL syntax** - Use Drizzle ORM operators (eq, and, or), not raw SQL strings
- **Incorrect types** - Ensure dimension/measure types match the actual data types
- **Missing imports** - Import necessary operators from drizzle-orm

## Next Steps

- Learn about **cube joins** with the `cube-joins` skill
- Build **queries** using your cubes with the `queries` skill
- Set up **server APIs** with the `server-setup` skill
