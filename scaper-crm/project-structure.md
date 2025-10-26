# Project Structure - Landscaping CRM SaaS

## Recommended: Next.js 14+ App Router Structure

```
landscaping-crm/
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
│
├── app/                          # Next.js App Router
│   ├── (auth)/                   # Route group for auth pages
│   │   ├── login/
│   │   │   └── page.tsx
│   │   ├── signup/
│   │   │   └── page.tsx
│   │   ├── forgot-password/
│   │   │   └── page.tsx
│   │   └── layout.tsx            # Auth-specific layout
│   │
│   ├── (dashboard)/              # Route group for authenticated pages
│   │   ├── layout.tsx            # Main dashboard layout with sidebar
│   │   ├── page.tsx              # Dashboard home
│   │   │
│   │   ├── clients/
│   │   │   ├── page.tsx          # Client list
│   │   │   ├── [id]/
│   │   │   │   ├── page.tsx      # Client detail
│   │   │   │   └── edit/
│   │   │   │       └── page.tsx  # Edit client
│   │   │   └── new/
│   │   │       └── page.tsx      # New client
│   │   │
│   │   ├── properties/
│   │   │   ├── page.tsx
│   │   │   └── [id]/
│   │   │       └── page.tsx
│   │   │
│   │   ├── jobs/
│   │   │   ├── page.tsx          # Job board
│   │   │   ├── calendar/
│   │   │   │   └── page.tsx      # Calendar view
│   │   │   ├── [id]/
│   │   │   │   ├── page.tsx      # Job detail
│   │   │   │   └── edit/
│   │   │   │       └── page.tsx
│   │   │   └── new/
│   │   │       └── page.tsx
│   │   │
│   │   ├── schedule/
│   │   │   ├── page.tsx          # Schedule overview
│   │   │   └── [date]/
│   │   │       └── page.tsx      # Daily schedule
│   │   │
│   │   ├── invoices/
│   │   │   ├── page.tsx
│   │   │   ├── [id]/
│   │   │   │   └── page.tsx
│   │   │   └── new/
│   │   │       └── page.tsx
│   │   │
│   │   ├── reports/
│   │   │   ├── page.tsx
│   │   │   ├── revenue/
│   │   │   │   └── page.tsx
│   │   │   └── performance/
│   │   │       └── page.tsx
│   │   │
│   │   └── settings/
│   │       ├── page.tsx
│   │       ├── profile/
│   │       │   └── page.tsx
│   │       ├── team/
│   │       │   └── page.tsx
│   │       ├── billing/
│   │       │   └── page.tsx
│   │       └── integrations/
│   │           └── page.tsx
│   │
│   ├── (admin)/                  # Route group for admin/super-admin
│   │   ├── admin/
│   │   │   ├── layout.tsx
│   │   │   ├── tenants/
│   │   │   │   ├── page.tsx      # Manage all tenants
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx
│   │   │   └── analytics/
│   │   │       └── page.tsx
│   │   └── middleware.ts
│   │
│   ├── api/                      # API routes (if needed)
│   │   ├── webhooks/
│   │   │   ├── stripe/
│   │   │   │   └── route.ts
│   │   │   └── clerk/
│   │   │       └── route.ts
│   │   └── cron/
│   │       ├── daily-reminders/
│   │       │   └── route.ts
│   │       └── invoice-overdue/
│   │           └── route.ts
│   │
│   ├── actions/                  # Server Actions
│   │   ├── clients.ts
│   │   ├── jobs.ts
│   │   ├── invoices.ts
│   │   ├── schedule.ts
│   │   ├── users.ts
│   │   └── auth.ts
│   │
│   ├── layout.tsx                # Root layout
│   ├── page.tsx                  # Landing page (public)
│   ├── error.tsx                 # Global error boundary
│   ├── not-found.tsx            # 404 page
│   └── global.css               # Global styles
│
├── components/                   # React components
│   ├── ui/                       # ShadCN UI components
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── dialog.tsx
│   │   ├── card.tsx
│   │   ├── table.tsx
│   │   ├── calendar.tsx
│   │   └── ...
│   │
│   ├── forms/                    # Form components
│   │   ├── client-form.tsx
│   │   ├── job-form.tsx
│   │   ├── invoice-form.tsx
│   │   └── property-form.tsx
│   │
│   ├── clients/                  # Client-specific components
│   │   ├── client-list.tsx
│   │   ├── client-card.tsx
│   │   ├── client-details.tsx
│   │   └── client-search.tsx
│   │
│   ├── jobs/                     # Job-specific components
│   │   ├── job-board.tsx
│   │   ├── job-card.tsx
│   │   ├── job-timeline.tsx
│   │   ├── job-status-badge.tsx
│   │   └── job-assignment.tsx
│   │
│   ├── schedule/                 # Scheduling components
│   │   ├── calendar-view.tsx
│   │   ├── daily-schedule.tsx
│   │   ├── route-optimizer.tsx
│   │   └── crew-assignment.tsx
│   │
│   ├── invoices/                 # Invoice components
│   │   ├── invoice-list.tsx
│   │   ├── invoice-preview.tsx
│   │   ├── payment-form.tsx
│   │   └── invoice-pdf.tsx
│   │
│   ├── layout/                   # Layout components
│   │   ├── header.tsx
│   │   ├── sidebar.tsx
│   │   ├── mobile-nav.tsx
│   │   └── footer.tsx
│   │
│   ├── dashboard/                # Dashboard widgets
│   │   ├── stats-card.tsx
│   │   ├── revenue-chart.tsx
│   │   ├── recent-jobs.tsx
│   │   └── upcoming-schedule.tsx
│   │
│   └── shared/                   # Shared/common components
│       ├── data-table.tsx
│       ├── search-input.tsx
│       ├── status-badge.tsx
│       ├── avatar-group.tsx
│       ├── file-upload.tsx
│       ├── map-view.tsx
│       └── loading-spinner.tsx
│
├── lib/                          # Library code
│   ├── db/                       # Database
│   │   ├── index.ts              # Drizzle client
│   │   ├── schema/               # Database schemas
│   │   │   ├── index.ts
│   │   │   ├── tenants.ts
│   │   │   ├── users.ts
│   │   │   ├── clients.ts
│   │   │   ├── properties.ts
│   │   │   ├── jobs.ts
│   │   │   ├── invoices.ts
│   │   │   ├── schedules.ts
│   │   │   └── activity-log.ts
│   │   ├── migrations/           # Database migrations
│   │   │   └── 0001_initial.sql
│   │   └── seed.ts               # Seed data
│   │
│   ├── auth/                     # Authentication
│   │   ├── index.ts
│   │   ├── session.ts
│   │   ├── permissions.ts
│   │   └── tenant-context.ts
│   │
│   ├── api/                      # API utilities (if using FastAPI)
│   │   ├── client.ts
│   │   └── endpoints/
│   │       ├── clients.ts
│   │       ├── jobs.ts
│   │       └── invoices.ts
│   │
│   ├── utils/                    # Utility functions
│   │   ├── cn.ts                 # className utilities
│   │   ├── date.ts               # Date formatting
│   │   ├── currency.ts           # Currency formatting
│   │   ├── validation.ts         # Validation helpers
│   │   └── strings.ts            # String utilities
│   │
│   ├── hooks/                    # Custom React hooks
│   │   ├── use-tenant.ts
│   │   ├── use-user.ts
│   │   ├── use-clients.ts
│   │   ├── use-jobs.ts
│   │   ├── use-debounce.ts
│   │   └── use-local-storage.ts
│   │
│   ├── validations/              # Zod schemas
│   │   ├── client.ts
│   │   ├── job.ts
│   │   ├── invoice.ts
│   │   ├── property.ts
│   │   └── user.ts
│   │
│   ├── services/                 # Business logic services
│   │   ├── client-service.ts
│   │   ├── job-service.ts
│   │   ├── invoice-service.ts
│   │   ├── notification-service.ts
│   │   └── billing-service.ts
│   │
│   └── constants/                # Constants
│       ├── job-statuses.ts
│       ├── permissions.ts
│       └── config.ts
│
├── types/                        # TypeScript types
│   ├── index.ts
│   ├── database.ts
│   ├── api.ts
│   └── enums.ts
│
├── public/                       # Static files
│   ├── images/
│   │   └── logo.svg
│   ├── icons/
│   └── fonts/
│
├── tests/                        # Tests
│   ├── unit/
│   │   ├── services/
│   │   └── utils/
│   ├── integration/
│   │   ├── actions/
│   │   └── api/
│   └── e2e/
│       └── flows/
│
├── docs/                         # Documentation
│   ├── architecture.md
│   ├── database.md
│   ├── deployment.md
│   └── api.md
│
├── scripts/                      # Utility scripts
│   ├── seed-db.ts
│   ├── migrate.ts
│   └── generate-types.ts
│
├── .env.example
├── .env.local
├── .eslintrc.json
├── .prettierrc
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── drizzle.config.ts
├── package.json
└── README.md
```

