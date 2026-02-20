---
name: create-claude-md
description: Expert knowledge for managing Claude Code memory with CLAUDE.md files. Use when the user wants to give Claude persistent project context, document patterns, or manage Claude's auto memory system.
---

# Manage Claude Code Memory with CLAUDE.md

You are an expert in managing Claude Code's memory system. Guide users through creating and organizing CLAUDE.md files that give Claude persistent context about their projects.

## What is CLAUDE.md?

`CLAUDE.md` is a **project memory file** that Claude automatically reads at the start of every conversation. It contains information about:

- Project structure and architecture
- Coding conventions and style guides
- Important patterns and gotchas
- Commands and workflows
- Team context and decisions

**Key benefits**:
- **Always available**: Loaded automatically in every session
- **Persistent knowledge**: Share context across conversations
- **Team alignment**: Version control memory with your code
- **Hierarchical**: Organize with imports and directory-specific files

## CLAUDE.md vs Auto Memory

Claude Code has two memory systems:

| Feature | CLAUDE.md | Auto Memory |
|:--------|:----------|:------------|
| **Location** | `CLAUDE.md` in project | `.claude/auto-memory/` |
| **Management** | You write and edit | Claude writes automatically |
| **Version control** | Yes, commit to git | No, gitignored |
| **When loaded** | Every session start | Every session start |
| **Purpose** | Explicit project context | Claude's notes about project |
| **Scope** | Project-wide | Project-specific |

**Use both**: CLAUDE.md for things you want to communicate, auto memory for things Claude learns over time.

## Where CLAUDE.md Files Live

### Standard Location

```
project-root/
├── CLAUDE.md          # Main project memory
└── src/
    └── CLAUDE.md      # Component-specific memory
```

### Hierarchical Loading

Claude loads CLAUDE.md files from:
1. **Current working directory** upward to project root
2. **All parent directories** in the path
3. **Imports** referenced in CLAUDE.md files

**Example**:
```
/project/CLAUDE.md              # Always loaded
/project/src/CLAUDE.md          # Loaded when cwd is in src/
/project/src/components/CLAUDE.md  # Loaded when cwd is in components/
```

## Create Your First CLAUDE.md

```bash
cat > CLAUDE.md << 'EOF'
# Project Name

## Overview
Brief description of what this project does.

## Tech Stack
- Framework: React + TypeScript
- Build: Vite
- Testing: Vitest + React Testing Library
- Styling: Tailwind CSS

## Important Commands
- `npm run dev` - Start dev server
- `npm run build` - Build for production
- `npm test` - Run test suite
- `npm run lint` - Check code quality

## Code Conventions
- Use functional components with hooks
- Prefer named exports over default
- Keep components under 200 lines
- Write tests alongside components

## Architecture
- `/src/components` - Reusable UI components
- `/src/features` - Feature-specific code
- `/src/lib` - Shared utilities
- `/src/api` - API client code
EOF
```

## Content Guidelines

### What to Include

**Essential information**:
- Project purpose and goals
- Tech stack and dependencies
- Important commands
- Code conventions
- Architecture overview
- Common workflows

**Helpful context**:
- Design decisions and rationale
- Known issues and workarounds
- Performance considerations
- Security requirements
- Testing strategy

**Team knowledge**:
- Naming conventions
- PR requirements
- Deployment process
- Contact information

### What to Avoid

**Don't include**:
- ❌ Obvious information (file names, directory listings)
- ❌ Generated documentation (use tools to generate on demand)
- ❌ Temporary todos (use task management instead)
- ❌ Sensitive information (tokens, passwords, keys)
- ❌ Very long reference docs (import them instead)

**Keep it concise**: Aim for 100-500 lines. Use imports for longer content.

## Advanced Features

### Import Other Files

Reference external documentation:

```markdown
# Main Project Memory

## Overview
...

## API Documentation
See [API.md](import:API.md) for complete API reference.

## Database Schema
See [schema.sql](import:docs/schema.sql) for database structure.
```

**Import syntax**: `[Link text](import:relative/path/to/file.md)`

**When to import**:
- Long reference documentation
- Generated files (OpenAPI specs, schema dumps)
- Files that change independently
- Content shared across multiple CLAUDE.md files

**Import limits**:
- Imports count toward context budget
- Deep import chains can exceed limits
- Use selectively for truly necessary content

### Directory-Specific Context

