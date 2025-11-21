---
name: treemap-chart
description: Configure treemap charts in drizzle-cube dashboards for hierarchical data visualization. Use when creating treemaps, hierarchical visualizations, or nested proportions.
---

# Treemap Chart Configuration

Configure treemap charts for drizzle-cube dashboards. Treemaps display hierarchical data as nested rectangles, where size represents a metric.

## Chart Type

```typescript
chartType: 'treemap'
```

## Basic Configuration

```typescript
{
  id: 'treemap-1',
  title: 'Revenue by Category',
  query: JSON.stringify({
    dimensions: ['Products.category'],
    measures: ['Sales.totalRevenue']
  }),
  chartType: 'treemap',
  chartConfig: {
    series: ['Products.category'],
    yAxis: ['Sales.totalRevenue']
  },
  x: 0, y: 0, w: 8, h: 6
}
```

## Chart Configuration (`chartConfig`)

### series (Hierarchy)
- **Type**: `string[]`
- **Purpose**: Dimensions for hierarchy levels
- **Example**: `['Category.name', 'Subcategory.name']`

### yAxis (Size)
- **Type**: `string[]`
- **Purpose**: Measure determining rectangle size
- **Example**: `['Sales.revenue']`

## Examples

### Sales by Category

```typescript
{
  id: 'sales-treemap',
  title: 'Sales Distribution',
  query: JSON.stringify({
    dimensions: ['Products.category', 'Products.subcategory'],
    measures: ['Sales.totalRevenue']
  }),
  chartType: 'treemap',
  chartConfig: {
    series: ['Products.category', 'Products.subcategory'],
    yAxis: ['Sales.totalRevenue']
  },
  x: 0, y: 0, w: 12, h: 7
}
```

## Use Cases

- **Hierarchical Proportions**: Show nested categories
- **Disk Space Usage**: Visualize file/folder sizes
- **Budget Allocation**: Display spending hierarchy
- **Market Composition**: Show market segments

## Related Skills

- Use `pie-chart` for flat proportions
- Use `bar-chart` for hierarchical comparisons
