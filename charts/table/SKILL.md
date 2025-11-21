---
name: table
description: Configure data tables in drizzle-cube dashboards for displaying detailed records with sorting. Use when creating data tables, sortable lists, detailed views, or tabular data displays.
---

# Data Table Configuration

Configure data tables for drizzle-cube dashboards. Tables display detailed records with columns for dimensions and measures, supporting sorting and pagination.

## Chart Type

```typescript
chartType: 'table'
```

## Basic Configuration

```typescript
{
  id: 'table-1',
  title: 'Employee List',
  query: JSON.stringify({
    dimensions: [
      'Employees.name',
      'Employees.email',
      'Departments.name',
      'Employees.createdAt'
    ],
    measures: ['Employees.salary'],
    order: { 'Employees.createdAt': 'desc' },
    limit: 20
  }),
  chartType: 'table',
  x: 0, y: 0, w: 12, h: 6
}
```

## Chart Configuration

Tables don't require `chartConfig` - all dimensions and measures from the query are displayed as columns.

## Query Configuration

### dimensions
- Displayed as standard columns
- Sortable by clicking column headers

### measures
- Displayed as numeric columns
- Formatted appropriately (currency, decimals)

### order
- Set default sort order
- Example: `{ 'Employees.name': 'asc' }`

### limit
- Control number of rows
- Recommended: 10-50 rows

## Examples

### Employee List

```typescript
{
  id: 'employee-table',
  title: 'All Employees',
  query: JSON.stringify({
    dimensions: [
      'Employees.id',
      'Employees.name',
      'Employees.email',
      'Departments.name',
      'Employees.isActive'
    ],
    measures: ['Employees.salary'],
    order: { 'Employees.name': 'asc' }
  }),
  chartType: 'table',
  x: 0, y: 0, w: 12, h: 6
}
```

### Recent Orders

```typescript
{
  id: 'recent-orders',
  title: 'Recent Orders',
  query: JSON.stringify({
    dimensions: [
      'Orders.id',
      'Orders.customerName',
      'Orders.status',
      'Orders.createdAt'
    ],
    measures: ['Orders.totalAmount'],
    order: { 'Orders.createdAt': 'desc' },
    limit: 20
  }),
  chartType: 'table',
  x: 0, y: 0, w: 12, h: 5
}
```

### Top Products

```typescript
{
  id: 'top-products',
  title: 'Top Selling Products',
  query: JSON.stringify({
    dimensions: [
      'Products.name',
      'Products.category'
    ],
    measures: [
      'Orders.count',
      'Orders.totalRevenue'
    ],
    order: { 'Orders.totalRevenue': 'desc' },
    limit: 10
  }),
  chartType: 'table',
  x: 0, y: 0, w: 8, h: 5
}
```

## Use Cases

- **Detailed Records**: Show full record details
- **Rankings**: Display top/bottom items
- **Recent Activity**: List recent transactions
- **Comparison Tables**: Compare multiple items
- **Export Data**: Provide data for copying/exporting

## Best Practices

1. **Limit rows** - Use `limit` for performance
2. **Set default sort** - Provide meaningful default order
3. **Choose relevant columns** - Don't show all fields
4. **Format measures** - Ensure proper number formatting
5. **Consider pagination** - Use `offset` for large datasets

## Related Skills

- Use `queries` skill for complex table queries
- Use `bar-chart` for visual comparisons
