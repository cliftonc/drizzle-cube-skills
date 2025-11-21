---
name: markdown
description: Configure markdown content portlets in drizzle-cube dashboards for static text, documentation, and formatted content. Use when adding text, documentation, headers, or formatted content to dashboards.
---

# Markdown Portlet Configuration

Configure markdown content portlets for drizzle-cube dashboards. Markdown portlets display static formatted text, useful for headers, documentation, and annotations.

## Chart Type

```typescript
chartType: 'markdown'
```

## Basic Configuration

```typescript
{
  id: 'markdown-1',
  title: '', // Usually leave empty for markdown
  query: JSON.stringify({
    // Empty query or placeholder
  }),
  chartType: 'markdown',
  displayConfig: {
    content: '# Dashboard Title\n\nWelcome to the analytics dashboard.'
  },
  x: 0, y: 0, w: 12, h: 2
}
```

## Display Configuration (`displayConfig`)

### content
- **Type**: `string`
- **Purpose**: Markdown content to display
- **Supports**: Standard markdown formatting

## Examples

### Dashboard Header

```typescript
{
  id: 'header',
  title: '',
  query: JSON.stringify({}),
  chartType: 'markdown',
  displayConfig: {
    content: `# Sales Analytics Dashboard

**Last Updated:** ${new Date().toLocaleDateString()}

This dashboard provides real-time insights into sales performance.`
  },
  x: 0, y: 0, w: 12, h: 2
}
```

### Section Divider

```typescript
{
  id: 'section-header',
  title: '',
  query: JSON.stringify({}),
  chartType: 'markdown',
  displayConfig: {
    content: '## Employee Metrics\n\nThe following charts show employee-related KPIs and trends.'
  },
  x: 0, y: 4, w: 12, h: 1
}
```

### Documentation

```typescript
{
  id: 'docs',
  title: 'How to Use',
  query: JSON.stringify({}),
  chartType: 'markdown',
  displayConfig: {
    content: `### Dashboard Guide

- **Revenue Charts**: Track monthly revenue trends
- **User Metrics**: Monitor active user counts
- **Performance**: View system performance indicators

For questions, contact: analytics@company.com`
  },
  x: 0, y: 0, w: 4, h: 4
}
```

### Alert/Notice

```typescript
{
  id: 'alert',
  title: '',
  query: JSON.stringify({}),
  chartType: 'markdown',
  displayConfig: {
    content: `> **Important Notice**
>
> Data is updated every 15 minutes. Last sync: ${new Date().toLocaleTimeString()}`
  },
  x: 0, y: 0, w: 6, h: 1
}
```

## Markdown Syntax Supported

```markdown
# Heading 1
## Heading 2
### Heading 3

**Bold text**
*Italic text*

- Bullet list
- Item 2

1. Numbered list
2. Item 2

[Link text](https://example.com)

> Blockquote

`inline code`

---
Horizontal rule
```

## Use Cases

- **Dashboard Headers**: Title and description
- **Section Dividers**: Organize dashboard sections
- **Documentation**: Explain metrics and charts
- **Alerts**: Display important notices
- **Instructions**: Guide dashboard users

## Best Practices

1. **Keep it concise** - Brief, scannable content
2. **Use headings** - Organize information clearly
3. **Add context** - Explain what users are seeing
4. **Update timestamps** - Show data freshness
5. **Link to docs** - Provide additional resources

## Related Skills

- Use with `dashboard` skill to add context
- Combine with data visualizations for complete dashboards