## Key Design Decisions

### 1. Route Groups
- `(auth)` - Authentication pages without dashboard layout
- `(dashboard)` - Main app with sidebar layout
- `(admin)` - Super admin section with different layout

### 2. Colocation
- Components are organized by feature domain
- Each feature has its own folder with related components
- Shared components in `components/shared`

### 3. Server Actions Location
- All server actions in `app/actions/`
- Grouped by resource (clients, jobs, etc.)
- Keep actions close to the app directory for easy imports

### 4. Database Schema Organization
- Each table in its own file
- Exported from central `schema/index.ts`
- Makes schema manageable as it grows

### 5. Type Safety
- Types generated from Drizzle schema
- Additional types in `types/` for API contracts
- Zod schemas for validation

## Configuration Files

### drizzle.config.ts
```typescript
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

### next.config.js
```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    serverActions: {
      bodySizeLimit: '2mb',
    },
  },
  images: {
    domains: ['your-s3-bucket.s3.amazonaws.com'],
  },
}

module.exports = nextConfig
```

### tailwind.config.ts
```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  darkMode: ['class'],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        border: 'hsl(var(--border))',
        input: 'hsl(var(--input))',
        ring: 'hsl(var(--ring))',
        background: 'hsl(var(--background))',
        foreground: 'hsl(var(--foreground))',
        primary: {
          DEFAULT: 'hsl(var(--primary))',
          foreground: 'hsl(var(--primary-foreground))',
        },
        // ... ShadCN color variables
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
}

