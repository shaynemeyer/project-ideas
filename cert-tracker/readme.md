# SSL Certificate Tracker

> **Status**: Planning Phase - Architecture complete, implementation not started

A production-ready web application for monitoring SSL certificate expiration dates across multiple domains with automated checking and multi-channel alerting.

## Overview

SSL Certificate Tracker helps organizations proactively manage certificate renewals by:

- **Automated Monitoring**: Daily checks of all registered SSL certificates
- **Smart Alerting**: Configurable thresholds (default: 30, 14, 7, 1 days before expiry)
- **Multi-Channel Notifications**: Email, Slack, and Webhook integrations
- **Role-Based Access**: Admin, Editor, and Viewer roles with granular permissions
- **Audit Trail**: Complete history of all certificate checks and changes
- **Dashboard Analytics**: Real-time statistics and certificate status overview

## Technology Stack

- **Frontend**: Next.js 14+ with App Router, TypeScript, TailwindCSS, shadcn/ui
- **Backend**: Next.js API Routes with service layer architecture
- **Database**: PostgreSQL with Drizzle ORM (type-safe queries)
- **Authentication**: NextAuth.js or custom JWT with RBAC
- **Background Jobs**: Vercel Cron or external scheduler
- **Notifications**: SendGrid/Resend (Email), Slack API, Webhooks
- **Deployment**: Vercel (recommended) or Docker
- **Monitoring**: Sentry for error tracking, structured logging

## Features

### Core Functionality

- **Certificate Management**: Add, edit, delete, and manually check certificates
- **Automated Checks**: Cron-based daily monitoring of all active certificates
- **Status Tracking**: Real-time certificate status (Active, Expiring, Expired, Error)
- **Alert System**: Configurable per-certificate alert thresholds
- **Notification Dispatch**: Multi-channel alert delivery with retry logic
- **Alert Acknowledgment**: Track which alerts have been reviewed
- **Audit History**: Complete log of certificate changes and checks

### User Management

- **Role-Based Access Control**: Three permission levels
  - **Admin**: Full system access including user and settings management
  - **Editor**: Manage certificates and acknowledge alerts
  - **Viewer**: Read-only access to certificates and alerts
- **User Preferences**: Opt-in/out of alert notifications
- **Authentication**: Secure login with JWT or NextAuth.js

### Administration

- **System Settings**: Configure notification channels globally
- **Dashboard**: Overview statistics and quick insights
- **Notification Testing**: Test email/Slack/webhook delivery
- **Health Monitoring**: System health checks and status endpoints

## Architecture

The system follows a modern serverless architecture with clear separation of concerns:

```
                 Clients
                (Web/Mobile)
                    |
        +-----------v-----------+
        | Next.js Application   |
        | +-------------------+ |
        | | App Router        | |
        | +--------+----------+ |
        | +--------v----------+ |
        | | API Routes        | |
        | +--------+----------+ |
        | +--------v----------+ |
        | | Service Layer     | |
        | | - Cert Service    | |
        | | - Alert Service   | |
        | | - Notif Service   | |
        | +--------+----------+ |
        +-----------+------------+
                    |
        +-----------v-----------+
        |   Drizzle ORM         |
        +-----------+-----------+
                    |
        +-----------v-----------+
        |   PostgreSQL          |
        +-----------------------+
```

### Key Design Patterns

1. **Service Layer**: Business logic separated from API routes for testability
2. **State Machine**: Certificates transition through defined states
3. **Event-Driven Alerts**: Threshold-based alert generation
4. **Audit Logging**: All changes tracked in certificate_history table

## Database Schema

Five main tables with clear relationships:

- **certificates**: Primary table for tracked domains
- **certificate_history**: Audit trail of all checks
- **alerts**: Generated alerts with acknowledgment tracking
- **users**: User accounts with role-based permissions
- **notification_settings**: Global notification configuration

See `diagrams/02-database-schema-erd.md` for complete ERD.

## API Endpoints

### Certificates
- `GET /api/certificates` - List all certificates
- `POST /api/certificates` - Add new certificate
- `GET /api/certificates/:id` - Get certificate details
- `PUT /api/certificates/:id` - Update certificate
- `DELETE /api/certificates/:id` - Remove certificate
- `POST /api/certificates/:id/check` - Manual check