Create CLAUDE.md in subdirectories for component-specific context:

```
project/
├── CLAUDE.md                    # General project info
├── src/
│   └── api/
│       └── CLAUDE.md           # API-specific conventions
└── tests/
    └── CLAUDE.md               # Testing guidelines
```

**Example** (`src/api/CLAUDE.md`):
```markdown
# API Module

## Conventions
- All endpoints use RESTful naming
- Errors follow RFC 7807 Problem Details
- Authentication via Bearer tokens
- Rate limits: 100 req/min per token

## Common Patterns
```typescript
// Standard error handling
try {
  const result = await api.call();
} catch (error) {
  if (error instanceof ApiError) {
    // Handle API-specific errors
  }
}
```

## Testing
- Mock API calls with MSW
- Test error scenarios
- Verify request/response types
```

### Dynamic Content

Use shell commands to inject dynamic information:

```markdown
# Project Status

## Current Branch
Working on: !`git rev-parse --abbrev-ref HEAD`

## Recent Changes
!`git log --oneline -5`

## Dependencies
!`npm list --depth=0 2>/dev/null || echo "Run npm install"`
```

**Syntax**: `` !`shell command` ``

**Commands execute** when Claude loads the file. Output replaces the command.

**Use cases**:
- Current git branch
- Latest commit info
- Dependency versions
- Environment status

**Caution**: Commands run on every session start. Keep them fast.

## Example CLAUDE.md Files

### Web Application

```markdown
# MyApp Web Application

## Overview
SaaS platform for team collaboration built with Next.js and PostgreSQL.

## Tech Stack
- Framework: Next.js 14 (App Router)
- Language: TypeScript (strict mode)
- Database: PostgreSQL via Prisma
- Styling: Tailwind CSS
- Auth: NextAuth.js
- Deployment: Vercel

## Getting Started
```bash
npm install
npm run dev         # http://localhost:3000
npm run db:migrate  # Run migrations
npm test           # Run test suite
```

## Architecture

### Directory Structure
- `/app` - Next.js app router pages
- `/components` - Reusable UI components
- `/lib` - Shared utilities and helpers
- `/prisma` - Database schema and migrations
- `/public` - Static assets

### Key Patterns
- Server Components by default
- Client Components only when needed (use "use client")
- API routes in `/app/api`
- Database queries in Server Components or API routes

## Code Conventions

### Components
- Functional components with TypeScript
- Named exports preferred
- Props interfaces defined inline or nearby
- Keep components under 200 lines

### Styling
- Tailwind utility classes
- Custom utilities in `tailwind.config.js`
- Avoid inline styles
- Use `cn()` helper for conditional classes

### Database
- Prisma for all database access
- Migrations for schema changes
- Seed data in `prisma/seed.ts`

## Testing
- Unit tests: Vitest
- E2E tests: Playwright
- Run tests before committing
- Test coverage goal: 80%

## Important Notes
- API routes are rate-limited (100 req/min)
- Use React Server Components when possible
- Images must be optimized before upload
- All user inputs must be sanitized

## Common Tasks

### Add a new page
1. Create file in `/app/(route)/page.tsx`
2. Update navigation in `/components/nav.tsx`
3. Add tests in `/app/(route)/page.test.tsx`

### Add a database table
1. Update `prisma/schema.prisma`
2. Run `npm run db:migrate`
3. Update TypeScript types will auto-generate
```

### API Service

