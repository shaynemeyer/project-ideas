# Project Ideas

A collection of software project ideas in various stages of planning and design.

## Projects

### [Fictional Airline Database](fictional_airline/)
**Status**: Planning | **Type**: Academic Database Design | **Technology**: PostgreSQL

A comprehensive relational database design for "Anchor Air", a fictional US domestic airline. Covers flight operations, crew management, passenger bookings, and airport operations with proper 3NF normalization.

**Key Features**:
- Multi-leg flight routing with layovers
- Crew ranking and assignment system
- Passenger booking and baggage tracking
- Pricing with multiple fare classes
- Airport gate capacity management

### [Landscaping CRM SaaS](scaper-crm/)
**Status**: Planning | **Type**: Production SaaS Application | **Technology**: Next.js, PostgreSQL

A modern multi-tenant SaaS CRM system designed for landscaping companies to manage clients, properties, jobs, scheduling, and invoicing.

**Key Features**:
- Multi-tenant architecture with Row Level Security
- Client and property management
- Job tracking and crew scheduling
- Invoice generation and payment tracking
- Calendar-based scheduling with route optimization
- Role-based access control (RBAC)

## Getting Started

Each project has its own detailed documentation:

- Read the project-specific `README.md` for overview and features
- Check `CLAUDE.md` files for development guidance and architecture details
- Review implementation plans and design documents in each project folder

## Repository Structure

```
project-ideas/
├── CLAUDE.md              # Repository-wide development guidance
├── readme.md              # This file
├── fictional_airline/     # Airline database design project
│   ├── CLAUDE.md
│   ├── README.md
│   ├── airline.md         # SQL schema
│   └── requirements.md
└── scaper-crm/           # Landscaping CRM SaaS
    ├── CLAUDE.md
    ├── readme.md
    ├── implementation-roadmap.md
    ├── tenant-isolation-strategy.md
    └── *.mermaid          # Architecture diagrams
```

## Status

All projects are currently in the **planning/design phase** with no implementation started. Each project includes comprehensive documentation, architecture designs, and implementation roadmaps.
