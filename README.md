# Drizzle Cube Skills for Claude Code

A comprehensive set of Claude Code skills for working with [Drizzle Cube](https://github.com/cliftonc/drizzle-cube) - a Drizzle ORM-first semantic layer with type-safe analytics and dashboards.

## What is This?

This marketplace provides Claude Code with specialized knowledge about Drizzle Cube, enabling it to help you:

- **Define semantic cubes** with dimensions, measures, and time dimensions
- **Configure cube joins** using belongsTo, hasOne, hasMany, and belongsToMany relationships
- **Build type-safe queries** with filters, aggregations, and date ranges
- **Set up server APIs** with Express, Fastify, Hono, or Next.js adapters
- **Create analytics dashboards** using React components
- **Configure charts** - 15 different chart types with specific parameters

## Installation

### Step 1: Add the Marketplace

In Claude Code, run:

```bash
/plugin marketplace add cliftonc/drizzle-cube-skills
```

### Step 2: Install the Skills

```bash
/plugin install drizzle-cube@drizzle-cube-skills
```

That's it! Claude Code now has access to all Drizzle Cube skills.

## Available Skills

### Core Skills

- **`cube-definition`** - Define semantic layer cubes with dimensions, measures, and security context
- **`cube-joins`** - Configure relationships between cubes (belongsTo, hasOne, hasMany, belongsToMany)
- **`queries`** - Build semantic queries with filters, time dimensions, and aggregations
- **`server-setup`** - Initialize the semantic layer with framework adapters
- **`dashboard`** - Create interactive analytics dashboards with React components

### Chart Skills

Each chart type has its own skill with specific configuration guidance:

- **`bar-chart`** - Bar and column charts with stacking support
- **`line-chart`** - Line charts with multiple series
- **`area-chart`** - Area charts with stacking
- **`pie-chart`** - Pie and doughnut charts
- **`scatter-chart`** - Scatter plots
- **`radar-chart`** - Radar/spider charts
- **`radial-bar-chart`** - Radial bar charts
- **`treemap-chart`** - Hierarchical treemaps
- **`bubble-chart`** - Bubble charts with size and color dimensions
- **`activity-grid`** - GitHub-style activity heatmaps
- **`kpi-number`** - Single-value KPI displays
- **`kpi-delta`** - KPI with delta/change indicators
- **`kpi-text`** - Text-based KPI displays
- **`markdown`** - Markdown content portlets
- **`table`** - Sortable data tables

## Usage

Skills are activated automatically when you ask Claude about related tasks. For example:

- "Help me create a cube for my users table with dimensions and measures" → activates `cube-definition`
- "How do I join the employees cube to departments?" → activates `cube-joins`
- "Create a bar chart showing sales by region" → activates `bar-chart`
- "Build a dashboard with multiple charts" → activates `dashboard`

## Example Prompts

### Creating a Cube

```
"Create a semantic cube for my products table with:
- Dimensions: name, category, sku
- Measures: count, total_price (sum), average_price
- Security filtering by organisation_id"
```

### Building Queries

```
"Write a query to get product counts by category for the last 30 days,
filtered to only active products, ordered by count descending"
```

### Creating Dashboards

```
"Create a dashboard with:
- A bar chart showing sales by month
- A KPI number for total revenue
- An activity grid showing daily engagement"
```

## Features

All skills include:

- **Type-safe patterns** - Full TypeScript support with Drizzle ORM
- **Security-first** - Multi-tenant security context patterns
- **Multi-database support** - Works with PostgreSQL, MySQL, and SQLite
- **Real examples** - Code snippets from the Drizzle Cube codebase
- **Best practices** - Guardrails and common pitfalls

## Documentation

For complete Drizzle Cube documentation, visit:
- Main docs: https://drizzle-cube.dev
- GitHub: https://github.com/cliftonc/drizzle-cube
- NPM: https://www.npmjs.com/package/drizzle-cube

## Contributing

Found an issue or want to improve these skills?
Open an issue or PR at: https://github.com/cliftonc/drizzle-cube-skills

## License

MIT License - See LICENSE file for details