### Alerts
- `GET /api/alerts` - List alerts (paginated)
- `PUT /api/alerts/:id/acknowledge` - Acknowledge alert
- `DELETE /api/alerts/:id` - Delete alert

### Users
- `GET /api/users` - List users
- `POST /api/users` - Create user
- `PUT /api/users/:id` - Update user
- `DELETE /api/users/:id` - Delete user

### Settings
- `GET /api/settings/notifications` - Get notification config
- `PUT /api/settings/notifications` - Update notification config

### Dashboard
- `GET /api/dashboard/stats` - Dashboard statistics

### Cron
- `GET /api/cron/check-certificates` - Trigger checks (protected)

See `diagrams/03-api-architecture.md` for complete API documentation.

## Documentation

This project includes comprehensive architectural documentation in the `diagrams/` directory:

1. **System Architecture Overview** - High-level component diagram
2. **Database Schema (ERD)** - Complete entity relationships
3. **API Architecture** - Endpoint structure and service layer
4. **Certificate Checking Flow** - Automated monitoring sequence
5. **User Interaction Flow** - Page navigation and workflows
6. **Component Hierarchy** - React component structure
7. **Data Flow Diagram** - Data transformation pipeline
8. **Security & Authentication** - Auth flow with code examples
9. **Notification System Architecture** - Multi-channel alerts
10. **Deployment Architecture** - Infrastructure and hosting options
11. **Certificate Status State Machine** - Lifecycle management
12. **Monitoring & Observability** - Logging and metrics strategy

