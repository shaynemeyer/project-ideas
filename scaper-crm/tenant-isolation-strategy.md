# Multi-Tenant Isolation Strategy

## Overview
For this landscaping CRM SaaS, we need to choose the right tenant isolation strategy. Here are the three main approaches:

## 1. Shared Database with Tenant Column (RECOMMENDED)

### Pros:
- **Simple to implement**: Just add `tenant_id` to all tables
- **Cost-effective**: Single database instance
- **Easy to scale initially**: PostgreSQL can handle thousands of tenants
- **Cross-tenant analytics**: Easy to run queries across all tenants
- **Easier backups**: One database to manage
- **Connection pooling**: More efficient resource usage

### Cons:
- **Security concerns**: Risk of tenant data leakage if queries are wrong
- **Single point of failure**: All tenants affected by downtime
- **Performance**: Large tables may slow down over time
- **Cannot customize schema per tenant**

### Implementation:
```sql
-- All tables have tenant_id
CREATE TABLE clients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id UUID NOT NULL REFERENCES tenants(id),
    name VARCHAR(255) NOT NULL,
    -- other fields
    CONSTRAINT clients_tenant_unique UNIQUE (tenant_id, email)
);

-- Row Level Security (RLS) for automatic filtering
ALTER TABLE clients ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON clients
    FOR ALL
    TO authenticated
    USING (tenant_id = current_setting('app.current_tenant_id')::UUID);

-- Indexes for performance
CREATE INDEX idx_clients_tenant_id ON clients(tenant_id);
```

## 2. Separate Schema per Tenant

### Pros:
- **Better isolation**: Schemas are separate namespaces
- **Can customize per tenant**: Different columns/tables if needed
- **Easier to backup individual tenants**
- **Better security**: Harder to accidentally query wrong tenant

### Cons:
- **More complex**: Need to manage multiple schemas
- **Migration complexity**: Must run migrations on all schemas
- **Connection management**: Need to set search_path per request
- **Cross-tenant queries harder**
- **Connection pool less efficient**

### Implementation:
```sql
-- Create schema per tenant
CREATE SCHEMA tenant_abc123;
CREATE SCHEMA tenant_def456;

-- Each schema has identical structure
CREATE TABLE tenant_abc123.clients (
    id UUID PRIMARY KEY,
    name VARCHAR(255),
    -- no tenant_id needed!
);

-- Set schema at connection time
SET search_path TO tenant_abc123;
```

## 3. Separate Database per Tenant

### Pros:
- **Maximum isolation**: Complete separation
- **Can scale databases independently**
- **Easy to offboard**: Just drop the database
- **Custom configurations per tenant** (extensions, parameters)

### Cons:
- **Very complex**: Managing hundreds/thousands of databases
- **Expensive**: More infrastructure costs
- **Migration nightmare**: Must migrate every database
- **Cross-tenant analytics impossible**
- **Connection pool per database**: Resource intensive

## Recommendation: Shared Database with RLS

**For your landscaping CRM SaaS, I recommend Option 1: Shared Database with Row Level Security**

### Reasons:
1. **Scale**: You'll likely have 10-500 tenants initially, perfect for shared DB
2. **Cost**: Much cheaper to operate
3. **Simplicity**: Easier development and maintenance
4. **Performance**: PostgreSQL can easily handle this with proper indexing
5. **Security**: RLS provides automatic tenant filtering at the database level

### When to Consider Other Options:
- **Separate Schema**: When you have 1000+ tenants or need per-tenant customization
- **Separate Database**: Enterprise clients who demand complete isolation and pay premium

## Implementation Best Practices

### 1. Always Use Tenant Context
```typescript
// Middleware sets tenant context
export async function setTenantContext(tenantId: string, db: PostgresJsDatabase) {
  await db.execute(sql`SET app.current_tenant_id = ${tenantId}`);
}
```

### 2. Drizzle ORM with Tenant Filtering
```typescript
// Create a helper that always includes tenant_id
export function withTenantId<T>(tenantId: string) {
  return (query: any) => query.where(eq(schema.tenantId, tenantId));
}

// Usage
const clients = await db
  .select()
  .from(schema.clients)
  .where(eq(schema.clients.tenantId, tenantId));
```

### 3. Foreign Key Constraints
```sql
-- Always include tenant_id in compound foreign keys
ALTER TABLE jobs ADD CONSTRAINT fk_jobs_client 
  FOREIGN KEY (tenant_id, client_id) 
  REFERENCES clients(tenant_id, id);
```

### 4. Unique Constraints
```sql
-- Unique per tenant, not globally
CONSTRAINT uk_client_email UNIQUE (tenant_id, email)
```

### 5. Testing Tenant Isolation
```typescript
// Always test that tenant A cannot access tenant B's data
describe('Tenant Isolation', () => {
  it('should not return data from other tenants', async () => {
    // Create data for tenant A
    const tenantA = await createTenant();
    const clientA = await createClient(tenantA.id);
    
    // Try to access from tenant B context
    const tenantB = await createTenant();
    setContext(tenantB.id);
    
    const result = await getClient(clientA.id);
    expect(result).toBeNull(); // Should not find it!
  });
});
```

## Performance Optimization

### Indexing Strategy
```sql
-- Composite indexes with tenant_id first
CREATE INDEX idx_clients_tenant_email ON clients(tenant_id, email);
CREATE INDEX idx_jobs_tenant_status ON jobs(tenant_id, status);
CREATE INDEX idx_jobs_tenant_scheduled ON jobs(tenant_id, scheduled_date);

-- Partial indexes for common queries
CREATE INDEX idx_active_jobs ON jobs(tenant_id, status) 
  WHERE status IN ('scheduled', 'in_progress');
```

### Partitioning (Future)
```sql
-- When you have millions of records, consider partitioning
CREATE TABLE jobs (
    id UUID,
    tenant_id UUID,
    -- other fields
) PARTITION BY LIST (tenant_id);

-- Create partition for large tenants
CREATE TABLE jobs_tenant_abc123 PARTITION OF jobs
  FOR VALUES IN ('abc123-uuid-here');
```

### Caching Strategy
```typescript
// Cache keys include tenant_id
const cacheKey = `tenant:${tenantId}:clients:${clientId}`;
const cached = await redis.get(cacheKey);

// Invalidate cache for specific tenant
await redis.del(`tenant:${tenantId}:*`);
```

## Security Checklist

- [ ] All tables have `tenant_id` column
- [ ] All tables have RLS policies enabled
- [ ] All queries include tenant_id in WHERE clause
- [ ] Middleware validates user belongs to requested tenant
- [ ] Foreign keys include tenant_id to prevent cross-tenant references
- [ ] Unique constraints are scoped to tenant_id
- [ ] Indexes include tenant_id as first column
- [ ] API routes validate tenant context before processing
- [ ] Background jobs include tenant context
- [ ] Tests verify tenant isolation
- [ ] Audit logs include tenant_id
- [ ] File uploads are scoped to tenant
- [ ] WebSocket connections validate tenant context
