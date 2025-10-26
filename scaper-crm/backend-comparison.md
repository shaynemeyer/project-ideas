# Backend Technology Stack Comparison

## Option A: Next.js 14+ Server Actions

### Architecture
```
Client Components → Server Actions → Drizzle ORM → PostgreSQL
                    ↓
                 Server Components (direct DB access)
```

### Pros
✅ **Simplicity**: No separate backend to deploy
✅ **Type Safety**: End-to-end TypeScript
✅ **Performance**: Server components can directly query DB
✅ **Developer Experience**: Single codebase, no API contracts
✅ **Deployment**: Single deployment (Vercel, etc.)
✅ **Modern**: Uses latest Next.js App Router features
✅ **Real-time**: Easy to integrate with Server-Sent Events
✅ **Caching**: Built-in Next.js caching strategies

### Cons
❌ **Vendor Lock-in**: Tied to Next.js/Vercel ecosystem
❌ **Limited Background Jobs**: Need external service or Vercel Cron
❌ **Scaling**: Cannot scale API independently from frontend
❌ **Mobile Apps**: Cannot reuse backend easily
❌ **WebSockets**: Limited support, need separate service
❌ **Long-running Tasks**: Lambda timeout limits (300s max on Vercel)

### Code Example
```typescript
// app/actions/clients.ts
'use server'

import { db } from '@/lib/db'
import { clients } from '@/lib/db/schema'
import { getCurrentUser, getTenantId } from '@/lib/auth'
import { eq, and } from 'drizzle-orm'
import { revalidatePath } from 'next/cache'

export async function getClients() {
  const user = await getCurrentUser()
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

// app/clients/page.tsx
import { getClients } from '@/app/actions/clients'
import { ClientList } from '@/components/ClientList'

export default async function ClientsPage() {
  const clients = await getClients() // Direct server-side call!
  
  return <ClientList clients={clients} />
}
```

### When to Choose This
- You're building web-only (no mobile app planned)
- You want fastest development speed
- Team is comfortable with Next.js
- You don't need complex background jobs
- You're okay with Vercel/Next.js ecosystem

---

## Option B: FastAPI (Python) Backend

### Architecture
```
Next.js Client → REST/GraphQL API → FastAPI → SQLAlchemy/Drizzle-like → PostgreSQL
                                     ↓
                                  Celery Workers → Redis → Background Jobs
```

### Pros
✅ **Flexibility**: Can use any frontend (Next.js, mobile, etc.)
✅ **Scalability**: Scale API independently
✅ **Background Jobs**: Native Celery/RQ integration
✅ **Long-running Tasks**: No timeout limits
✅ **WebSockets**: Native support with websockets
✅ **Python Ecosystem**: ML, data processing, PDF generation
✅ **API-First**: Easy to add mobile apps later
✅ **Microservices**: Can split into multiple services

### Cons
❌ **Complexity**: Two codebases to maintain
❌ **Deployment**: Separate frontend and backend deployments
❌ **Type Safety**: Need to maintain API contracts (OpenAPI)
❌ **Development Speed**: Slower initial development
❌ **CORS**: Need to handle cross-origin requests
❌ **Infrastructure**: More complex (API server + worker processes)

### Code Example
```python
# app/api/clients.py
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.core.deps import get_db, get_current_user, get_tenant_context
from app.models.client import Client
from app.schemas.client import ClientCreate, ClientResponse
from typing import List

router = APIRouter()

@router.get("/clients", response_model=List[ClientResponse])
async def get_clients(
    db: Session = Depends(get_db),
    user = Depends(get_current_user),
    tenant_id: str = Depends(get_tenant_context)
):
    """Get all clients for the current tenant"""
    clients = db.query(Client).filter(
        Client.tenant_id == tenant_id
    ).all()
    
    return clients

@router.post("/clients", response_model=ClientResponse, status_code=201)
async def create_client(
    client_data: ClientCreate,
    db: Session = Depends(get_db),
    user = Depends(get_current_user),
    tenant_id: str = Depends(get_tenant_context)
):
    """Create a new client"""
    if "clients.create" not in user.permissions:
        raise HTTPException(status_code=403, detail="Unauthorized")
    
    client = Client(
        **client_data.dict(),
        tenant_id=tenant_id,
        created_by=user.id
    )
    
    db.add(client)
    db.commit()
    db.refresh(client)
    
    # Trigger background job
    from app.tasks.notifications import send_client_welcome
    send_client_welcome.delay(client.id)
    
    return client

# app/core/deps.py
from fastapi import Depends, HTTPException, Header
from sqlalchemy.orm import Session
from app.core.database import get_session
from app.core.auth import decode_token

async def get_current_user(
    authorization: str = Header(...),
    db: Session = Depends(get_session)
):
    """Extract and validate user from JWT token"""
    try:
        token = authorization.replace("Bearer ", "")
        payload = decode_token(token)
        user_id = payload.get("sub")
        
        # Get user from DB with tenant
        user = db.query(User).filter(User.id == user_id).first()
        if not user:
            raise HTTPException(status_code=401, detail="Invalid user")
        
        return user
    except Exception as e:
        raise HTTPException(status_code=401, detail="Invalid token")

async def get_tenant_context(
    user = Depends(get_current_user)
) -> str:
    """Get tenant ID from authenticated user"""
    return user.tenant_id

# Frontend API client
// lib/api/client.ts
import { useQuery, useMutation } from '@tanstack/react-query'

const API_URL = process.env.NEXT_PUBLIC_API_URL

async function fetchClients() {
  const res = await fetch(`${API_URL}/clients`, {
    headers: {
      'Authorization': `Bearer ${getToken()}`
    }
  })
  return res.json()
}

export function useClients() {
  return useQuery({
    queryKey: ['clients'],
    queryFn: fetchClients
  })
}

export function useCreateClient() {
  return useMutation({
    mutationFn: async (data: ClientInput) => {
      const res = await fetch(`${API_URL}/clients`, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${getToken()}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(data)
      })
      return res.json()
    }
  })
}

// Usage in component
// components/ClientList.tsx
'use client'

export function ClientList() {
  const { data: clients, isLoading } = useClients()
  const createClient = useCreateClient()
  
  // ... render logic
}
```

