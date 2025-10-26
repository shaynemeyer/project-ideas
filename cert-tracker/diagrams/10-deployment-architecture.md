# Deployment Architecture

This diagram shows the recommended production deployment architecture on Vercel with supporting services.

```mermaid
graph TB
    subgraph "Client Devices"
        A[Web Browser]
        B[Mobile Browser]
    end
    
    subgraph "CDN Layer"
        C[Vercel Edge Network]
        D[Static Assets Cache]
    end
    
    subgraph "Application Layer - Vercel"
        E[Next.js App<br/>Serverless Functions]
        F[API Routes]
        G[Background Jobs<br/>Vercel Cron]
    end
    
    subgraph "Database Layer"
        H[(PostgreSQL<br/>Supabase/Neon)]
        I[Connection Pool]
    end
    
    subgraph "External Services"
        J[Email Service]
        K[Slack Webhooks]
        L[Monitoring<br/>Sentry/DataDog]
    end
    
    A --> C
    B --> C
    C --> D
    C --> E
    D --> E
    E --> F
    F --> I
    I --> H
    G --> F
    F --> J
    F --> K
    E --> L
    F --> L
    
    style A fill:#e1f5ff
    style B fill:#e1f5ff
    style C fill:#fff4e6
    style H fill:#f0f0f0
```

## Deployment Options

### Option 1: Vercel (Recommended)

**Advantages:**
- Seamless Next.js integration
- Built-in CI/CD
- Edge network for global performance
- Serverless functions auto-scale
- Built-in cron jobs support
- Zero-downtime deployments

**Configuration:**

```json
// vercel.json
{
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "regions": ["iad1", "sfo1", "lhr1"],
  "crons": [
    {
      "path": "/api/cron/check-certificates",
      "schedule": "0 */6 * * *"
    }
  ],
  "env": {
    "DATABASE_URL": "@database-url",
    "JWT_SECRET": "@jwt-secret",
    "CRON_SECRET": "@cron-secret",
    "RESEND_API_KEY": "@resend-api-key"
  }
}
```

### Option 2: Docker + Cloud Provider

**Docker Deployment:**

```dockerfile
# Dockerfile
FROM node:20-alpine AS base

# Dependencies
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json package-lock.json* ./
RUN npm ci

# Builder
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED 1

RUN npm run build

# Runner
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/ssl_tracker
      - JWT_SECRET=${JWT_SECRET}
      - CRON_SECRET=${CRON_SECRET}
    depends_on:
      - db
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=ssl_tracker
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    ports:
      - "5432:5432"
    restart: unless-stopped

  cron:
    build: .
    command: ["node", "cron.js"]
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/ssl_tracker
      - CRON_SECRET=${CRON_SECRET}
    depends_on:
      - db
      - app
    restart: unless-stopped

volumes:
  postgres_data:
```

## Database Hosting Options

### Option 1: Supabase

**Advantages:**
- Free tier available
- Managed PostgreSQL
- Built-in connection pooling
- Automatic backups
- Real-time subscriptions (optional)

**Setup:**
```bash
# Connect to Supabase
DATABASE_URL=postgresql://postgres:[PASSWORD]@db.[PROJECT-ID].supabase.co:5432/postgres
```

### Option 2: Neon

**Advantages:**
- Serverless PostgreSQL
- Auto-scaling compute
- Branching for dev/staging
- Pay per usage
- Generous free tier

**Setup:**
```bash
# Connect to Neon
DATABASE_URL=postgresql://[USER]:[PASSWORD]@[ENDPOINT].neon.tech/[DATABASE]?sslmode=require
```

### Option 3: Railway

**Advantages:**
- Simple setup
- Includes PostgreSQL
- Built-in monitoring
- Automatic SSL

**Setup:**
```bash
# Railway provides DATABASE_URL automatically
# Just connect your repo
```

## Environment Configuration

### Production Environment Variables

```env
# .env.production

# Database
DATABASE_URL=postgresql://...

# Authentication
JWT_SECRET=your-super-secret-jwt-key-min-32-characters
NEXTAUTH_SECRET=your-nextauth-secret-key
NEXTAUTH_URL=https://yourdomain.com

# Cron Security
CRON_SECRET=your-cron-secret-key

# Email Service
RESEND_API_KEY=re_...
EMAIL_FROM=alerts@yourdomain.com

# Slack (Optional)
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/...

# Monitoring
SENTRY_DSN=https://...@sentry.io/...
NEXT_PUBLIC_SENTRY_DSN=https://...@sentry.io/...

# Analytics (Optional)
NEXT_PUBLIC_GA_ID=G-...
```

