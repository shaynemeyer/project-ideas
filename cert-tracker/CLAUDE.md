# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**SSL Certificate Tracking System** - A production web application for monitoring SSL certificate expiration dates across multiple domains with automated checks and multi-channel alerting.

**Type**: Production SaaS Application
**Technology**: Next.js 14+ App Router, PostgreSQL, Drizzle ORM, TypeScript, TailwindCSS, shadcn/ui
**Status**: Planning - Complete architecture design, no implementation yet

## Project Architecture

The system follows a modern serverless architecture with clear separation of concerns:

- **Frontend**: Next.js 14+ with App Router, Server Components, and React Server Actions
- **Backend**: Next.js API Routes with service layer architecture
- **Database**: PostgreSQL with Drizzle ORM (type-safe queries)
- **Authentication**: NextAuth.js or custom JWT with role-based access control (RBAC)
- **Background Jobs**: Cron-triggered certificate checking (via Vercel Cron or external scheduler)
- **Notifications**: Multi-channel (Email via SendGrid/Resend, Slack API, Webhooks)

### Key Architectural Patterns

1. **Service Layer Pattern**: Business logic separated from API routes
   - Certificate Service: SSL checking and validation
   - Alert Service: Threshold evaluation and alert generation
   - Notification Service: Multi-channel notification dispatch
   - User Service: Authentication and authorization

2. **Certificate Status State Machine**: Certificates transition through states:
   - `pending` → `active` → `expiring` → `expired`
   - `error` state for failed checks
   - See `diagrams/11-certificate-status-state-machine.md`

3. **Alert Threshold System**: Configurable per-certificate alert days
   - Default thresholds: [30, 14, 7, 1] days before expiry
   - One-time alerts at each threshold crossing
   - Prevents duplicate notifications

## Database Schema

All tables use UUID primary keys. Key relationships:

- **certificates** (main table): Stores domain, validity dates, status, alert thresholds
  - Unique constraint on `domain` column
  - `alert_thresholds` stored as JSONB array

- **certificate_history**: Audit trail of all checks and certificate changes
  - FK to certificates
  - Records errors for troubleshooting

- **alerts**: Generated alerts with acknowledgment tracking
  - FK to certificates
  - Types: `expiry_warning`, `expired`, `error`

- **users**: Role-based access (admin, editor, viewer)
  - Email is unique
  - `receive_alerts` flag for notification opt-in

- **notification_settings**: Singleton table for system-wide notification config

See `diagrams/02-database-schema-erd.md` for complete ERD.

## API Routes

### Core Endpoints

```
GET    /api/certificates              List all certificates (with filters)
POST   /api/certificates              Add new certificate
GET    /api/certificates/:id          Get certificate details
PUT    /api/certificates/:id          Update certificate
DELETE /api/certificates/:id          Remove certificate
POST   /api/certificates/:id/check    Manual certificate check

GET    /api/alerts                    List alerts (paginated)
PUT    /api/alerts/:id/acknowledge    Mark alert as acknowledged
DELETE /api/alerts/:id                Delete alert

GET    /api/users                     List users
POST   /api/users                     Create user
GET    /api/users/:id                 Get user
PUT    /api/users/:id                 Update user
DELETE /api/users/:id                 Delete user

GET    /api/settings/notifications    Get notification settings
PUT    /api/settings/notifications    Update notification settings

GET    /api/dashboard/stats           Dashboard statistics

GET    /api/cron/check-certificates   Trigger certificate checks (cron secret protected)
```

See `diagrams/03-api-architecture.md` for service layer responsibilities.

## Security & Authentication

### Role-Based Access Control (RBAC)

Three roles with different permission levels:

| Role | Certificates | Alerts | Users | Settings |
|------|--------------|--------|-------|----------|
| **admin** | Full CRUD + manual check | Full CRUD | Full CRUD | Full CRUD |
| **editor** | Full CRUD + manual check | Read + Acknowledge | Read only | Read only |
| **viewer** | Read only | Read only | Read only | Read only |

### Implementation Requirements

