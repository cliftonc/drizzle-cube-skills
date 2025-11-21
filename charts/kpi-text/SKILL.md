---
name: kpi-text
description: Configure KPI text displays in drizzle-cube dashboards for showing text-based metrics and status indicators. Use when displaying text KPIs, status values, or categorical indicators.
---

# KPI Text Configuration

Configure KPI text displays for drizzle-cube dashboards. KPI text shows non-numeric values like status, categories, or text-based indicators.

## Chart Type

```typescript
chartType: 'kpiText'
```

## Basic Configuration

```typescript
{
  id: 'kpi-text-1',
  title: 'System Status',
  query: JSON.stringify({
    dimensions: ['System.status']
  }),
  chartType: 'kpiText',
  x: 0, y: 0, w: 3, h: 2
}
```

## Examples

### Status Indicator

```typescript
{
  id: 'system-status',
  title: 'System Status',
  query: JSON.stringify({
    dimensions: ['System.status']
  }),
  chartType: 'kpiText',
  x: 0, y: 0, w: 3, h: 2
}
```

### Latest Value

```typescript
{
  id: 'latest-update',
  title: 'Last Updated',
  query: JSON.stringify({
    dimensions: ['System.lastUpdateTime'],
    order: { 'System.lastUpdateTime': 'desc' },
    limit: 1
  }),
  chartType: 'kpiText',
  x: 3, y: 0, w: 3, h: 2
}
```

### Category Value

```typescript
{
  id: 'top-category',
  title: 'Top Performing Category',
  query: JSON.stringify({
    dimensions: ['Products.category'],
    measures: ['Sales.revenue'],
    order: { 'Sales.revenue': 'desc' },
    limit: 1
  }),
  chartType: 'kpiText',
  x: 6, y: 0, w: 3, h: 2
}
```

## Use Cases

- **Status Displays**: Show current system/service status
- **Latest Values**: Display most recent text value
- **Top Items**: Show highest-ranking category/item
- **Text Indicators**: Display non-numeric metrics

## Related Skills

- Use `kpi-number` for numeric metrics
- Use `kpi-delta` for trending text values