### Development Environment

```env
# .env.local

# Database
DATABASE_URL=postgresql://localhost:5432/ssl_tracker_dev

# Authentication
JWT_SECRET=dev-secret-key
NEXTAUTH_SECRET=dev-nextauth-secret
NEXTAUTH_URL=http://localhost:3000

# Cron Security
CRON_SECRET=dev-cron-secret

# Email Service (use test mode)
RESEND_API_KEY=re_test_...
EMAIL_FROM=dev@localhost
```

## CI/CD Pipeline

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to Vercel

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run type check
        run: npm run type-check
      
      - name: Run tests
        run: npm test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

## Monitoring & Observability

### Sentry Integration

```typescript
// lib/sentry.ts
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: 1.0,
  integrations: [
    new Sentry.BrowserTracing(),
  ],
});

// Use in API routes
export async function GET(request: Request) {
  try {
    // Your code
  } catch (error) {
    Sentry.captureException(error);
    throw error;
  }
}
```

### Logging

```typescript
// lib/logger.ts
import pino from 'pino';

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => {
      return { level: label };
    },
  },
  timestamp: pino.stdTimeFunctions.isoTime,
});

// Usage
logger.info({ certificateId: cert.id }, 'Certificate checked');
logger.error({ error, certificateId: cert.id }, 'Check failed');
```

### Uptime Monitoring

**Services to use:**
- UptimeRobot (free tier available)
- Better Uptime
- Pingdom
- StatusCake

**Endpoints to monitor:**
```
https://yourdomain.com/api/health
https://yourdomain.com/
```

### Health Check Endpoint

```typescript
// app/api/health/route.ts
import { db } from '@/lib/db';

export async function GET() {
  try {
    // Check database connection
    await db.execute('SELECT 1');
    
    // Check critical services
    const checks = {
      database: 'ok',
      timestamp: new Date().toISOString(),
      version: process.env.npm_package_version,
    };
    
    return Response.json(checks, { status: 200 });
  } catch (error) {
    return Response.json(
      { error: 'Health check failed', database: 'error' },
      { status: 503 }
    );
  }
}
```

## Scaling Considerations

### Database Connection Pooling

```typescript
// lib/db.ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';

const connectionString = process.env.DATABASE_URL!;

// Use connection pooling in serverless
const client = postgres(connectionString, {
  max: 10, // Max connections
  idle_timeout: 20,
  connect_timeout: 10,
});

export const db = drizzle(client);
```

### Rate Limiting

```typescript
// Use Upstash Redis for distributed rate limiting
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(100, '1 h'),
  analytics: true,
});
```

### Caching Strategy

```typescript
// Use Next.js caching
export const revalidate = 3600; // Revalidate every hour

// Or use Redis for distributed caching
import { Redis } from '@upstash/redis';

const redis = Redis.fromEnv();

export async function getCertificates() {
  const cached = await redis.get('certificates');
  if (cached) return cached;
  
  const certs = await db.select().from(certificates);
  await redis.setex('certificates', 300, JSON.stringify(certs));
  
  return certs;
}
```

## Security Configuration

### HTTPS/SSL
- Vercel provides automatic HTTPS
- Custom domains get free SSL certificates
- Automatic certificate renewal

### Security Headers

```typescript
// next.config.js
const securityHeaders = [
  {
    key: 'X-DNS-Prefetch-Control',
    value: 'on'
  },
  {
    key: 'Strict-Transport-Security',
    value: 'max-age=63072000; includeSubDomains; preload'
  },
  {
    key: 'X-Frame-Options',
    value: 'SAMEORIGIN'
  },
  {
    key: 'X-Content-Type-Options',
    value: 'nosniff'
  },
  {
    key: 'Referrer-Policy',
    value: 'origin-when-cross-origin'
  }
];

module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: securityHeaders,
      },
    ];
  },
};
```

## Backup Strategy

### Database Backups
```bash
# Automated daily backups (Supabase/Neon handle this)
# Manual backup script
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d).sql

# Restore
psql $DATABASE_URL < backup_20241026.sql
```

### Code Backups
- Git repository (GitHub/GitLab)
- Vercel keeps deployment history
- Tag releases for easy rollback

```bash
# Tag a release
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```
