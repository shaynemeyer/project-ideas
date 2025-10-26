# SSL Certificate Tracker - System Diagrams

This directory contains comprehensive architectural and technical diagrams for the SSL Certificate Tracking System.

## Table of Contents

### 1. [System Architecture Overview](./01-system-architecture-overview.md)
High-level view of all system components including:
- Client Layer (Web & Mobile)
- Next.js Application Layer
- Business Logic Services
- Data Layer (PostgreSQL + Drizzle ORM)
- External Services (Email, Slack, Webhooks)
- Background Jobs (Cron Scheduler)

**Use this when**: Understanding the complete system architecture and how components interact.

---

### 2. [Database Schema (ERD)](./02-database-schema-erd.md)
Entity Relationship Diagram showing:
- Certificates table (main data store)
- Certificate History (audit trail)
- Alerts (notification records)
- Users (authentication & authorization)
- Notification Settings (system configuration)

**Use this when**: Designing database schema, understanding data relationships, or implementing Drizzle ORM models.

---

### 3. [API Architecture](./03-api-architecture.md)
RESTful API structure including:
- API endpoint definitions
- Service layer architecture
- Data access patterns
- Request/response flow

**Endpoints covered**:
- `/api/certificates` - Certificate CRUD operations
- `/api/alerts` - Alert management
- `/api/users` - User management
- `/api/settings` - System configuration
- `/api/dashboard` - Dashboard statistics
- `/api/cron/check-certificates` - Automated checks

**Use this when**: Implementing API routes or understanding service layer responsibilities.

---

### 4. [Certificate Checking Flow](./04-certificate-checking-flow.md)
Sequence diagram showing:
- Cron job triggering
- Certificate validation process
- SSL extraction workflow
- Alert threshold evaluation
- Multi-channel notifications

**Use this when**: Implementing certificate checking logic or understanding the automated monitoring process.

---

### 5. [User Interaction Flow](./05-user-interaction-flow.md)
State diagram covering:
- Navigation between pages
- Certificate management workflows
- Alert acknowledgment process
- Settings configuration
- User management flows

**Use this when**: Designing user interface flows or implementing navigation logic.

---

### 6. [Component Hierarchy](./06-component-hierarchy.md)
React component structure showing:
- Layout components
- Page components
- Reusable UI components (shadcn/ui)
- File and folder organization

**Components included**:
- Dashboard with stats cards
- Certificate table and forms
- Alert management
- User administration
- Settings panels

**Use this when**: Building frontend components or organizing the component structure.

---

### 7. [Data Flow Diagram](./07-data-flow-diagram.md)
Complete data transformation pipeline:
- User input validation
- Certificate checking process
- Status calculations
- Database storage
- Alert generation
- Notification dispatch

**Use this when**: Understanding how data moves through the system from input to notification.

---

### 8. [Security & Authentication Flow](./08-security-authentication-flow.md)
Security architecture including:
- Authentication strategies (NextAuth.js, JWT)
- Authorization middleware
- Role-Based Access Control (RBAC)
- API route protection
- Security best practices

**Permission matrix** for Admin, Editor, and Viewer roles.

**Use this when**: Implementing authentication, authorization, or securing API routes.

---

### 9. [Notification System Architecture](./09-notification-system-architecture.md)
Multi-channel notification system:
- Alert detection and rules engine
- Email channel (SendGrid/Resend)
- Slack integration
- Webhook support
- Delivery tracking and retry logic

**Use this when**: Implementing the notification system or integrating new channels.

---

### 10. [Deployment Architecture](./10-deployment-architecture.md)
Production deployment setup:
- Vercel deployment (recommended)
- Docker containerization (alternative)
- Database hosting options (Supabase, Neon, Railway)
- CI/CD pipeline (GitHub Actions)
- Monitoring setup (Sentry, Uptime monitoring)
- Scaling considerations

**Use this when**: Deploying the application or setting up infrastructure.

---

### 11. [Certificate Status State Machine](./11-certificate-status-state-machine.md)
Certificate lifecycle management:
- All possible states (Pending, Active, Expiring, Expired, Error)
- State transitions
- Alert triggering conditions
- Status-based actions and recommendations

**Use this when**: Implementing certificate status logic or understanding the lifecycle.

---

### 12. [Monitoring & Observability](./12-monitoring-observability.md)
Complete monitoring strategy:
- Application performance metrics
- Business metrics tracking
- Error tracking (Sentry)
- Structured logging (Pino)
- Alerting rules
- Health checks and dashboards

**Use this when**: Setting up monitoring, implementing logging, or creating dashboards.

---

## How to Use These Diagrams

### For Development
1. Start with **System Architecture Overview** to understand the big picture
2. Review **Database Schema** before implementing data models
3. Follow **API Architecture** when building API endpoints
4. Reference **Component Hierarchy** when structuring the frontend
5. Use **Security & Authentication Flow** for implementing auth

### For Operations
1. Study **Deployment Architecture** for infrastructure setup
2. Implement **Monitoring & Observability** for production readiness
3. Review **Certificate Checking Flow** to understand the core functionality
4. Use **Notification System Architecture** for alert configuration

### For Understanding
1. **Data Flow Diagram** shows how data moves through the system
2. **User Interaction Flow** explains the user experience
3. **Certificate Status State Machine** details the certificate lifecycle

## Diagram Format

All diagrams are created using [Mermaid](https://mermaid.js.org/), which means they:
- Render automatically on GitHub, GitLab, and in many documentation tools
- Can be viewed in VS Code with the Mermaid extension
- Are version-controllable as plain text
- Can be exported to PNG/SVG for presentations

## Technology Stack

- **Frontend**: Next.js 14+, TypeScript, TailwindCSS, shadcn/ui
- **Backend**: Next.js API Routes
- **Database**: PostgreSQL with Drizzle ORM
- **Authentication**: NextAuth.js or custom JWT
- **Deployment**: Vercel (recommended) or Docker
- **Monitoring**: Sentry, Vercel Analytics
- **Notifications**: SendGrid/Resend, Slack, Webhooks

## Quick Start

To view these diagrams:

1. **GitHub/GitLab**: Open any `.md` file and the diagrams will render automatically
2. **VS Code**: Install the "Markdown Preview Mermaid Support" extension
3. **Local Viewer**: Use [Mermaid Live Editor](https://mermaid.live/)
4. **Documentation Sites**: Most support Mermaid natively (Docusaurus, MkDocs, etc.)

## Architecture Principles

The system is designed following these principles:

1. **Separation of Concerns**: Clear boundaries between presentation, business logic, and data layers
2. **Scalability**: Serverless architecture supports auto-scaling
3. **Observability**: Comprehensive logging and monitoring
4. **Security**: Role-based access control and authentication
5. **Maintainability**: Structured code organization and TypeScript type safety
6. **Reliability**: Error handling, retry logic, and health checks

## Next Steps

After reviewing the diagrams:

1. Set up the development environment
2. Initialize the database schema using Drizzle
3. Implement core certificate checking functionality
4. Build the frontend components
5. Deploy to staging environment
6. Set up monitoring and alerts
7. Deploy to production

## Contributing

When adding new features:
1. Update relevant diagrams to reflect changes
2. Follow the established architectural patterns
3. Maintain consistency in naming and structure
4. Document significant architectural decisions

## Questions?

These diagrams represent the recommended architecture. They can be adapted based on:
- Specific organizational requirements
- Scale and performance needs
- Budget constraints
- Team expertise

Each diagram file includes detailed implementation guidance and code examples.
