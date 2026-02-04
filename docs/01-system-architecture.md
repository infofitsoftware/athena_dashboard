# High-Level System Architecture

## Overview
The Athena Dashboard is a Business Intelligence web application that provides analytics and visualization capabilities for healthcare AI product data stored in Amazon S3 (Parquet format) and queried via Amazon Athena.

## Component Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLIENT LAYER                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              React Frontend Application                   │  │
│  │  - Dashboard UI                                           │  │
│  │  - Charts & Visualizations                                │  │
│  │  - Authentication UI                                       │  │
│  │  - Query Builder UI                                       │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                              │ HTTPS / REST API
                              │ JWT Tokens
┌─────────────────────────────▼────────────────────────────────────┐
│                      API GATEWAY LAYER                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              FastAPI Backend                              │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │   Auth       │  │   Query      │  │   Dashboard  │   │  │
│  │  │   Service    │  │   Service    │  │   Service    │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │   Athena     │  │   Cache      │  │   Export     │   │  │
│  │  │   Client     │  │   Service    │  │   Service    │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                              │ AWS SDK / boto3
                              │ IAM Roles / Credentials
┌─────────────────────────────▼────────────────────────────────────┐
│                      AWS SERVICES LAYER                          │
│  ┌──────────────────┐  ┌──────────────────┐  ┌──────────────┐  │
│  │  Amazon Athena   │  │   Amazon S3      │  │  IAM /       │  │
│  │  - Query Engine  │  │   - Parquet Data │  │  Secrets     │  │
│  │  - Workgroups    │  │   - Query Results│  │  Manager     │  │
│  └──────────────────┘  └──────────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Data Flow

### Query Execution Flow
```
1. User Action (Frontend)
   └─> Dashboard component requests data
       └─> API call to /api/v1/analytics/query

2. API Layer (FastAPI)
   └─> Authentication middleware validates JWT
       └─> Query Service receives request
           └─> Validates query parameters
               └─> Checks cache (Redis/memory)
                   └─> If miss: Athena Service

3. Athena Service
   └─> Constructs SQL query from parameters
       └─> Executes query via boto3 Athena client
           └─> Polls for query completion
               └─> Retrieves results from S3
                   └─> Normalizes data structure
                       └─> Caches result
                           └─> Returns to API

4. API Response
   └─> Transforms data for frontend consumption
       └─> Returns JSON response

5. Frontend Processing
   └─> Receives data
       └─> Updates chart/table components
           └─> Renders visualization
```

### Authentication Flow
```
1. Login Request
   └─> POST /api/v1/auth/login
       └─> Validates credentials (v1: mock/env-based)
           └─> Generates JWT token
               └─> Returns token + user metadata

2. Subsequent Requests
   └─> Frontend includes JWT in Authorization header
       └─> FastAPI middleware validates token
           └─> Extracts user context
               └─> Proceeds with request
```

## Component Responsibilities

### Frontend (React)
- **UI Rendering**: Dashboard layouts, charts, tables, filters
- **State Management**: Query parameters, user preferences, cached data
- **API Communication**: RESTful API calls, error handling, loading states
- **Routing**: Protected routes, navigation
- **Authentication**: Login/logout, token management

### Backend (FastAPI)
- **API Layer**: REST endpoints, request validation, response formatting
- **Authentication**: JWT validation, user context, role-based access
- **Query Service**: Query construction, parameter validation, result transformation
- **Athena Client**: AWS SDK integration, query execution, result retrieval
- **Caching**: Query result caching, metadata caching
- **Error Handling**: Graceful degradation, error logging, user-friendly messages

### AWS Services
- **Athena**: SQL query execution on Parquet data
- **S3**: Data storage (Parquet files) and query result storage
- **IAM**: Credential management, access control
- **Secrets Manager** (future): Secure credential storage

## Scalability Considerations

### Horizontal Scaling
- **Frontend**: Stateless, can be deployed behind CDN (CloudFront)
- **Backend**: Stateless API, can scale horizontally behind load balancer
- **Athena**: Serverless, auto-scales based on query volume

### Performance Optimization
- **Query Caching**: Cache frequent queries (TTL-based, invalidation strategy)
- **Connection Pooling**: Reuse Athena client connections
- **Async Operations**: FastAPI async endpoints for concurrent requests
- **Frontend Optimization**: Code splitting, lazy loading, memoization
- **CDN**: Static asset delivery, API response caching where appropriate

### Data Volume Handling
- **Query Limits**: Pagination, result size limits
- **Streaming**: For large result sets, consider streaming responses
- **Background Jobs**: Long-running queries moved to async task queue (future)
- **Query Optimization**: Athena query hints, partition pruning

### Cost Optimization
- **Query Result Caching**: Reduce redundant Athena queries
- **Workgroup Configuration**: Cost controls, query limits
- **S3 Lifecycle Policies**: Archive old data, reduce storage costs
- **Monitoring**: Track query costs, set alerts

## Security Architecture

### Network Security
- **HTTPS Only**: All communication encrypted in transit
- **CORS**: Restricted to known frontend origins
- **API Rate Limiting**: Prevent abuse, DDoS protection

### Application Security
- **JWT Tokens**: Stateless authentication, short-lived tokens
- **Credential Isolation**: Environment variables, never in code
- **Input Validation**: SQL injection prevention, parameter sanitization
- **Output Encoding**: XSS prevention

### AWS Security
- **IAM Roles**: Least privilege access
- **VPC**: Isolate backend in private subnet (production)
- **Secrets Rotation**: Regular credential rotation
- **Audit Logging**: CloudTrail for all AWS API calls

## Deployment Architecture

### Development
```
Local Machine
├── Frontend (npm dev server)
├── Backend (uvicorn dev server)
└── AWS Credentials (env vars)
```

### Production (Future)
```
CloudFront (CDN)
    └─> S3 (Static Frontend)
    └─> ALB (Application Load Balancer)
        └─> ECS/Fargate (Backend Containers)
            └─> RDS/ElastiCache (Session/Cache)
                └─> AWS Services (Athena, S3, IAM)
```

## Monitoring & Observability

### Metrics
- API response times
- Athena query execution times
- Error rates
- Cache hit rates
- User activity

### Logging
- Structured logging (JSON format)
- Request/response logging
- Error stack traces
- Athena query logs

### Alerts
- High error rates
- Slow query performance
- Authentication failures
- Cost threshold breaches
