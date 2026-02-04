# Project Skeleton Structure

## Root Directory Structure

```
athena_dashboard/
├── backend/                 # FastAPI backend application
├── frontend/                # React frontend application
├── docs/                    # Documentation (this folder)
├── .gitignore              # Git ignore rules
├── README.md               # Project overview
├── docker-compose.yml      # Local development (future)
└── LICENSE                 # License file
```

## Backend Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py              # FastAPI application entry point
│   ├── config.py            # Configuration & settings
│   │
│   ├── api/                 # API routes
│   │   ├── __init__.py
│   │   ├── deps.py          # Dependencies (auth, db, etc.)
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── auth.py      # Authentication endpoints
│   │       ├── analytics.py # Analytics/query endpoints
│   │       ├── dashboard.py # Dashboard data endpoints
│   │       └── health.py    # Health check endpoint
│   │
│   ├── core/                # Core functionality
│   │   ├── __init__.py
│   │   ├── security.py      # JWT, password hashing
│   │   ├── config.py        # Settings management
│   │   └── exceptions.py    # Custom exceptions
│   │
│   ├── services/            # Business logic
│   │   ├── __init__.py
│   │   ├── auth_service.py  # Authentication logic
│   │   ├── athena_service.py # Athena query execution
│   │   ├── query_service.py # Query construction & validation
│   │   ├── cache_service.py # Caching logic
│   │   └── export_service.py # Data export (future)
│   │
│   ├── models/              # Data models
│   │   ├── __init__.py
│   │   ├── user.py          # User model
│   │   ├── query.py         # Query request/response models
│   │   └── analytics.py     # Analytics data models
│   │
│   ├── schemas/             # Pydantic schemas
│   │   ├── __init__.py
│   │   ├── auth.py          # Auth request/response schemas
│   │   ├── query.py         # Query schemas
│   │   └── analytics.py     # Analytics schemas
│   │
│   └── utils/               # Utility functions
│       ├── __init__.py
│       ├── logging.py       # Logging configuration
│       └── validators.py    # Custom validators
│
├── tests/                   # Test suite
│   ├── __init__.py
│   ├── conftest.py          # Pytest configuration
│   ├── test_auth.py
│   ├── test_athena.py
│   └── test_api.py
│
├── scripts/                 # Utility scripts
│   ├── setup_env.py         # Environment setup
│   └── seed_data.py         # Seed test data (future)
│
├── .env.example             # Environment variable template
├── .env                     # Local environment (gitignored)
├── pyproject.toml           # Python project config (Poetry)
├── poetry.lock              # Dependency lock file
└── README.md                # Backend-specific docs
```

### Backend File Naming Conventions

- **Files**: `snake_case.py`
- **Classes**: `PascalCase`
- **Functions/Variables**: `snake_case`
- **Constants**: `UPPER_SNAKE_CASE`
- **Private**: `_leading_underscore`

## Frontend Structure

```
frontend/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── assets/              # Static assets
│
├── src/
│   ├── main.tsx             # Application entry point
│   ├── App.tsx              # Root component
│   ├── vite-env.d.ts        # Vite type definitions
│   │
│   ├── api/                 # API client
│   │   ├── client.ts        # Axios instance, interceptors
│   │   ├── auth.ts          # Auth API calls
│   │   ├── analytics.ts     # Analytics API calls
│   │   └── types.ts         # API response types
│   │
│   ├── components/          # Reusable components
│   │   ├── common/          # Generic components
│   │   │   ├── Button/
│   │   │   ├── Card/
│   │   │   ├── Loading/
│   │   │   └── ErrorBoundary/
│   │   │
│   │   ├── charts/          # Chart components
│   │   │   ├── LineChart/
│   │   │   ├── BarChart/
│   │   │   ├── PieChart/
│   │   │   └── ChartContainer/
│   │   │
│   │   ├── tables/          # Table components
│   │   │   ├── DataTable/
│   │   │   └── TableToolbar/
│   │   │
│   │   ├── filters/         # Filter components
│   │   │   ├── DateRangePicker/
│   │   │   ├── MultiSelect/
│   │   │   └── FilterBar/
│   │   │
│   │   └── layout/          # Layout components
│   │       ├── Header/
│   │       ├── Sidebar/
│   │       ├── Footer/
│   │       └── PageLayout/
│   │
│   ├── features/            # Feature modules
│   │   ├── auth/
│   │   │   ├── components/
│   │   │   │   └── LoginForm/
│   │   │   ├── hooks/
│   │   │   │   └── useAuth.ts
│   │   │   └── types.ts
│   │   │
│   │   ├── dashboard/
│   │   │   ├── components/
│   │   │   │   ├── KPICard/
│   │   │   │   ├── DashboardGrid/
│   │   │   │   └── DashboardFilters/
│   │   │   ├── hooks/
│   │   │   │   └── useDashboard.ts
│   │   │   └── types.ts
│   │   │
│   │   └── analytics/
│   │       ├── components/
│   │       ├── hooks/
│   │       └── types.ts
│   │
│   ├── pages/               # Page components
│   │   ├── LoginPage.tsx
│   │   ├── DashboardPage.tsx
│   │   ├── AnalyticsPage.tsx
│   │   ├── SettingsPage.tsx
│   │   └── NotFoundPage.tsx
│   │
│   ├── store/               # State management
│   │   ├── authStore.ts     # Zustand store for auth
│   │   ├── preferencesStore.ts
│   │   └── queryClient.ts   # React Query client
│   │
│   ├── hooks/               # Shared hooks
│   │   ├── useApi.ts
│   │   ├── useDebounce.ts
│   │   └── useLocalStorage.ts
│   │
│   ├── utils/               # Utility functions
│   │   ├── formatters.ts    # Date, number formatters
│   │   ├── validators.ts    # Form validation
│   │   └── constants.ts     # Constants
│   │
│   ├── styles/              # Global styles
│   │   ├── theme.ts         # MUI theme configuration
│   │   ├── colors.ts        # Color palette
│   │   └── globals.css      # Global CSS
│   │
│   └── types/               # TypeScript types
│       ├── user.ts
│       ├── analytics.ts
│       └── api.ts
│
├── .env.example             # Environment variable template
├── .env.local               # Local environment (gitignored)
├── .eslintrc.js             # ESLint configuration
├── .prettierrc              # Prettier configuration
├── index.html
├── package.json
├── pnpm-lock.yaml           # pnpm lock file
├── tsconfig.json            # TypeScript configuration
├── vite.config.ts           # Vite configuration
└── README.md                # Frontend-specific docs
```

### Frontend File Naming Conventions

- **Files**: `PascalCase.tsx` (components), `camelCase.ts` (utilities)
- **Components**: `PascalCase`
- **Functions/Variables**: `camelCase`
- **Constants**: `UPPER_SNAKE_CASE`
- **Types/Interfaces**: `PascalCase`
- **Hooks**: `usePascalCase`

## Configuration Management

### Backend Configuration

#### Environment Variables
```python
# backend/app/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # AWS Configuration
    aws_access_key_id: str
    aws_secret_access_key: str
    aws_region: str = "us-east-1"
    
    # Athena Configuration
    athena_workgroup: str
    athena_database: str
    athena_table: str
    athena_output_s3: str | None = None
    
    # Application Configuration
    app_name: str = "Athena Dashboard API"
    app_version: str = "1.0.0"
    debug: bool = False
    
    # Security
    jwt_secret_key: str
    jwt_algorithm: str = "HS256"
    jwt_expiration_minutes: int = 15
    
    # CORS
    cors_origins: list[str] = ["http://localhost:3000"]
    
    # Cache
    cache_ttl_seconds: int = 300
    
    class Config:
        env_file = ".env"
        case_sensitive = False

