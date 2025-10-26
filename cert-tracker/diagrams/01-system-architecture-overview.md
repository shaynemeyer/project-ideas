# System Architecture Overview

This diagram shows the high-level architecture of the SSL Certificate Tracking System, including all major components and their relationships.

```mermaid
graph TB
    subgraph "Client Layer"
        A[Web Browser]
        B[Mobile Browser]
    end
    
    subgraph "Next.js Application"
        C[Next.js Frontend]
        D[App Router]
        E[API Routes]
        F[Server Components]
    end
    
    subgraph "Business Logic"
        G[Certificate Checker Service]
        H[Alert Service]
        I[Notification Service]
    end
    
    subgraph "Data Layer"
        J[(PostgreSQL Database)]
        K[Drizzle ORM]
    end
    
    subgraph "External Services"
        L[Email Service<br/>SendGrid/Resend]
        M[Slack API]
        N[Webhook Endpoints]
    end
    
    subgraph "Background Jobs"
        O[Cron Job Scheduler]
        P[Certificate Check Worker]
    end
    
    A --> C
    B --> C
    C --> D
    D --> E
    D --> F
    E --> K
    F --> K
    K --> J
    E --> G
    E --> H
    G --> K
    H --> I
    I --> L
    I --> M
    I --> N
    O --> P
    P --> G
    P --> E
    
    style A fill:#e1f5ff
    style B fill:#e1f5ff
    style C fill:#fff4e6
    style J fill:#f0f0f0
    style O fill:#e8f5e9
```

## Component Description

### Client Layer
- **Web Browser**: Desktop web interface
- **Mobile Browser**: Mobile-responsive interface

### Next.js Application
- **Next.js Frontend**: React-based UI with TailwindCSS and shadcn/ui
- **App Router**: Next.js 14+ App Router for routing
- **API Routes**: RESTful API endpoints
- **Server Components**: Server-side rendered components

### Business Logic
- **Certificate Checker Service**: Core service for SSL certificate validation
- **Alert Service**: Manages alert generation and rules
- **Notification Service**: Handles multi-channel notifications

### Data Layer
- **PostgreSQL Database**: Primary data store
- **Drizzle ORM**: Type-safe database access layer

### External Services
- **Email Service**: Transactional email delivery
- **Slack API**: Team collaboration notifications
- **Webhook Endpoints**: Custom integrations

### Background Jobs
- **Cron Job Scheduler**: Scheduled task execution
- **Certificate Check Worker**: Automated certificate monitoring