```markdown
# Payment Processing API

## Overview
RESTful API for processing payments and managing subscriptions.

## Stack
- Runtime: Node.js 20
- Framework: Express + TypeScript
- Database: PostgreSQL
- Cache: Redis
- Queue: Bull
- Deployment: AWS ECS

## Development
```bash
npm install
npm run dev         # http://localhost:3000
npm test           # Run tests
npm run db:migrate # Database migrations
```

## API Conventions

### Endpoints
- RESTful naming: `/api/v1/resource`
- Versioning in URL path
- Plural resource names

### Authentication
- Bearer token in `Authorization` header
- Tokens expire after 1 hour
- Refresh tokens valid for 30 days

### Error Responses
All errors follow RFC 7807:
```json
{
  "type": "https://api.example.com/errors/invalid-card",
  "title": "Invalid Card",
  "status": 400,
  "detail": "Card number is invalid",
  "instance": "/api/v1/payments/123"
}
```

### Rate Limits
- 1000 requests per hour per API key
- 429 status when exceeded
- `Retry-After` header indicates wait time

## Database

### Conventions
- Snake_case for tables and columns
- Always include `created_at`, `updated_at`
- Soft deletes with `deleted_at`
- Use UUIDs for IDs

### Migrations
- Located in `/db/migrations`
- Run with `npm run db:migrate`
- Never edit past migrations

## Testing

### Levels
- Unit tests: Individual functions
- Integration tests: API endpoints
- E2E tests: Full payment flows

### Coverage
- Minimum 80% coverage required
- Critical paths: 100% coverage
- Run `npm run test:coverage`

## Security

### Requirements
- All inputs validated with Zod
- SQL injection prevention (use parameterized queries)
- Rate limiting on all endpoints
- Sensitive data encrypted at rest
- PCI DSS compliance for card data

### Secrets Management
- Never commit secrets
- Use AWS Secrets Manager
- Rotate keys quarterly

## Deployment

### Environments
- `dev` - Development (auto-deploy from main)
- `staging` - Pre-production (manual deploy)
- `prod` - Production (manual deploy + approval)

### Process
1. Create PR with changes
2. Pass CI/CD checks (tests, lint, security scan)
3. Get approval from team lead
4. Merge to main (auto-deploys to dev)
5. Test in dev environment
6. Deploy to staging for QA
7. Deploy to prod after approval

## Important Notes
- Payment webhooks are idempotent
- All monetary amounts in cents (integer)
- Timezone: All dates in UTC
- Async jobs processed by Bull queue
- Failed payments retry 3 times with backoff
```

### Python Data Science Project

```markdown
# Data Analysis Pipeline

## Overview
ETL pipeline and analysis tools for customer behavior data.

## Stack
- Python: 3.11+
- Data: pandas, numpy
- Analysis: scikit-learn, scipy
- Viz: matplotlib, seaborn
- DB: PostgreSQL via SQLAlchemy
- Workflow: Apache Airflow

## Setup
```bash
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install -r requirements.txt
```

## Code Style

### Formatting
- Black for code formatting
- isort for import sorting
- flake8 for linting
- mypy for type checking

### Conventions
- Type hints on all functions
- Docstrings (Google style)
- Max line length: 88 (Black default)
- Snake_case for variables and functions

## Project Structure
- `/pipelines` - Airflow DAGs
- `/analysis` - Jupyter notebooks
- `/src` - Shared Python modules
- `/tests` - Test suite
- `/data` - Local data (gitignored)
- `/models` - Trained ML models

## Data Conventions

### DataFrames
- Column names: lowercase with underscores
- Datetime columns: suffix with `_at` or `_date`
- Boolean columns: prefix with `is_` or `has_`
- Keep data immutable when possible

### File Formats
- Raw data: CSV or Parquet
- Processed data: Parquet (compressed)
- Models: Pickle or joblib
- Configs: YAML

## Testing
- pytest for all tests
- Test coverage: 85%+ required
- Data validation tests for all inputs
- Mock external APIs

## Notebooks

### Best Practices
- Clear cell execution order
- Remove output before committing
- Extract reusable code to `/src`
- Document assumptions and decisions

### Naming
- `exploratory_` - Initial exploration
- `analysis_` - Finalized analysis
- `model_` - Model training/evaluation

## Common Tasks

### Run pipeline
```bash
airflow dags trigger customer_behavior_pipeline
```

### Train model
```bash
python -m src.models.train --data data/processed/train.parquet
```

### Generate report
```bash
python -m src.reports.generate --period 2024-01
```

## Important Notes
- All dates stored in UTC
- PII must be anonymized before analysis
- Results are cached (check `/cache` first)
- Large datasets: Use Dask for distributed computing
- Always validate data quality before analysis
```

### Mobile App