settings = Settings()
```

### Frontend Configuration

#### Environment Variables
```typescript
// frontend/src/config/env.ts
export const config = {
    apiUrl: import.meta.env.VITE_API_URL || 'http://localhost:8000',
    appName: 'Athena Dashboard',
    version: import.meta.env.VITE_APP_VERSION || '1.0.0',
    enableDevTools: import.meta.env.DEV,
} as const;
```

#### Vite Environment Variables
```bash
# .env.local
VITE_API_URL=http://localhost:8000
VITE_APP_VERSION=1.0.0
```

## Documentation Structure

```
docs/
├── 01-system-architecture.md
├── 02-technology-stack.md
├── 03-ui-ux-design-plan.md
├── 04-security-authentication.md
├── 05-project-structure.md
├── 06-execution-plan.md
├── 07-future-extensibility.md
└── api/                      # API documentation (future)
    └── openapi.yaml          # OpenAPI spec (auto-generated)
```

## Git Structure

### Branching Strategy
```
main          # Production-ready code
├── develop   # Integration branch
    ├── feature/auth-implementation
    ├── feature/athena-integration
    └── feature/dashboard-ui
```

### .gitignore
```
# Backend
backend/.env
backend/__pycache__/
backend/.pytest_cache/
backend/.venv/
backend/poetry.lock  # Optional: commit for reproducibility

# Frontend
frontend/node_modules/
frontend/.env.local
frontend/dist/
frontend/.vite/

# IDE
.vscode/
.idea/
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/
```

## Development Workflow

### Local Development Setup

1. **Backend**
   ```bash
   cd backend
   poetry install
   cp .env.example .env
   # Edit .env with your credentials
   poetry run uvicorn app.main:app --reload
   ```

2. **Frontend**
   ```bash
   cd frontend
   pnpm install
   cp .env.example .env.local
   # Edit .env.local if needed
   pnpm dev
   ```

### Build Commands

**Backend:**
```bash
poetry run uvicorn app.main:app --host 0.0.0.0 --port 8000
```

**Frontend:**
```bash
pnpm build        # Production build
pnpm preview      # Preview production build
```

## Testing Structure

### Backend Tests
```
backend/tests/
├── unit/           # Unit tests
│   ├── test_services/
│   └── test_utils/
├── integration/    # Integration tests
│   └── test_api/
└── fixtures/       # Test data
```

### Frontend Tests (Future)
```
frontend/src/
├── __tests__/      # Component tests
└── __mocks__/      # Mock data
```

## Deployment Structure (Future)

```
deployment/
├── docker/
│   ├── Dockerfile.backend
│   └── Dockerfile.frontend
├── kubernetes/     # K8s manifests (future)
└── terraform/      # Infrastructure as code (future)
```
