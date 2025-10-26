# Landscaping CRM SaaS

A modern, multi-tenant SaaS CRM system designed specifically for landscaping companies to manage clients, properties, jobs, scheduling, and invoicing.

## 🎯 Project Status

**Planning Phase** - This repository currently contains design documents and architecture planning. Implementation has not yet started.

## 📋 Overview

This CRM helps landscaping businesses streamline their operations by providing:

- **Client & Property Management** - Track customers and their properties with location data
- **Job Management** - Create, assign, and track jobs from quote to completion
- **Crew Scheduling** - Calendar-based scheduling with crew assignments and route optimization
- **Invoicing & Payments** - Generate invoices, track payments, and manage billing
- **Team Collaboration** - Multi-user support with role-based permissions
- **Mobile-First Design** - Responsive interface optimized for field crews

## 🏗️ Architecture

### Tech Stack

**Frontend & Backend**:
- Next.js 14+ (App Router, TypeScript)
- Server Actions for backend logic
- Tailwind CSS + ShadCN UI
- React Query for state management
- Drizzle ORM for database operations

**Database & Infrastructure**:
- PostgreSQL 15+ with Row Level Security
- Redis for caching
- Clerk for authentication
- Vercel for deployment

### Multi-Tenant Design

Uses a **shared database with Row Level Security (RLS)** approach:
- Single PostgreSQL database
- All tables include `tenant_id` column
- RLS policies enforce automatic tenant isolation
- Optimized for cost-effectiveness and scalability

See [tenant-isolation-strategy.md](./tenant-isolation-strategy.md) for detailed implementation.

## 📁 Repository Structure

```
scaper-crm/
├── readme.md                          # This file
├── CLAUDE.md                          # AI assistant context
├── project-structure.md               # Detailed file structure
├── implementation-roadmap.md          # 16-week implementation plan
├── tenant-isolation-strategy.md       # Multi-tenant architecture
├── backend-comparison.md              # Backend technology analysis
├── database-schema.mermaid            # Database schema diagram
├── data-flow.mermaid                  # Data flow diagram
└── architecture.mermaid               # System architecture diagram
```

## 🚀 Getting Started

### Prerequisites

- Node.js 18+
- PostgreSQL 15+
- npm or pnpm

### Initial Setup (When Implementation Begins)

```bash
# Create Next.js project
npx create-next-app@latest landscaping-crm --typescript --tailwind --app
cd landscaping-crm

# Install core dependencies
npm install drizzle-orm postgres @tanstack/react-query
npm install -D drizzle-kit

# Initialize ShadCN UI
npx shadcn-ui@latest init

# Set up environment variables
cp .env.example .env.local
# Edit .env.local with your database credentials
```

### Database Setup

```bash
# Generate initial migration
npx drizzle-kit generate:pg

# Push schema to database
npx drizzle-kit push:pg

# Seed development data
npm run db:seed

# Open database GUI
npx drizzle-kit studio
```

## 📖 Documentation

- **[Project Structure](./project-structure.md)** - Detailed directory layout and organization
- **[Implementation Roadmap](./implementation-roadmap.md)** - 16-week development plan with phases
- **[Tenant Isolation Strategy](./tenant-isolation-strategy.md)** - Multi-tenant architecture and security
- **[Backend Comparison](./backend-comparison.md)** - Analysis of Next.js vs FastAPI approaches
- **[CLAUDE.md](./CLAUDE.md)** - Development context for AI assistants

## 🎯 MVP Features (8-Week Timeline)

### Core Features
1. ✅ Client & Property Management
2. ✅ Job Tracking & Management
3. ✅ Calendar-Based Scheduling
4. ✅ Invoice Generation
5. ✅ Payment Tracking
6. ✅ User Management & RBAC
7. ✅ Dashboard with Analytics

### Post-MVP Features
- Route optimization with mapping
- Automated email/SMS reminders
- Customer self-service portal
- Equipment & materials tracking
- Time tracking for crews
- Mobile app (React Native)
- QuickBooks integration
- Advanced reporting & analytics

## 🔒 Security & Multi-Tenancy

All data is isolated by tenant with multiple security layers:

1. **Row Level Security** - PostgreSQL RLS policies enforce tenant boundaries
2. **Application-level filtering** - All queries include `tenant_id` checks
3. **Middleware validation** - Tenant context verified on every request
4. **Foreign key constraints** - Include `tenant_id` to prevent cross-tenant references
5. **Comprehensive testing** - Tenant isolation verified in test suite

## 🧪 Testing Strategy

```bash
# Unit tests
npm run test

# Integration tests
npm run test:integration

# E2E tests
npm run test:e2e

# Tenant isolation tests (critical)
npm run test:tenant-isolation
```

## 📊 Database Schema

Key tables:
- `tenants` - Company/organization records
- `users` - User accounts with roles
- `clients` - Customer information
- `properties` - Client properties/locations
- `jobs` - Job records with status tracking
- `schedules` - Crew scheduling and assignments
- `invoices` - Billing and payment records
- `activity_log` - Audit trail

See [database-schema.mermaid](./database-schema.mermaid) for complete schema.

## 🛠️ Development Commands

```bash
# Development
npm run dev              # Start dev server
npm run build            # Production build
npm run start            # Start production server

# Database
npm run db:generate      # Generate migration
npm run db:push          # Push schema changes
npm run db:seed          # Seed data
npm run db:studio        # Open database GUI

# Code Quality
npm run lint             # Run ESLint
npm run type-check       # TypeScript validation
npm run format           # Format with Prettier
```

## 🌟 Key Design Decisions

### Why Next.js Server Actions?
- Single codebase (no separate API)
- End-to-end TypeScript type safety
- Simplified deployment
- Built-in caching strategies
- Can migrate to FastAPI later if mobile apps needed

### Why Shared Database?
- Cost-effective for 10-500+ tenants
- Simpler to manage and deploy
- Good performance with proper indexing
- Easy cross-tenant analytics for admins
- PostgreSQL RLS provides strong isolation

### Why Drizzle ORM?
- Full TypeScript type safety
- SQL-like query builder
- No code generation required
- Excellent performance
- Great developer experience

## 📈 Success Metrics

### Technical Goals
- Page load time < 1s
- Time to interactive < 2s
- 99.9% uptime
- < 1% error rate

### Business Goals
- User onboarding < 10 minutes
- 80% weekly active usage
- < 5% monthly churn
- NPS > 50

### Product Goals
- 90% of jobs tracked in system
- 80% of invoices generated in system
- 95% on-time scheduling accuracy

## 📝 License

MIT

## 🤝 Contributing

This is currently a personal project in the planning phase. Contributions welcome once implementation begins.

## 📧 Contact

For questions or feedback, please open an issue in this repository.