```markdown
# TaskMaster Mobile App

## Overview
Task management app for iOS and Android built with React Native.

## Tech Stack
- Framework: React Native 0.73
- Language: TypeScript
- Navigation: React Navigation
- State: Redux Toolkit + RTK Query
- Backend: Firebase (Auth, Firestore, Storage)
- Deployment: EAS (Expo Application Services)

## Development Setup
```bash
npm install
npx expo start  # Open in Expo Go app
npm run ios     # iOS simulator
npm run android # Android emulator
npm test        # Run tests
```

## Architecture

### Directory Structure
- `/src/screens` - Screen components
- `/src/components` - Reusable UI components
- `/src/store` - Redux slices and API
- `/src/navigation` - Navigation config
- `/src/hooks` - Custom React hooks
- `/src/utils` - Helper functions
- `/assets` - Images, fonts, icons

### State Management
- Redux for global state
- RTK Query for API calls
- Local component state for UI-only state
- AsyncStorage for persistence

## Code Conventions

### Components
- Functional components only
- TypeScript with strict types
- Props interfaces exported
- Styled with StyleSheet.create()

### Styling
- No inline styles
- Responsive with Dimensions API
- Theme constants in `/src/theme`
- Platform-specific code with Platform.select()

### Navigation
- Type-safe navigation with TypeScript
- Deep linking configured
- Bottom tabs for main sections
- Stack navigation within sections

## Firebase

### Collections
- `users` - User profiles
- `tasks` - Task documents
- `projects` - Project data
- `settings` - User preferences

### Security Rules
- Read: Authenticated users only
- Write: Owner of document only
- Validate data structure server-side

## Testing
- Jest + React Native Testing Library
- Test user interactions, not implementation
- Mock Firebase calls
- Snapshot tests for static UI

## Building

### Development
- Expo Go app for live reload
- EAS Build for standalone development builds

### Production
- EAS Build for production binaries
- Submit to App Store / Play Store via EAS Submit
- Version format: X.Y.Z (semantic versioning)

## Important Notes
- Offline support via AsyncStorage
- Images compressed before upload (max 1MB)
- Push notifications via Firebase Cloud Messaging
- Analytics tracked with Firebase Analytics
- Crash reporting with Sentry

## Common Issues

### iOS Simulator Not Starting
```bash
npx expo run:ios --clean
```

### Android Build Fails
Check Java version: `java --version` (needs Java 11)

### Firebase Connection Issues
Verify google-services.json and GoogleService-Info.plist are present
```

## Organize Large Projects

### Strategy 1: Topic Imports

Keep main CLAUDE.md concise, import details:

```markdown
# Large Application

## Overview
Enterprise CRM platform with multiple services.

## Architecture
See [ARCHITECTURE.md](import:docs/ARCHITECTURE.md)

## API Documentation
See [API.md](import:docs/API.md)

## Database
See [DATABASE.md](import:docs/DATABASE.md)

## Quick Reference
- Authentication: JWT tokens (1h expiry)
- Rate limits: 1000 req/hour
- Deployment: Kubernetes on AWS
```

### Strategy 2: Service-Specific Files

Monorepo with multiple CLAUDE.md files:

```
monorepo/
├── CLAUDE.md              # Shared conventions
├── services/
│   ├── api/
│   │   └── CLAUDE.md     # API service specifics
│   ├── web/
│   │   └── CLAUDE.md     # Web app specifics
│   └── worker/
│       └── CLAUDE.md     # Worker service specifics
```

Each service CLAUDE.md contains only relevant information.

### Strategy 3: Progressive Disclosure

Start with essentials, link to details:

```markdown
# Project Memory

## Essential Info
Key information Claude needs immediately...

## Detailed Documentation
When you need specific details:
- API: `docs/api.md`
- Database: `docs/database.md`
- Deployment: `docs/deployment.md`

Claude, only import these if needed for the current task.
```

## Auto Memory

Claude automatically takes notes about your project in `.claude/auto-memory/`.

### How It Works

**Automatic note-taking**:
- Claude observes patterns during work
- Records insights, conventions, gotcas
- Organizes notes by topic
- Updates over time as understanding grows

**Storage**:
- Location: `.claude/auto-memory/MEMORY.md`
- Gitignored by default
- Per-project, not shared
- Automatically loaded each session

### Viewing Auto Memory

```
Show me your auto memory for this project
```

Claude displays current memory notes.

### Managing Auto Memory

**Clear all memory**:
```bash
rm -rf .claude/auto-memory
```

**Clear specific memories**:
Edit `.claude/auto-memory/MEMORY.md` directly.

**Disable auto memory**:
```json
{
  "memory": {
    "autoMemory": false
  }
}
```

**Convert to CLAUDE.md**:
Copy useful insights from auto memory to CLAUDE.md for version control.

### Auto Memory vs CLAUDE.md

**Use auto memory for**:
- Claude's observations
- Discovered patterns
- Learned preferences
- Temporary context

