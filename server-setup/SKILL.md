---
name: server-setup
description: Set up drizzle-cube API server with Express, Fastify, Hono, or Next.js framework adapters. Use when configuring the semantic layer server, setting up API endpoints, extracting security context, or initializing drizzle-cube with different web frameworks.
---

# Drizzle Cube Server Setup

This skill helps you set up a Drizzle Cube API server using framework adapters for Express, Fastify, Hono, or Next.js. These adapters provide Cube.js-compatible API endpoints for your semantic layer.

## Core Concept

Drizzle Cube provides framework adapters that:
- Expose Cube.js-compatible REST API endpoints
- Handle security context extraction from requests
- Integrate with your existing web framework
- Support `/load`, `/sql`, and `/meta` endpoints

## Semantic Layer Initialization

First, create and configure the semantic layer compiler:

```typescript
import { SemanticLayerCompiler } from 'drizzle-cube'
import { drizzle } from 'drizzle-orm/postgres-js'
import postgres from 'postgres'
import * as schema from './schema'
import { employeesCube, departmentsCube } from './cubes'

// Initialize database connection
const queryClient = postgres(process.env.DATABASE_URL)
const db = drizzle(queryClient, { schema })

// Create semantic layer compiler
const compiler = new SemanticLayerCompiler({
  drizzle: db,
  schema: schema,
  engineType: 'postgres' // or 'mysql', 'sqlite', or auto-detect
})

// Register your cubes
compiler.registerCube(employeesCube)
compiler.registerCube(departmentsCube)
// ... register more cubes

export { compiler }
```

## Express Adapter

### Installation

```bash
npm install express drizzle-cube
```

### Basic Setup

```typescript
import express from 'express'
import { createCubeApi } from 'drizzle-cube/adapters/express'
import { compiler } from './semantic-layer'

const app = express()
app.use(express.json())

// Create Cube API router
const cubeApi = createCubeApi({
  // Extract security context from request
  extractSecurityContext: async (req) => {
    // Example: Extract from authenticated user
    return {
      organisationId: req.user?.organisationId || 'default-org',
      userId: req.user?.id
    }
  },
  semanticLayer: compiler
})

// Mount the API
app.use('/cubejs-api/v1', cubeApi)

// Start server
app.listen(3000, () => {
  console.log('Drizzle Cube API listening on port 3000')
})
```

### With Authentication Middleware

```typescript
import express from 'express'
import { createCubeApi } from 'drizzle-cube/adapters/express'
import { authenticateJWT } from './auth'
import { compiler } from './semantic-layer'

const app = express()
app.use(express.json())

// Authentication middleware
app.use('/cubejs-api', authenticateJWT)

const cubeApi = createCubeApi({
  extractSecurityContext: async (req) => {
    // req.user is populated by authenticateJWT middleware
    if (!req.user) {
      throw new Error('Unauthorized: No user found')
    }

    return {
      organisationId: req.user.organisationId,
      userId: req.user.id,
      role: req.user.role
    }
  },
  semanticLayer: compiler
})

app.use('/cubejs-api/v1', cubeApi)

app.listen(3000)
```

## Fastify Adapter

### Installation

```bash
npm install fastify drizzle-cube
```

### Basic Setup

```typescript
import Fastify from 'fastify'
import { registerCubeApi } from 'drizzle-cube/adapters/fastify'
import { compiler } from './semantic-layer'

const fastify = Fastify({
  logger: true
})

// Register Cube API plugin
await registerCubeApi(fastify, {
  extractSecurityContext: async (request) => {
    // Extract from Fastify request
    return {
      organisationId: request.user?.organisationId || 'default-org',
      userId: request.user?.id
    }
  },
  semanticLayer: compiler
})

// Start server
await fastify.listen({ port: 3000 })
```

### With JWT Authentication

