# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a **monorepo containing multiple project ideas** in various stages of planning and design. No implementation has started yet - all projects are in the design/planning phase with documentation only.

## Project Structure

```
project-ideas/
├── fictional_airline/     # Airline database design project (PostgreSQL)
└── scaper-crm/           # Multi-tenant landscaping CRM SaaS (Next.js)
```

## Projects

### 1. Fictional Airline (`fictional_airline/`)

**Type**: Academic database design project
**Domain**: US domestic airline operations management
**Technology**: PostgreSQL relational database
**Status**: Planning - schema design complete, no implementation

A complete relational database design for "Anchor Air", a fictional airline. The project covers flight operations, crew management, passenger bookings, and airport operations. Schema is normalized to 3NF with composite keys for temporal data.

**Key Files**:
- `CLAUDE.md` - Project-specific development guidance
- `README.md` - Complete project documentation
- `airline.md` - Database schema (SQL DDL)
- `requirements.md` - Business requirements
- `FRONTEND_DESIGN.md` - Frontend design specifications

**When working in this project**:
- Refer to `fictional_airline/CLAUDE.md` for detailed schema information
- Follow PostgreSQL table creation order due to foreign key dependencies
- All airports must be US domestic (3-letter codes)
- Schema must maintain 3NF normalization

### 2. Landscaping CRM SaaS (`scaper-crm/`)

**Type**: Production SaaS application
**Domain**: Multi-tenant CRM for landscaping businesses
**Technology**: Next.js 14+, PostgreSQL with RLS, Drizzle ORM
**Status**: Planning - architecture and roadmap complete, no implementation

A modern multi-tenant SaaS CRM system for landscaping companies to manage clients, properties, jobs, scheduling, and invoicing. Uses shared database architecture with Row Level Security.

**Key Files**:
- `CLAUDE.md` - Comprehensive development guidance including multi-tenant patterns
- `readme.md` - Project overview and setup instructions
- `project-structure.md` - Detailed file organization
- `implementation-roadmap.md` - 16-week implementation plan
- `tenant-isolation-strategy.md` - Critical multi-tenant security architecture
- `backend-comparison.md` - Technology decision analysis
- `*.mermaid` - Architecture diagrams

**When working in this project**:
- **CRITICAL**: Read `scaper-crm/CLAUDE.md` for multi-tenant security requirements
- Every table MUST include `tenant_id` with RLS policies
- All queries MUST filter by `tenant_id`
- Follow the Server Actions pattern for backend logic
- Refer to implementation roadmap for feature prioritization

## Working in This Repository

### Navigation

When the user mentions a project by name or context:
- "airline" or "database project" → work in `fictional_airline/`
- "CRM", "landscaping", or "SaaS" → work in `scaper-crm/`
- Always read the project-specific `CLAUDE.md` before starting work

### General Guidelines

1. **Check project context first**: Both projects have detailed CLAUDE.md files - always read them before making changes
2. **No cross-project dependencies**: These are independent projects with different tech stacks
3. **Planning phase**: No implementation has started - focus on design documents when asked
4. **Respect project architecture**: Each project has specific architectural decisions documented

### Common Commands

Since no implementation exists yet, there are no build/test commands. When implementation begins:

**fictional_airline** (future):
```bash
# PostgreSQL database operations
psql -d anchor_air -f airline.md        # Create schema
psql -d anchor_air -f seed.sql          # Populate data
```

**scaper-crm** (future):
```bash
# Development
npm run dev                             # Start dev server
npm run build                           # Production build

# Database
npx drizzle-kit generate:pg             # Generate migration
npx drizzle-kit push:pg                 # Push schema changes
npx drizzle-kit studio                  # Database GUI

# Testing
npm run test                            # Unit tests
npm run test:tenant-isolation           # Critical tenant security tests
```

## Important Notes

### For fictional_airline:
- Academic project focused on proper database design
- Must maintain 3NF normalization
- Complex foreign key dependencies require specific table creation order
- See `fictional_airline/CLAUDE.md` for complete schema architecture

### For scaper-crm:
- **SECURITY CRITICAL**: Multi-tenant architecture requires strict tenant isolation
- Every database operation must include `tenant_id` checks
- Read `scaper-crm/tenant-isolation-strategy.md` before any implementation
- See `scaper-crm/CLAUDE.md` for complete development patterns