- All API routes must use authentication middleware
- Check user role before executing actions using RBAC permissions
- Cron endpoint must validate `CRON_SECRET` from Authorization header
- Passwords must be hashed with bcrypt (10 rounds minimum)
- JWT tokens should expire after 7 days
- Input validation required using Zod schemas

See `diagrams/08-security-authentication-flow.md` for complete auth flow and code examples.

## Certificate Checking Process

1. **Cron Trigger**: Runs every 24 hours (configurable)
2. **Fetch Active Certificates**: Query database for certificates with `is_active = true`
3. **Check Each Certificate**:
   - HTTPS connection to domain:443
   - Extract SSL certificate details
   - Calculate days until expiry
   - Update certificate record
4. **Alert Evaluation**: Check if days remaining crosses any threshold
5. **Notification Dispatch**: Send to all enabled channels (Email, Slack, Webhook)

**Performance**: Parallel certificate checking with 10-second timeout per domain.

See `diagrams/04-certificate-checking-flow.md` for complete sequence diagram.

## Development Workflow (When Implementation Begins)

### Setup Commands
```bash
# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Edit .env.local with your credentials

# Initialize database
npx drizzle-kit generate:pg        # Generate migrations from schema
npx drizzle-kit push:pg            # Push schema to database
npm run db:seed                    # Seed initial data (if available)

# Development server
npm run dev                        # Start at http://localhost:3000
```

### Database Commands
```bash
npx drizzle-kit studio             # Open Drizzle Studio (database GUI)
npx drizzle-kit generate:pg        # Generate new migration
npx drizzle-kit push:pg            # Push schema changes
npx drizzle-kit drop               # Drop database (destructive!)
```

### Testing Commands
```bash
npm run test                       # Run all tests
npm run test:watch                 # Watch mode
npm run test:coverage              # Coverage report
npm run test:auth                  # Auth tests only
npm run test:cert-checker          # Certificate checker tests
```

### Build and Deploy
```bash
npm run build                      # Production build
npm run start                      # Start production server
npm run lint                       # Run ESLint
npm run type-check                 # TypeScript type checking
```

## Critical Implementation Notes

### Drizzle ORM Schema Location
Database schema should be defined in `/src/db/schema.ts` (or similar). All tables must be exported for use in queries.

Example structure:
```typescript
// src/db/schema.ts
export const certificates = pgTable('certificates', { /* ... */ });
export const alerts = pgTable('alerts', { /* ... */ });
export const users = pgTable('users', { /* ... */ });
```

### Certificate Checking Service
The core certificate checker must:
- Use native Node.js `https` module or similar
- Handle timeouts (10 seconds recommended)
- Extract: `valid_from`, `valid_to`, `issuer`, `subject` (common name)
- Calculate days until expiry
- Return structured data or error
- Log all checks to `certificate_history` table

### Alert Logic
Alerts should only fire ONCE per threshold crossing:
- Check if alert already exists for certificate + threshold
- Only create new alert if threshold crossed and no existing alert
- Mark certificate status as `expiring` when first threshold crossed

### Environment Variables Required
```env
DATABASE_URL=postgresql://...
JWT_SECRET=minimum-32-character-secret
CRON_SECRET=secret-for-cron-endpoint
NEXTAUTH_SECRET=nextauth-secret  # if using NextAuth.js
NEXTAUTH_URL=https://your-domain.com
EMAIL_API_KEY=sendgrid-or-resend-key
SLACK_WEBHOOK_URL=https://hooks.slack.com/...  # optional
SENTRY_DSN=https://...  # optional, for error tracking
```

## File Organization (Recommended)