```typescript
import Fastify from 'fastify'
import fastifyJWT from '@fastify/jwt'
import { registerCubeApi } from 'drizzle-cube/adapters/fastify'
import { compiler } from './semantic-layer'

const fastify = Fastify({ logger: true })

// Register JWT plugin
await fastify.register(fastifyJWT, {
  secret: process.env.JWT_SECRET
})

// Authentication decorator
fastify.decorate('authenticate', async (request, reply) => {
  try {
    await request.jwtVerify()
  } catch (err) {
    reply.send(err)
  }
})

// Register Cube API with authentication
await registerCubeApi(fastify, {
  extractSecurityContext: async (request) => {
    // request.user is populated by JWT verification
    if (!request.user) {
      throw new Error('Unauthorized')
    }

    return {
      organisationId: request.user.organisationId,
      userId: request.user.sub,
      tenantId: request.user.tenantId
    }
  },
  semanticLayer: compiler,
  prefix: '/cubejs-api/v1' // Optional: custom prefix
})

// Protect routes
fastify.addHook('onRequest', fastify.authenticate)

await fastify.listen({ port: 3000 })
```

## Hono Adapter

### Installation

```bash
npm install hono drizzle-cube
```

### Basic Setup

```typescript
import { Hono } from 'hono'
import { createCubeApi } from 'drizzle-cube/adapters/hono'
import { compiler } from './semantic-layer'

const app = new Hono()

// Create Cube API
const cubeApi = createCubeApi({
  extractSecurityContext: async (req) => {
    // Extract from Hono request
    const authHeader = req.header('Authorization')
    const token = authHeader?.replace('Bearer ', '')

    // Decode token and extract context
    const user = await verifyToken(token)

    return {
      organisationId: user.organisationId,
      userId: user.id
    }
  },
  semanticLayer: compiler
})

// Mount the API
app.route('/cubejs-api/v1', cubeApi)

// Start server (for Node.js)
export default app
```

### With JWT Middleware

```typescript
import { Hono } from 'hono'
import { jwt } from 'hono/jwt'
import { createCubeApi } from 'drizzle-cube/adapters/hono'
import { compiler } from './semantic-layer'

const app = new Hono()

// JWT middleware
app.use('/cubejs-api/*', jwt({
  secret: process.env.JWT_SECRET
}))

const cubeApi = createCubeApi({
  extractSecurityContext: async (req) => {
    // Get JWT payload from context
    const payload = req.get('jwtPayload')

    return {
      organisationId: payload.organisationId,
      userId: payload.sub,
      permissions: payload.permissions
    }
  },
  semanticLayer: compiler
})

app.route('/cubejs-api/v1', cubeApi)

export default app
```

### Edge Runtime (Cloudflare Workers)

```typescript
import { Hono } from 'hono'
import { createCubeApi } from 'drizzle-cube/adapters/hono'
import { drizzle } from 'drizzle-orm/d1'
import { compiler } from './semantic-layer'

const app = new Hono<{ Bindings: { DB: D1Database } }>()

app.use('*', async (c, next) => {
  // Initialize Drizzle with D1 database binding
  const db = drizzle(c.env.DB)
  c.set('db', db)
  await next()
})

const cubeApi = createCubeApi({
  extractSecurityContext: async (req) => {
    const authHeader = req.header('Authorization')
    // Extract and verify token
    const user = await verifyEdgeToken(authHeader)

    return {
      organisationId: user.orgId,
      userId: user.sub
    }
  },
  semanticLayer: compiler
})

app.route('/cubejs-api/v1', cubeApi)

export default app
```

## Next.js Adapter

### Installation

```bash
npm install next drizzle-cube
```

### API Route Setup (App Router)