### When to Choose This
- You plan to build mobile apps
- You need complex background processing
- You need long-running tasks (> 5 minutes)
- You want maximum flexibility and scalability
- Team has Python expertise
- You need ML/AI features

---

## Recommended Approach: **Next.js Server Actions + Background Job Service**

### The Best of Both Worlds

**Primary Architecture**: Next.js 14+ with Server Actions
**Background Jobs**: Separate lightweight service (Node.js + BullMQ or Python + Celery)

### Why This Hybrid?
1. **Fast Development**: Server Actions for 90% of features
2. **Background Jobs**: Dedicated service for heavy lifting
3. **Type Safety**: Stay in TypeScript ecosystem
4. **Scalability**: Can scale background workers independently
5. **Simple**: Still mostly single codebase

### Implementation
```typescript
// Server Actions for most operations
// app/actions/jobs.ts
'use server'

export async function createJob(data: JobInput) {
  const tenantId = await getTenantId()
  
  // Create job in database
  const [job] = await db.insert(jobs).values({
    ...data,
    tenantId
  }).returning()
  
  // Enqueue background tasks via API
  await fetch(process.env.WORKER_API_URL + '/tasks/job-created', {
    method: 'POST',
    body: JSON.stringify({ jobId: job.id, tenantId })
  })
  
  return job
}

// Separate lightweight worker service
// worker-service/src/tasks/job-created.ts
import { Queue } from 'bullmq'

const jobQueue = new Queue('jobs', {
  connection: redis
})

export async function handleJobCreated(jobId: string, tenantId: string) {
  // Send notifications
  await jobQueue.add('send-notifications', { jobId, tenantId })
  
  // Generate PDF
  await jobQueue.add('generate-pdf', { jobId, tenantId })
  
  // Update integrations
  await jobQueue.add('sync-external', { jobId, tenantId })
}
```

### Architecture Diagram
```
┌─────────────────────┐
│   Next.js App       │
│  (Server Actions)   │
│                     │
│  - CRUD operations  │
│  - Real-time UI     │
│  - Direct DB access │
└──────────┬──────────┘
           │
           │ HTTP (for job triggers)
           │
           ▼
┌─────────────────────┐      ┌──────────────┐
│  Worker Service     │◄─────┤    Redis     │
│  (Node.js/BullMQ)   │      │   (Queue)    │
│                     │      └──────────────┘
│  - Email sending    │
│  - PDF generation   │
│  - Data exports     │
│  - Integrations     │
└─────────────────────┘
```

## Final Recommendation

### Start with: **Next.js Server Actions Only**

For your landscaping CRM MVP:
1. Use Next.js 14+ with Server Actions
2. Use Vercel Cron for simple scheduled tasks
3. Use Vercel Queue or Inngest for async jobs (if needed)
4. Add separate worker service only when you need:
   - Long-running tasks (> 5 min)
   - Heavy background processing
   - Complex job orchestration

### Migration Path
```
Phase 1: Next.js Server Actions only
  ↓
Phase 2: Add Inngest/Vercel Queue for background jobs
  ↓
Phase 3: Add dedicated worker service if needed
  ↓
Phase 4: Consider FastAPI if you need mobile apps
```

### Decision Matrix

| Requirement | Next.js SA | FastAPI | Hybrid |
|-------------|-----------|---------|--------|
| Fast Development | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Type Safety | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐⭐ |
| Background Jobs | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Mobile App Support | ⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐ |
| Scalability | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |
| Deployment Complexity | ⭐⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐⭐⭐ |
| Team Learning Curve | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| Infrastructure Cost | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |

**Legend**: ⭐ = Poor, ⭐⭐⭐ = Good, ⭐⭐⭐⭐⭐ = Excellent
