# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a multi-tenant SaaS CRM system designed specifically for landscaping companies. The application manages clients, properties, jobs, scheduling, invoices, and crew assignments.

**Status**: Planning/design phase - implementation not yet started.

## Architecture Decisions

### Tech Stack (Recommended)

**Frontend & Backend**:
- Next.js 14+ with App Router (TypeScript, strict mode)
- Server Actions for backend logic (initially; can migrate to FastAPI if mobile apps are needed)
- Tailwind CSS + ShadCN UI for styling
- React Query (TanStack Query) for client-side caching
- React Hook Form + Zod for form validation

**Database**:
- PostgreSQL 15+ with Row Level Security (RLS)
- Drizzle ORM for type-safe queries
- Redis for caching (Upstash or self-hosted)

**Authentication**:
- Clerk (recommended) or NextAuth v5

**Deployment**:
- Vercel for Next.js application
- Neon or Supabase for PostgreSQL
- Upstash for Redis

### Multi-Tenant Strategy

**Shared Database with Row Level Security (RLS)** - Single database with `tenant_id` column on all tables.

**Critical Requirements**:
1. Every table MUST include `tenant_id` column
2. All tables MUST have RLS policies enabled
3. All queries MUST filter by `tenant_id`
4. Foreign keys MUST include `tenant_id` to prevent cross-tenant references
5. Unique constraints MUST be scoped to `tenant_id`
6. Indexes MUST include `tenant_id` as first column

**Example Schema Pattern**:
```sql
CREATE TABLE clients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    name VARCHAR(255) NOT NULL,
    CONSTRAINT clients_tenant_unique UNIQUE (tenant_id, email)
);

ALTER TABLE clients ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON clients
    FOR ALL TO authenticated
    USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

CREATE INDEX idx_clients_tenant_id ON clients(tenant_id);
```

**Tenant Context Helper**:
```typescript
// Always use this when accessing database
export async function setTenantContext(tenantId: string, db: PostgresJsDatabase) {
  await db.execute(sql`SET app.current_tenant_id = ${tenantId}`);
}
```

## Project Structure

### Route Organization (Next.js App Router)

- `app/(auth)/` - Authentication pages (login, signup, forgot-password)
- `app/(dashboard)/` - Main authenticated application routes
  - `/clients` - Client and property management
  - `/jobs` - Job tracking and management
  - `/schedule` - Calendar and daily scheduling
  - `/invoices` - Invoice generation and payment tracking
  - `/reports` - Revenue and performance analytics
  - `/settings` - User, team, and billing settings
- `app/(admin)/` - Super-admin tenant management
- `app/actions/` - Server Actions grouped by resource (clients.ts, jobs.ts, etc.)
- `app/api/` - API routes for webhooks and cron jobs

### Component Organization

Components are organized by feature domain:
- `components/ui/` - ShadCN UI base components
- `components/forms/` - Form components (client-form.tsx, job-form.tsx, etc.)
- `components/clients/` - Client-specific components
- `components/jobs/` - Job-specific components
- `components/schedule/` - Scheduling components
- `components/invoices/` - Invoice components
- `components/shared/` - Shared/common components

### Library Structure

- `lib/db/schema/` - Database schemas (one file per table, exported from index.ts)
- `lib/db/migrations/` - Database migrations
- `lib/auth/` - Authentication utilities (session, permissions, tenant-context)
- `lib/validations/` - Zod validation schemas
- `lib/services/` - Business logic services
- `lib/hooks/` - Custom React hooks
- `lib/utils/` - Utility functions

## Database Schema

### Core Tables

**Tenants & Users**:
- `tenants` - Company/organization records
- `users` - User accounts with roles and permissions
- `team_members` - Junction table for user-tenant relationships

**Client Management**:
- `clients` - Customer records
- `properties` - Client properties/locations
- `contacts` - Client contact information

**Job Management**:
- `jobs` - Job records with status tracking
- `job_tasks` - Individual tasks within jobs
- `job_visits` - Track site visits
- `job_attachments` - Photos and files

**Scheduling**:
- `schedules` - Daily crew assignments
- `crew_members` - Crew composition and availability

**Billing**:
- `invoices` - Invoice records
- `invoice_line_items` - Line items on invoices
- `payments` - Payment tracking

**System**:
- `activity_log` - Audit trail for all actions

### Schema Organization

Each table is defined in its own file under `lib/db/schema/`:
```
lib/db/schema/
├── index.ts           # Exports all schemas
├── tenants.ts
├── users.ts
├── clients.ts
├── properties.ts
├── jobs.ts
├── invoices.ts
├── schedules.ts
└── activity-log.ts
```

## Development Commands

### Initial Setup (when implementation begins)