```typescript
// app/api/cubejs/[...cube]/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { createCubeApi } from 'drizzle-cube/adapters/nextjs'
import { compiler } from '@/lib/semantic-layer'
import { getServerSession } from 'next-auth'
import { authOptions } from '@/lib/auth'

const cubeHandlers = createCubeApi({
  extractSecurityContext: async (req) => {
    // Get session from Next Auth
    const session = await getServerSession(authOptions)

    if (!session?.user) {
      throw new Error('Unauthorized')
    }

    return {
      organisationId: session.user.organisationId,
      userId: session.user.id
    }
  },
  semanticLayer: compiler
})

export async function POST(
  request: NextRequest,
  { params }: { params: { cube: string[] } }
) {
  const endpoint = params.cube[0]

  try {
    switch (endpoint) {
      case 'load':
        return cubeHandlers.load(request)
      case 'sql':
        return cubeHandlers.sql(request)
      default:
        return NextResponse.json(
          { error: 'Endpoint not found' },
          { status: 404 }
        )
    }
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    )
  }
}

export async function GET(
  request: NextRequest,
  { params }: { params: { cube: string[] } }
) {
  const endpoint = params.cube[0]

  if (endpoint === 'meta') {
    return cubeHandlers.meta(request)
  }

  return NextResponse.json(
    { error: 'Endpoint not found' },
    { status: 404 }
  )
}
```

### API Route Setup (Pages Router)

```typescript
// pages/api/cubejs/[...cube].ts
import { NextApiRequest, NextApiResponse } from 'next'
import { createCubeApi } from 'drizzle-cube/adapters/nextjs'
import { compiler } from '@/lib/semantic-layer'
import { getSession } from 'next-auth/react'

const cubeHandlers = createCubeApi({
  extractSecurityContext: async (req) => {
    const session = await getSession({ req })

    if (!session?.user) {
      throw new Error('Unauthorized')
    }

    return {
      organisationId: session.user.organisationId,
      userId: session.user.id
    }
  },
  semanticLayer: compiler
})

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { cube } = req.query
  const endpoint = Array.isArray(cube) ? cube[0] : cube

  try {
    switch (endpoint) {
      case 'load':
        if (req.method === 'POST') {
          return await cubeHandlers.load(req, res)
        }
        break
      case 'sql':
        if (req.method === 'POST') {
          return await cubeHandlers.sql(req, res)
        }
        break
      case 'meta':
        if (req.method === 'GET') {
          return await cubeHandlers.meta(req, res)
        }
        break
    }

    res.status(404).json({ error: 'Endpoint not found' })
  } catch (error) {
    res.status(500).json({ error: error.message })
  }
}
```

## Security Context Patterns

### Session-Based Authentication

```typescript
extractSecurityContext: async (req) => {
  const session = await getSession(req)

  if (!session) {
    throw new Error('Unauthorized: No session')
  }

  return {
    organisationId: session.organisationId,
    userId: session.userId,
    role: session.role
  }
}
```

### JWT Token Authentication

```typescript
import jwt from 'jsonwebtoken'

extractSecurityContext: async (req) => {
  const authHeader = req.headers.authorization
  const token = authHeader?.replace('Bearer ', '')

  if (!token) {
    throw new Error('Unauthorized: No token provided')
  }

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET)

    return {
      organisationId: payload.orgId,
      userId: payload.sub,
      tenantId: payload.tenantId
    }
  } catch (error) {
    throw new Error('Unauthorized: Invalid token')
  }
}
```

### API Key Authentication

```typescript
extractSecurityContext: async (req) => {
  const apiKey = req.headers['x-api-key']

  if (!apiKey) {
    throw new Error('Unauthorized: No API key')
  }

  // Lookup API key in database
  const keyInfo = await db
    .select()
    .from(apiKeys)
    .where(eq(apiKeys.key, apiKey))
    .limit(1)

  if (!keyInfo[0]) {
    throw new Error('Unauthorized: Invalid API key')
  }

  return {
    organisationId: keyInfo[0].organisationId,
    userId: keyInfo[0].userId,
    scope: keyInfo[0].scope
  }
}
```

### Multi-Tenant with Sub-domains

```typescript
extractSecurityContext: async (req) => {
  const host = req.headers.host
  const subdomain = host?.split('.')[0]

  // Lookup organization by subdomain
  const org = await db
    .select()
    .from(organisations)
    .where(eq(organisations.subdomain, subdomain))
    .limit(1)

  if (!org[0]) {
    throw new Error('Invalid subdomain')
  }

  // Also validate user session
  const session = await getSession(req)

  return {
    organisationId: org[0].id,
    userId: session?.userId,
    tenantId: org[0].tenantId
  }
}
```

