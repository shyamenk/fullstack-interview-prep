# DevOps & Deployment Interview Questions & Answers

## Table of Contents

1. [What is a CI/CD Pipeline using GitHub Actions?](#q1-what-is-a-cicd-pipeline-using-github-actions)
2. [What are the core components of GitHub Actions and how do they work together?](#q2-what-are-the-core-components-of-github-actions-and-how-do-they-work-together)

---

## Q1. What is a CI/CD Pipeline using GitHub Actions?

### Concept:

A CI/CD Pipeline automates the process of **testing, building, and deploying** code. GitHub Actions runs these pipelines directly in your GitHub repository.

- **CI (Continuous Integration):** Automatically test and build code on every commit/PR
- **CD (Continuous Deployment):** Automatically deploy to production when tests pass

### How It Works:

1. Developer pushes code to GitHub
2. GitHub Actions **triggers** a workflow
3. Workflow runs **tests** (CI)
4. If tests pass → **builds** the application
5. If build succeeds → **deploys** to server (CD)

### Basic CI/CD Workflow:

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install dependencies
        run: npm install
      - name: Run tests
        run: npm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build application
        run: npm run build

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: |
          echo "Deploying to server..."
          # SSH into server, pull code, restart services
```

### Pipeline Stages:

| Stage | Purpose | Example |
|-------|---------|---------|
| **Test** | Verify code quality | Run Jest, ESLint |
| **Build** | Compile/package application | `npm run build`, Docker build |
| **Deploy** | Push to production | Deploy to AWS, Vercel, Heroku |

### Real-World Example (Node.js + Docker + AWS):

```yaml
name: Deploy to AWS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Run tests
        run: |
          npm install
          npm test

      - name: Build Docker image
        run: docker build -t myapp:latest .

      - name: Push to ECR
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin
          docker tag myapp:latest 123456.dkr.ecr.us-east-1.amazonaws.com/myapp:latest
          docker push 123456.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

      - name: Deploy to ECS
        run: aws ecs update-service --cluster prod --service myapp --force-new-deployment
```

### Benefits:

| Without CI/CD | With CI/CD |
|---------------|------------|
| Manual testing (error-prone) | Automated tests on every commit |
| Manual deployment (slow) | One-click or auto deployment |
| Bugs reach production | Catch bugs before merge |
| Deployment takes hours | Deploy in minutes |

### Common Use Cases:

- Run tests on every PR
- Deploy to staging on merge to `develop`
- Deploy to production on merge to `main`
- Build and publish Docker images
- Run security scans and linting

### Analogy:

Think of a **car assembly line**:
- Each station (job) performs a specific task (test, build, deploy)
- If one station fails, the car doesn't move forward
- Everything is automated, fast, and consistent

### ⚡ One-liner to remember:
> CI/CD = Automate test, build, deploy so code reaches production faster and safer.

---

## Q2. What are the core components of GitHub Actions and how do they work together?

### Concept:

GitHub Actions has 5 core components that work together to automate workflows:

1. **Workflow** - The entire automation process
2. **Event** - What triggers the workflow
3. **Job** - A set of steps that run on the same runner
4. **Step** - Individual task within a job
5. **Action** - Reusable code block (pre-built or custom)

### Anatomy of GitHub Actions:

```yaml
# .github/workflows/example.yml

# 1. WORKFLOW
name: My Workflow

# 2. EVENT (trigger)
on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight

# 3. JOB
jobs:
  build:
    runs-on: ubuntu-latest  # Runner environment

    # 4. STEPS
    steps:
      # 5. ACTION (pre-built)
      - uses: actions/checkout@v3

      # 4. STEP (run command)
      - name: Install dependencies
        run: npm install

      # 4. STEP (run script)
      - name: Run tests
        run: npm test
```

### Core Components Explained:

#### 1. Workflow
The entire automation file (`.yml` in `.github/workflows/`)

```yaml
name: CI Pipeline  # Workflow name
on: [push]         # When to run
jobs: {...}        # What to do
```

#### 2. Event (Trigger)
What causes the workflow to run

```yaml
on:
  push:              # On code push
    branches: [main]
  pull_request:      # On PR creation
  workflow_dispatch: # Manual trigger
  schedule:          # Cron schedule
    - cron: '0 0 * * 0'
```

#### 3. Job
A set of steps that run on the same runner (can run in parallel)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps: [...]

  build:
    needs: test      # Runs only after 'test' succeeds
    runs-on: ubuntu-latest
    steps: [...]
```

#### 4. Step
Individual task within a job (runs sequentially)

```yaml
steps:
  - name: Checkout code
    uses: actions/checkout@v3

  - name: Run linter
    run: npm run lint

  - name: Run tests
    run: npm test
```

#### 5. Action
Reusable code block (from marketplace or custom)

```yaml
# Pre-built action
- uses: actions/checkout@v3

# Pre-built action with inputs
- uses: actions/setup-node@v3
  with:
    node-version: '18'

# Custom action
- uses: ./.github/actions/my-custom-action
```

### How They Work Together:

```
┌─────────────────────────────────────────────┐
│ WORKFLOW (entire automation)                │
│                                             │
│  ┌────────────────────────────────────┐    │
│  │ EVENT (trigger)                    │    │
│  │ on: push to main                   │    │
│  └────────────────────────────────────┘    │
│                                             │
│  ┌────────────────────────────────────┐    │
│  │ JOB: test                          │    │
│  │  ┌──────────────────────────────┐  │    │
│  │  │ STEP: Checkout code          │  │    │
│  │  │  ACTION: actions/checkout@v3 │  │    │
│  │  └──────────────────────────────┘  │    │
│  │  ┌──────────────────────────────┐  │    │
│  │  │ STEP: Run tests              │  │    │
│  │  │  RUN: npm test               │  │    │
│  │  └──────────────────────────────┘  │    │
│  └────────────────────────────────────┘    │
│                                             │
│  ┌────────────────────────────────────┐    │
│  │ JOB: deploy (needs: test)          │    │
│  │  ┌──────────────────────────────┐  │    │
│  │  │ STEP: Deploy to AWS          │  │    │
│  │  └──────────────────────────────┘  │    │
│  └────────────────────────────────────┘    │
└─────────────────────────────────────────────┘
```

### Complete Example with All Components:

```yaml
# WORKFLOW
name: Full CI/CD Pipeline

# EVENT
on:
  push:
    branches: [main, develop]
  pull_request:
    types: [opened, synchronize]

# JOBS
jobs:
  # JOB 1: Lint and Test
  quality-check:
    runs-on: ubuntu-latest

    # STEPS
    steps:
      # ACTION (pre-built)
      - uses: actions/checkout@v3

      # ACTION (pre-built with config)
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      # STEP (run command)
      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run tests
        run: npm test

  # JOB 2: Build (runs after quality-check)
  build:
    needs: quality-check
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Build application
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: build-files
          path: dist/

  # JOB 3: Deploy (runs after build, only on main branch)
  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - uses: actions/checkout@v3

      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: build-files

      - name: Deploy to AWS S3
        run: aws s3 sync dist/ s3://my-bucket --delete
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Component Relationships:

| Component | Contains | Purpose |
|-----------|----------|---------|
| Workflow | Events + Jobs | Entire automation |
| Event | - | When to run workflow |
| Job | Steps | Group of related tasks |
| Step | Actions or Commands | Individual task |
| Action | Code | Reusable functionality |

### Common Events:

```yaml
on:
  push:                    # Code pushed
  pull_request:            # PR opened/updated
  release:                 # Release created
  workflow_dispatch:       # Manual trigger
  schedule:                # Cron schedule
  issues:                  # Issue opened/closed
  workflow_call:           # Called by another workflow
```

### Job Dependencies:

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps: [...]

  build:
    needs: test           # Wait for test to complete
    runs-on: ubuntu-latest
    steps: [...]

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    steps: [...]

  deploy-prod:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'  # Conditional
    steps: [...]
```

### Analogy:

Think of a **restaurant kitchen**:
- **Workflow** = Entire dinner service
- **Event** = Customer order arrives (trigger)
- **Job** = Station (appetizers, mains, desserts)
- **Step** = Task within station (chop, cook, plate)
- **Action** = Pre-made sauce or tool you reuse

### ⚡ One-liner to remember:
> Event triggers Workflow → Workflow runs Jobs → Jobs execute Steps → Steps use Actions.

---
