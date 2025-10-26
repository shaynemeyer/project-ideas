# API Architecture

This diagram illustrates the API layer architecture, showing how client requests flow through the API routes to services and database.

```mermaid
graph LR
    subgraph "Client"
        A[React Components]
    end
    
    subgraph "Next.js API Routes"
        B[/api/certificates]
        C[/api/alerts]
        D[/api/users]
        E[/api/settings]
        F[/api/dashboard]
        G[/api/cron/check-certificates]
    end
    
    subgraph "Services"
        H[Certificate Service]
        I[Alert Service]
        J[User Service]
        K[Settings Service]
    end
    
    subgraph "Data Access"
        L[Drizzle ORM]
        M[(Database)]
    end
    
    A -->|HTTP/REST| B
    A -->|HTTP/REST| C
    A -->|HTTP/REST| D
    A -->|HTTP/REST| E
    A -->|HTTP/REST| F
    
    B --> H
    C --> I
    D --> J
    E --> K
    F --> H
    G --> H
    
    H --> L
    I --> L
    J --> L
    K --> L
    L --> M
    
    style A fill:#e1f5ff
    style M fill:#f0f0f0
```

## API Endpoints

### /api/certificates
- `GET /` - List all certificates (with optional filters)
- `POST /` - Add new certificate to monitor
- `GET /:id` - Get specific certificate details
- `PUT /:id` - Update certificate information
- `DELETE /:id` - Remove certificate from monitoring
- `POST /:id/check` - Manually trigger certificate check

### /api/alerts
- `GET /` - List all alerts (with pagination)
- `PUT /:id/acknowledge` - Mark alert as acknowledged
- `DELETE /:id` - Delete an alert

### /api/users
- `GET /` - List all users
- `POST /` - Create new user
- `GET /:id` - Get user details
- `PUT /:id` - Update user information
- `DELETE /:id` - Delete user

### /api/settings
- `GET /notifications` - Get notification settings
- `PUT /notifications` - Update notification settings

### /api/dashboard
- `GET /stats` - Get dashboard statistics (total, active, expiring, expired)

### /api/cron/check-certificates
- `GET /` - Trigger certificate checks (protected by cron secret)

## Service Layer Responsibilities

### Certificate Service
- Certificate validation and checking
- HTTPS connection and SSL extraction
- Status calculation
- Database CRUD operations

### Alert Service
- Alert generation based on thresholds
- Alert rule evaluation
- Acknowledgment handling

### User Service
- User management
- Role-based access control
- User preferences

### Settings Service
- System-wide configuration
- Notification channel management
- Webhook configuration