## Available Endpoints

All adapters expose these Cube.js-compatible endpoints:

### POST /cubejs-api/v1/load

Execute semantic queries and return results.

```typescript
// Request
POST /cubejs-api/v1/load
Content-Type: application/json
Authorization: Bearer <token>

{
  "measures": ["Employees.count"],
  "dimensions": ["Departments.name"]
}

// Response
{
  "data": [
    {
      "Departments.name": "Engineering",
      "Employees.count": 50
    }
  ],
  "annotation": { ... },
  "requestId": "req-123",
  "slowQuery": false
}
```

### POST /cubejs-api/v1/sql

Generate SQL without executing (dry-run).

```typescript
// Request
POST /cubejs-api/v1/sql
Content-Type: application/json
Authorization: Bearer <token>

{
  "measures": ["Employees.count"],
  "dimensions": ["Departments.name"]
}

// Response
{
  "sql": {
    "sql": ["SELECT departments.name, COUNT(employees.id) FROM ..."],
    "params": ["org-123"]
  }
}
```

### GET /cubejs-api/v1/meta

Get cube metadata (dimensions, measures, types).

```typescript
// Request
GET /cubejs-api/v1/meta
Authorization: Bearer <token>

// Response
{
  "cubes": [
    {
      "name": "Employees",
      "title": "Employees",
      "measures": [...],
      "dimensions": [...]
    }
  ]
}
```

## Environment Configuration

```bash
# .env file

# Database connection
DATABASE_URL=postgresql://user:password@localhost:5432/mydb

# JWT authentication
JWT_SECRET=your-secret-key

# API configuration
CUBEJS_API_SECRET=api-secret-key
PORT=3000

# Multi-database support
DB_TYPE=postgres  # or mysql, sqlite
```

## Complete Example: Express with TypeScript

```typescript
// src/server.ts
import express from 'express'
import cors from 'cors'
import helmet from 'helmet'
import { createCubeApi } from 'drizzle-cube/adapters/express'
import { initializeSemanticLayer } from './semantic-layer'
import { authenticateJWT } from './middleware/auth'

const app = express()

// Middleware
app.use(helmet())
app.use(cors())
app.use(express.json())

// Initialize semantic layer
const compiler = await initializeSemanticLayer()

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'ok' })
})

// Cube API (protected)
const cubeApi = createCubeApi({
  extractSecurityContext: async (req) => {
    if (!req.user) {
      throw new Error('Unauthorized')
    }

    return {
      organisationId: req.user.organisationId,
      userId: req.user.id,
      role: req.user.role,
      permissions: req.user.permissions
    }
  },
  semanticLayer: compiler
})

app.use('/cubejs-api/v1', authenticateJWT, cubeApi)

// Error handling
app.use((err, req, res, next) => {
  console.error(err.stack)
  res.status(500).json({
    error: err.message,
    requestId: req.id
  })
})

const PORT = process.env.PORT || 3000
app.listen(PORT, () => {
  console.log(`🚀 Drizzle Cube API running on port ${PORT}`)
})
```

## Best Practices

1. **Always validate security context** - Never trust client input
2. **Use HTTPS in production** - Protect API traffic
3. **Implement rate limiting** - Prevent abuse
4. **Log queries** - Monitor performance and usage
5. **Cache metadata** - Reduce compilation overhead
6. **Handle errors gracefully** - Return meaningful error messages
7. **Validate environment variables** - Check configuration on startup

## Common Pitfalls

- **Missing authentication** - Always protect Cube API endpoints
- **Exposing internal errors** - Sanitize error messages in production
- **No security context validation** - Verify context contains required fields
- **Incorrect CORS configuration** - Configure CORS for your client domains
- **Missing database connection pooling** - Use connection pools for production

## Next Steps

- Define **cubes** with the `cube-definition` skill
- Build **queries** with the `queries` skill
- Create **dashboards** with the `dashboard` skill
- Configure **charts** with the chart-specific skills
