# Security & Authentication Strategy

## Authentication Approach

### Version 1: Simplified Authentication

**Strategy**: Mock authentication with JWT tokens for rapid development

#### Implementation
- **Backend**: FastAPI endpoint `/api/v1/auth/login`
- **Credentials**: Environment variables or simple config file
- **Validation**: Compare against stored credentials (env vars or config)
- **Token Generation**: JWT with user metadata (username, role, exp)
- **Token Storage**: HTTP-only cookies (preferred) or localStorage

#### User Model (v1)
```python
{
    "username": str,
    "role": str,  # "admin", "analyst", "viewer"
    "email": str,
    "permissions": List[str]  # Future: granular permissions
}
```

#### Token Payload
```json
{
    "sub": "username",
    "role": "analyst",
    "email": "user@example.com",
    "exp": 1234567890,
    "iat": 1234567890
}
```

### Future: Production Authentication

**Options to Consider:**
1. **AWS Cognito**: Managed authentication, SSO integration
2. **OAuth 2.0 / OIDC**: Integration with existing identity provider
3. **LDAP / Active Directory**: Enterprise directory integration
4. **Custom Auth**: Database-backed with password hashing

## Token Handling

### JWT Configuration

#### Token Expiration
- **Access Token**: 15 minutes (short-lived)
- **Refresh Token**: 7 days (long-lived, stored securely)
- **Strategy**: Refresh token rotation on use

#### Token Storage Options

**Option 1: HTTP-Only Cookies (Recommended)**
```python
# Backend sets cookie
response.set_cookie(
    key="access_token",
    value=token,
    httponly=True,
    secure=True,  # HTTPS only
    samesite="strict",
    max_age=900  # 15 minutes
)
```

**Pros:**
- XSS protection (JavaScript cannot access)
- Automatic inclusion in requests
- Secure by default

**Cons:**
- CSRF protection needed
- Slightly more complex setup

**Option 2: localStorage + Authorization Header**
```javascript
// Frontend stores token
localStorage.setItem('access_token', token);

// Include in requests
headers: {
    'Authorization': `Bearer ${token}`
}
```

**Pros:**
- Simple implementation
- Works with SPAs
- Easy to debug

**Cons:**
- Vulnerable to XSS
- Manual token management

**Recommendation**: Start with localStorage for v1, migrate to HTTP-only cookies in production.

### Token Refresh Flow

```
1. User logs in
   └─> Receive access_token + refresh_token

2. Access token expires
   └─> Frontend detects 401 response
       └─> Call /api/v1/auth/refresh
           └─> Backend validates refresh_token
               └─> Issue new access_token
                   └─> Retry original request
```

**Implementation:**
- **Frontend**: Axios interceptor for automatic refresh
- **Backend**: `/api/v1/auth/refresh` endpoint
- **Error Handling**: Redirect to login if refresh fails

## Environment Variable Safety

### Development (Local)

#### Credential Storage
```bash
# .env file (gitignored)
AWS_ACCESS_KEY_ID=xxx
AWS_SECRET_ACCESS_KEY=xxx
AWS_REGION=us-east-1
ATHENA_WORKGROUP=primary
ATHENA_DATABASE=healthcare_analytics
ATHENA_TABLE=patient_data
ATHENA_OUTPUT_S3=s3://athena-results/

# Auth credentials (v1)
ADMIN_USERNAME=admin
ADMIN_PASSWORD=secure_password_here
JWT_SECRET_KEY=your-secret-key-here
JWT_ALGORITHM=HS256
```

#### Security Practices
- **`.env` file**: Never commit to git, add to `.gitignore`
- **`.env.example`**: Template file with placeholder values
- **Validation**: Validate all required env vars on startup
- **Type Safety**: Use Pydantic Settings for validation

#### Backend Implementation
```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    aws_access_key_id: str
    aws_secret_access_key: str
    aws_region: str
    athena_workgroup: str
    athena_database: str
    athena_table: str
    jwt_secret_key: str
    jwt_algorithm: str = "HS256"
    
    class Config:
        env_file = ".env"
        case_sensitive = False

settings = Settings()
```

### Production Strategy

#### Option 1: AWS Secrets Manager (Recommended)
```python
import boto3
import json

def get_secret(secret_name: str):
    client = boto3.client('secretsmanager', region_name='us-east-1')
    response = client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])

# Usage
secrets = get_secret('athena-dashboard/prod')
AWS_ACCESS_KEY_ID = secrets['aws_access_key_id']
```

**Benefits:**
- Centralized secret management
- Automatic rotation support
- Audit logging
- IAM-based access control

#### Option 2: IAM Roles (Best Practice)
```python
# No credentials needed - use IAM role
# Backend runs on EC2/ECS with IAM role attached
# boto3 automatically uses role credentials
```