All diagrams use [Mermaid](https://mermaid.js.org/) format and render automatically on GitHub.

See `diagrams/README.md` for detailed index and usage guidance.

## Getting Started

> **Note**: Implementation has not started yet. Commands below are for future reference.

### Prerequisites

- Node.js 18+
- PostgreSQL 14+
- npm or yarn

### Installation

```bash
# Clone repository
git clone <repository-url>
cd cert-tracker

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Edit .env.local with your credentials

# Initialize database
npx drizzle-kit generate:pg
npx drizzle-kit push:pg

# Seed initial data (optional)
npm run db:seed

# Start development server
npm run dev
```

Visit `http://localhost:3000`

### Environment Variables

Required environment variables:

```env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/cert_tracker

# Authentication
JWT_SECRET=your-super-secret-key-min-32-chars
CRON_SECRET=secret-for-cron-endpoint

# NextAuth.js (if using)
NEXTAUTH_SECRET=nextauth-secret-key
NEXTAUTH_URL=http://localhost:3000

# Notifications
EMAIL_API_KEY=sendgrid-or-resend-api-key
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/... (optional)

# Monitoring (optional)
SENTRY_DSN=https://...
```

### Development Commands

```bash
# Development
npm run dev                    # Start dev server
npm run build                  # Production build
npm run start                  # Start production server
npm run lint                   # Run ESLint
npm run type-check             # TypeScript type checking

# Database
npx drizzle-kit studio         # Open Drizzle Studio GUI
npx drizzle-kit generate:pg    # Generate migration
npx drizzle-kit push:pg        # Push schema changes

# Testing
npm run test                   # Run all tests
npm run test:watch             # Watch mode
npm run test:coverage          # Coverage report
npm run test:auth              # Auth tests only
npm run test:cert-checker      # Certificate checker tests
```

## Deployment

### Recommended: Vercel

1. Push code to GitHub/GitLab
2. Import project in Vercel dashboard
3. Configure environment variables
4. Deploy

Vercel configuration (`vercel.json`):

```json
{
  "crons": [{
    "path": "/api/cron/check-certificates",
    "schedule": "0 0 * * *"
  }]
}
```

### Alternative: Docker

```bash
docker build -t cert-tracker .
docker run -p 3000:3000 --env-file .env cert-tracker
```

See `diagrams/10-deployment-architecture.md` for complete deployment guide.

### Database Hosting

Recommended PostgreSQL hosting options:

- **Supabase** - PostgreSQL with connection pooling (recommended)
- **Neon** - Serverless PostgreSQL with auto-scaling
- **Railway** - Simple PostgreSQL deployment
- **Self-hosted** - Docker PostgreSQL

## Security

### Authentication

- Passwords hashed with bcrypt (10+ rounds)
- JWT tokens with 7-day expiration
- Secure session management

### Authorization

Role-based access control (RBAC) with three levels:

| Permission | Admin | Editor | Viewer |
|------------|-------|--------|--------|
| View Certificates | ✅ | ✅ | ✅ |
| Manage Certificates | ✅ | ✅ | ❌ |
| View Alerts | ✅ | ✅ | ✅ |
| Acknowledge Alerts | ✅ | ✅ | ❌ |
| Manage Users | ✅ | ❌ | ❌ |
| Edit Settings | ✅ | ❌ | ❌ |

### API Security

- Authentication middleware on all protected routes
- Cron endpoint protected with secret token
- Input validation with Zod schemas
- Rate limiting on public endpoints
- CORS configuration for allowed origins

See `diagrams/08-security-authentication-flow.md` for implementation details.

## Monitoring & Observability

- **Error Tracking**: Sentry integration for exception monitoring
- **Structured Logging**: JSON logs with Pino
- **Health Checks**: `/api/health` endpoint for uptime monitoring
- **Metrics**: Certificate check success rate, alert delivery rate
- **Dashboards**: Real-time system status and certificate overview

See `diagrams/12-monitoring-observability.md` for complete monitoring strategy.

## Testing

### Test Coverage Goals

- Unit tests for all service layer functions
- Integration tests for API endpoints
- E2E tests for critical user flows
- RBAC permission tests
- Certificate checking logic tests

### Running Tests

```bash
npm run test                   # All tests
npm run test:watch             # Watch mode
npm run test:coverage          # Coverage report
npm run test:unit              # Unit tests only
npm run test:integration       # Integration tests only
```

## Project Structure

```
cert-tracker/
├── src/
│   ├── app/                      # Next.js App Router
│   │   ├── (auth)/               # Auth layout group
│   │   ├── dashboard/
│   │   ├── certificates/
│   │   ├── alerts/
│   │   ├── users/
│   │   ├── settings/
│   │   └── api/                  # API routes
│   ├── components/               # React components
│   │   ├── ui/                   # shadcn/ui components
│   │   ├── certificates/
│   │   ├── alerts/
│   │   └── layout/
│   ├── lib/
│   │   ├── services/             # Business logic
│   │   ├── middleware/           # Auth & RBAC
│   │   ├── utils/
│   │   └── validations/          # Zod schemas
│   ├── db/
│   │   ├── schema.ts             # Drizzle schema
│   │   ├── index.ts              # DB connection
│   │   └── migrations/
│   └── types/
├── diagrams/                     # Architecture diagrams
├── public/
├── CLAUDE.md                     # Development guidance
├── README.md
└── package.json
```

## Contributing

This is currently a planning/design project. Once implementation begins:

1. Review architectural diagrams before implementing features
2. Follow the established patterns (service layer, RBAC, etc.)
3. Write tests for new functionality
4. Update diagrams if architecture changes
5. Follow TypeScript and ESLint conventions

## Roadmap

### Phase 1: Foundation (Weeks 1-2)
- [ ] Database schema implementation with Drizzle
- [ ] Authentication system (NextAuth.js or JWT)
- [ ] Basic API routes (certificates CRUD)

### Phase 2: Core Features (Weeks 3-5)
- [ ] Certificate checking service
- [ ] Alert generation system
- [ ] Basic UI (dashboard, certificate list)

### Phase 3: Notifications (Week 6)
- [ ] Email integration (SendGrid/Resend)
- [ ] Slack integration
- [ ] Webhook support

### Phase 4: Advanced Features (Weeks 7-8)
- [ ] Cron job setup
- [ ] Alert acknowledgment
- [ ] User management UI
- [ ] Settings configuration

### Phase 5: Polish & Deploy (Weeks 9-10)
- [ ] Testing (unit, integration, E2E)
- [ ] Monitoring setup (Sentry, logging)
- [ ] Documentation
- [ ] Production deployment

## License

[Your License Here]

## Support

For questions or issues, please refer to:
- `CLAUDE.md` - Complete development guidance
- `diagrams/README.md` - Architectural documentation index
- [Create an issue](link-to-issues) - Report bugs or request features

---

**Built with**: Next.js 14, TypeScript, PostgreSQL, Drizzle ORM, TailwindCSS, shadcn/ui