```bash
# Create Next.js project
npx create-next-app@latest landscaping-crm --typescript --tailwind --app
cd landscaping-crm

# Install dependencies
npm install drizzle-orm postgres
npm install -D drizzle-kit

# Initialize ShadCN UI
npx shadcn-ui@latest init
```

### Database Operations

```bash
# Generate migration from schema changes
npx drizzle-kit generate:pg

# Push schema changes to database
npx drizzle-kit push:pg

# Open Drizzle Studio (database GUI)
npx drizzle-kit studio

# Run seed data
npm run db:seed
```

### Development Workflow

```bash
# Run development server
npm run dev

# Run linting
npm run lint

# Run type checking
npm run type-check

# Run tests
npm run test

# Run E2E tests
npm run test:e2e
```

## Server Actions Pattern

Server Actions should be organized by resource in `app/actions/`:

```typescript
// app/actions/clients.ts
'use server'

import { db } from '@/lib/db'
import { clients } from '@/lib/db/schema'
import { getCurrentUser, getTenantId } from '@/lib/auth'
import { eq } from 'drizzle-orm'
import { revalidatePath } from 'next/cache'

export async function getClients() {
  const tenantId = await getTenantId()

  return await db
    .select()
    .from(clients)
    .where(eq(clients.tenantId, tenantId))
    .orderBy(clients.createdAt)
}

export async function createClient(data: ClientInput) {
  const user = await getCurrentUser()
  const tenantId = await getTenantId()

  // Permission check
  if (!user.permissions.includes('clients.create')) {
    throw new Error('Unauthorized')
  }

  const [client] = await db
    .insert(clients)
    .values({
      ...data,
      tenantId,
      createdBy: user.id
    })
    .returning()

  revalidatePath('/clients')
  return client
}
```

## Testing Requirements

### Tenant Isolation Tests

**Critical**: Always test that tenant A cannot access tenant B's data:

```typescript
describe('Tenant Isolation', () => {
  it('should not return data from other tenants', async () => {
    const tenantA = await createTenant()
    const clientA = await createClient(tenantA.id)

    const tenantB = await createTenant()
    setContext(tenantB.id)

    const result = await getClient(clientA.id)
    expect(result).toBeNull() // MUST not find it
  })
})
```

## Migration Path

The recommended implementation approach:

**Phase 1 (MVP - 8 weeks)**:
1. Next.js Server Actions only
2. Vercel Cron for simple scheduled tasks
3. Focus on: Client management, Job tracking, Basic scheduling, Invoice generation

**Phase 2 (Enhancement)**:
- Add Inngest/Vercel Queue for background jobs
- Implement advanced scheduling and route optimization

**Phase 3 (Scale)**:
- Add dedicated worker service if needed (for long-running tasks)
- Consider FastAPI backend if mobile apps are required

## Important Constraints

### Security Checklist

Before any production deployment:
- [ ] All tables have `tenant_id` column
- [ ] All tables have RLS policies enabled
- [ ] All queries include tenant_id in WHERE clause
- [ ] Middleware validates user belongs to requested tenant
- [ ] Foreign keys include tenant_id
- [ ] Unique constraints scoped to tenant_id
- [ ] Indexes include tenant_id as first column
- [ ] API routes validate tenant context
- [ ] Tests verify tenant isolation

### Performance Considerations

**Indexing Strategy**:
```sql
-- Composite indexes with tenant_id first
CREATE INDEX idx_clients_tenant_email ON clients(tenant_id, email);
CREATE INDEX idx_jobs_tenant_status ON jobs(tenant_id, status);

-- Partial indexes for common queries
CREATE INDEX idx_active_jobs ON jobs(tenant_id, status)
  WHERE status IN ('scheduled', 'in_progress');
```

**Caching**:
- Cache keys must include tenant_id: `tenant:${tenantId}:clients:${clientId}`
- Invalidation must be tenant-scoped

## Configuration Files

### Drizzle Config
```typescript
// drizzle.config.ts
import type { Config } from 'drizzle-kit'

export default {
  schema: './lib/db/schema/index.ts',
  out: './lib/db/migrations',
  driver: 'pg',
  dbCredentials: {
    connectionString: process.env.DATABASE_URL!,
  },
} satisfies Config
```

### Environment Variables

Required for development:
```env
DATABASE_URL=postgresql://...
REDIS_URL=redis://...
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_...
CLERK_SECRET_KEY=sk_...
```

## Key Features to Implement

### MVP Features (Priority Order):
1. Client & Property Management
2. Job Tracking (with status: scheduled, in_progress, completed, cancelled)
3. Basic Scheduling (calendar view, crew assignment)
4. Invoice Generation & Payment Tracking
5. User Management & RBAC
6. Basic Dashboard (stats, revenue charts, upcoming jobs)

### Post-MVP Features:
- Route optimization
- Automated reminders
- Customer portal
- Equipment & materials tracking
- Time tracking
- Mobile app
- Integrations (QuickBooks, Stripe)