**Benefits:**
- No credential storage needed
- Automatic credential rotation
- Least privilege access
- AWS best practice

#### Option 3: Environment Variables (Container)
```yaml
# Docker/Kubernetes secrets
# Injected at runtime, not in code
```

**Benefits:**
- Simple for containerized deployments
- Works with any orchestrator
- Can be encrypted at rest

## API Security

### Request Validation

#### Input Sanitization
```python
from pydantic import BaseModel, validator

class QueryRequest(BaseModel):
    start_date: str
    end_date: str
    filters: dict
    
    @validator('start_date', 'end_date')
    def validate_date_format(cls, v):
        # Validate ISO 8601 format
        # Prevent SQL injection via date parsing
        pass
```

#### SQL Injection Prevention
- **Parameterized Queries**: Never concatenate user input into SQL
- **Query Builder**: Use structured query builder, not raw SQL
- **Whitelist**: Only allow specific columns, operators
- **Validation**: Validate all query parameters

### Rate Limiting

#### Implementation
```python
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@router.post("/api/v1/analytics/query")
@limiter.limit("10/minute")  # 10 queries per minute per IP
async def query_analytics(request: Request):
    pass
```

**Limits:**
- **Authentication**: 5 attempts per minute
- **Query Endpoints**: 10 queries per minute per user
- **Export Endpoints**: 5 exports per hour per user

### CORS Configuration

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://dashboard.example.com"],  # Production
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
    max_age=3600,
)
```

**Development:**
- Allow `http://localhost:3000` (or frontend dev server)
- **Production**: Strict origin whitelist

## Authorization (Role-Based Access Control)

### Roles (v1)

#### Admin
- Full access to all features
- User management (future)
- System settings

#### Analyst
- Full query access
- Export capabilities
- Saved queries

#### Viewer
- Read-only access
- Limited query complexity
- No export (or limited export)

### Implementation

#### Backend Middleware
```python
from functools import wraps

def require_role(allowed_roles: List[str]):
    def decorator(func):
        @wraps(func)
        async def wrapper(request: Request, *args, **kwargs):
            user = request.state.user  # Set by auth middleware
            if user.role not in allowed_roles:
                raise HTTPException(403, "Insufficient permissions")
            return await func(request, *args, **kwargs)
        return wrapper
    return decorator

@router.post("/api/v1/admin/users")
@require_role(["admin"])
async def create_user():
    pass
```

#### Frontend Route Guards
```typescript
// Protected route component
const ProtectedRoute = ({ allowedRoles, children }) => {
    const { user } = useAuth();
    
    if (!user || !allowedRoles.includes(user.role)) {
        return <Navigate to="/unauthorized" />;
    }
    
    return children;
};
```

## Data Security

### Athena Query Security

#### Query Scoping
- **User Context**: Include user/tenant context in queries (future multi-tenant)
- **Data Filtering**: Apply row-level security filters
- **Column Masking**: Hide sensitive columns based on role

#### Query Logging
```python
# Log all queries for audit
logger.info("Athena query executed", extra={
    "user": user.username,
    "query": query_string,  # Sanitized
    "execution_time": duration,
    "result_rows": row_count
})
```

### Data Export Security

#### Export Controls
- **Role-Based**: Only certain roles can export
- **Rate Limiting**: Prevent bulk data extraction
- **Audit Trail**: Log all exports (who, what, when)
- **Data Sanitization**: Remove sensitive fields if needed

## Security Headers

### HTTP Security Headers
```python
@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    response.headers["Strict-Transport-Security"] = "max-age=31536000"
    response.headers["Content-Security-Policy"] = "default-src 'self'"
    return response
```

## Audit Logging

### What to Log
- **Authentication**: Login attempts, failures, logouts
- **Authorization**: Permission denied events
- **Data Access**: Query executions, exports
- **Configuration Changes**: Settings modifications
- **Errors**: Security-related errors

### Log Format
```json
{
    "timestamp": "2024-01-15T10:30:00Z",
    "event_type": "query_executed",
    "user": "analyst@example.com",
    "ip_address": "192.168.1.100",
    "query_id": "abc123",
    "execution_time_ms": 1250,
    "result_rows": 500
}
```

## Security Checklist

### Development
- [ ] Environment variables in `.env` (gitignored)
- [ ] JWT secret key is strong (32+ characters)
- [ ] Input validation on all endpoints
- [ ] SQL injection prevention
- [ ] CORS configured correctly
- [ ] Rate limiting implemented
- [ ] Error messages don't leak sensitive info

### Production
- [ ] HTTPS only (TLS 1.2+)
- [ ] Secrets in AWS Secrets Manager or IAM roles
- [ ] Security headers configured
- [ ] Audit logging enabled
- [ ] Regular security updates
- [ ] Penetration testing (future)
- [ ] Vulnerability scanning (future)