export default config
```

## Alternative: FastAPI Backend Structure

```
landscaping-crm-backend/
├── app/
│   ├── main.py                   # FastAPI app entry
│   ├── core/
│   │   ├── config.py            # Settings
│   │   ├── deps.py              # Dependencies
│   │   ├── security.py          # Auth utilities
│   │   └── database.py          # DB connection
│   │
│   ├── models/                   # SQLAlchemy models
│   │   ├── tenant.py
│   │   ├── user.py
│   │   ├── client.py
│   │   ├── job.py
│   │   └── invoice.py
│   │
│   ├── schemas/                  # Pydantic schemas
│   │   ├── tenant.py
│   │   ├── user.py
│   │   ├── client.py
│   │   └── job.py
│   │
│   ├── api/                      # API routes
│   │   ├── deps.py
│   │   └── v1/
│   │       ├── clients.py
│   │       ├── jobs.py
│   │       ├── invoices.py
│   │       └── auth.py
│   │
│   ├── services/                 # Business logic
│   │   ├── client_service.py
│   │   ├── job_service.py
│   │   └── invoice_service.py
│   │
│   └── tasks/                    # Celery tasks
│       ├── notifications.py
│       └── reports.py
│
├── tests/
├── alembic/                      # Migrations
├── requirements.txt
└── Dockerfile
```

## Recommended Tooling

### Development
- **Prettier**: Code formatting
- **ESLint**: Linting
- **Husky**: Git hooks
- **Lint-staged**: Pre-commit linting

### Testing
- **Vitest**: Unit tests
- **Playwright**: E2E tests
- **React Testing Library**: Component tests

### Database
- **Drizzle Kit**: Migrations
- **Drizzle Studio**: DB GUI

### Deployment
- **Vercel**: Frontend + Server Actions
- **Railway/Render**: Postgres + Redis
- **AWS S3/Cloudflare R2**: File storage