```
cert-tracker/
├── src/
│   ├── app/                      # Next.js App Router
│   │   ├── (auth)/               # Auth layout group
│   │   │   └── login/
│   │   ├── dashboard/
│   │   ├── certificates/
│   │   ├── alerts/
│   │   ├── users/
│   │   ├── settings/
│   │   └── api/                  # API routes
│   │       ├── certificates/
│   │       ├── alerts/
│   │       ├── users/
│   │       ├── settings/
│   │       ├── dashboard/
│   │       └── cron/
│   ├── components/               # React components
│   │   ├── ui/                   # shadcn/ui components
│   │   ├── certificates/
│   │   ├── alerts/
│   │   └── layout/
│   ├── lib/
│   │   ├── services/             # Business logic
│   │   │   ├── certificate.service.ts
│   │   │   ├── alert.service.ts
│   │   │   ├── notification.service.ts
│   │   │   └── user.service.ts
│   │   ├── middleware/
│   │   │   ├── auth.ts
│   │   │   └── rbac.ts
│   │   ├── utils/
│   │   └── validations/          # Zod schemas
│   ├── db/
│   │   ├── schema.ts             # Drizzle schema
│   │   ├── index.ts              # DB connection
│   │   └── migrations/
│   └── types/
├── diagrams/                     # Architecture diagrams (Mermaid)
├── public/
├── drizzle.config.ts
├── next.config.js
├── tailwind.config.ts
└── tsconfig.json
```

## Comprehensive Diagram Documentation

All architectural decisions are documented in the `diagrams/` directory with Mermaid diagrams:

1. **System Architecture** - High-level component overview
2. **Database Schema (ERD)** - Complete entity relationships
3. **API Architecture** - Endpoint structure and service layer
4. **Certificate Checking Flow** - Automated monitoring process
5. **User Interaction Flow** - Page navigation and workflows
6. **Component Hierarchy** - React component structure
7. **Data Flow Diagram** - Data transformation pipeline
8. **Security & Authentication** - Auth flow with code examples
9. **Notification System** - Multi-channel alert architecture
10. **Deployment Architecture** - Infrastructure and hosting
11. **Certificate Status State Machine** - Lifecycle management
12. **Monitoring & Observability** - Logging and metrics

**Always consult these diagrams** before implementing features to ensure consistency with the planned architecture.

See `diagrams/README.md` for detailed index and usage guidance.

## Testing Strategy

### Unit Tests
- Service layer functions (certificate checking, alert generation)
- Utility functions (date calculations, domain validation)
- RBAC permission checks

### Integration Tests
- API endpoints with mocked database
- Authentication flows
- Certificate checking with mocked HTTPS requests

### E2E Tests (Optional)
- Complete user workflows
- Cron job execution
- Notification delivery

## Deployment

### Recommended: Vercel
- Zero-config deployment for Next.js
- Built-in cron job support
- Environment variable management
- Automatic HTTPS

### Alternative: Docker
See `diagrams/10-deployment-architecture.md` for Dockerfile and docker-compose setup.

### Database Hosting Options
- **Supabase** (recommended): PostgreSQL + connection pooling
- **Neon**: Serverless PostgreSQL
- **Railway**: Simple PostgreSQL hosting
- **Self-hosted**: Docker PostgreSQL

### Cron Job Setup
**Vercel**: Add to `vercel.json`:
```json
{
  "crons": [{
    "path": "/api/cron/check-certificates",
    "schedule": "0 0 * * *"
  }]
}
```

**Alternative**: Use external cron service (cron-job.org, EasyCron) to call the endpoint.

## Monitoring and Observability

- **Application Monitoring**: Sentry for error tracking
- **Logging**: Structured logging with Pino
- **Metrics**: Track certificate check success rate, alert delivery rate
- **Health Checks**: `/api/health` endpoint for uptime monitoring

See `diagrams/12-monitoring-observability.md` for complete monitoring architecture.

## Important Implementation Priorities

When starting implementation, follow this order:

1. **Database Setup**: Define Drizzle schema, create migrations
2. **Authentication**: Implement NextAuth.js or JWT auth + RBAC
3. **Certificate Checker**: Core SSL checking service
4. **API Routes**: CRUD endpoints for certificates
5. **Basic UI**: Dashboard and certificate list
6. **Alert System**: Threshold evaluation and alert generation
7. **Notifications**: Email, Slack, Webhook integration
8. **Cron Job**: Automated certificate checking
9. **Advanced UI**: Alert management, user administration
10. **Testing**: Unit and integration tests
11. **Deployment**: Vercel + database hosting
12. **Monitoring**: Sentry, logging, health checks