**Use CLAUDE.md for**:
- Team knowledge
- Official conventions
- Critical information
- Version-controlled context

## Best Practices

1. **Start small**: Begin with essential info, expand as needed
2. **Keep it current**: Update when conventions change
3. **Be specific**: "Use camelCase" not "follow style guide"
4. **Include examples**: Show, don't just tell
5. **Version control**: Commit CLAUDE.md with code
6. **Review regularly**: Update during retrospectives
7. **Use hierarchy**: Directory-specific files for subsystems
8. **Link don't duplicate**: Import shared content
9. **Think onboarding**: What would a new teammate need?
10. **Test it**: Start a new session and see if Claude understands

## Common Patterns

### Commands Section

```markdown
## Important Commands

### Development
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm test` - Run test suite

### Database
- `npm run db:migrate` - Run migrations
- `npm run db:seed` - Seed database
- `npm run db:reset` - Reset database

### Deployment
- `npm run deploy:staging` - Deploy to staging
- `npm run deploy:prod` - Deploy to production
```

### Gotchas Section

```markdown
## Known Issues & Workarounds

### Issue: Test flakiness in CI
The user login tests sometimes fail in CI due to timing.
**Workaround**: Added retry logic (3 attempts max).

### Issue: Large image uploads timeout
Files over 5MB fail to upload.
**Workaround**: Client-side compression before upload.

### Issue: Database connection pool exhaustion
Under heavy load, connections not released properly.
**Workaround**: Added explicit connection cleanup in finally blocks.
```

### Architecture Section

```markdown
## Architecture

### System Overview
Microservices architecture with:
- API Gateway (Kong)
- 5 backend services (Node.js)
- PostgreSQL (primary database)
- Redis (caching + sessions)
- RabbitMQ (message queue)

### Service Communication
- Synchronous: REST over HTTP
- Asynchronous: Events via RabbitMQ
- Service discovery: Consul

### Data Flow
1. Request hits API Gateway
2. Gateway routes to appropriate service
3. Service processes, emits events if needed
4. Other services consume events asynchronously
5. Response returned to client
```

## Troubleshooting

**CLAUDE.md not loading**:
- Verify filename is exactly `CLAUDE.md` (case-sensitive)
- Check file is in project directory or above current working directory
- Ensure file encoding is UTF-8
- Check for YAML frontmatter errors if using

**Import not working**:
- Use relative paths from CLAUDE.md location
- Verify imported file exists at path
- Check for circular imports
- Verify import syntax: `[text](import:path/to/file.md)`

**Too much content**:
- CLAUDE.md files count toward context window
- Keep main file under 500 lines
- Use imports for long content
- Consider directory-specific files

**Dynamic commands failing**:
- Commands must complete quickly (< 1 second)
- Check command works in shell independently
- Verify required tools are installed
- Use absolute paths if needed

## Quick Start Template

```bash
cat > CLAUDE.md << 'EOF'
# Project Name

## Overview
Brief description of the project and its purpose.

## Tech Stack
- Language/Framework:
- Database:
- Key libraries:

## Getting Started
```bash
# Installation
npm install

# Development
npm run dev

# Testing
npm test
```

## Code Conventions
- Convention 1
- Convention 2
- Convention 3

## Architecture
Brief overview of how the system is organized.

## Important Notes
- Critical information
- Common gotchas
- Things to remember
EOF
```

## Related Features

- Use `/create-skill` to add domain-specific knowledge
- Use `/create-agent` to create memory-enabled agents
- Use hooks to keep CLAUDE.md updated
- Check auto memory for learned patterns

## Additional Resources

- Official docs: https://code.claude.com/docs/en/memory
- CLAUDE.md examples: https://github.com/anthropics/claude-code-examples
- Agent Skills standard: https://agentskills.io

## Summary

**CLAUDE.md gives Claude persistent memory**:
- Loaded automatically every session
- Version controlled with your code
- Hierarchical organization
- Dynamic content via shell commands
- Importable external documentation

**Best for**:
- Project conventions
- Architecture overview
- Common commands
- Team knowledge
- Important patterns

**Key points**:
- Keep it concise (100-500 lines)
- Use imports for long content
- Update when conventions change
- Commit to version control
- Complement with auto memory

Start with essentials, expand as your project grows. CLAUDE.md helps Claude understand your project deeply from the first message.
