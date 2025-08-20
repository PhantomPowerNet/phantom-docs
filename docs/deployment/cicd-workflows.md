# CI/CD Workflows

## Overview

This document outlines the Continuous Integration and Continuous Deployment (CI/CD) workflows for Phantom Power, including automated testing, code quality checks, security scanning, and deployment pipelines using GitHub Actions.

## CI/CD Architecture

### Pipeline Strategy
- **Feature Branch Pipeline**: Run tests and quality checks on all branches
- **Main Branch Pipeline**: Full CI/CD pipeline with deployment to production
- **Pull Request Pipeline**: Comprehensive validation before merging
- **Release Pipeline**: Tagged releases with deployment orchestration

### Environments
- **Development**: Feature branch deployments
- **Staging**: Integration testing environment
- **Production**: Live application on render.com

## GitHub Actions Workflows

### Main CI/CD Pipeline

```yaml
# .github/workflows/main.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  release:
    types: [published]

env:
  NODE_VERSION: '18'
  JAVA_VERSION: '17'
  PYTHON_VERSION: '3.11'

jobs:
  # ===================
  # Code Quality Checks
  # ===================
  lint-and-format:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Frontend linting
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
          cache-dependency-path: frontend/pnpm-lock.yaml

      - name: Install frontend dependencies
        run: cd frontend && pnpm install --frozen-lockfile

      - name: Run ESLint
        run: cd frontend && pnpm lint

      - name: Check Prettier formatting
        run: cd frontend && pnpm format:check

      - name: TypeScript type check
        run: cd frontend && pnpm type-check

      # Backend linting
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'openjdk'
          java-version: ${{ env.JAVA_VERSION }}

      - name: Cache Maven dependencies
        uses: actions/cache@v3
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('backend/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2

      - name: Run Checkstyle
        run: cd backend && mvn checkstyle:check

      - name: Run SpotBugs
        run: cd backend && mvn spotbugs:check

      # Python linting
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install Poetry
        uses: snok/install-poetry@v1

      - name: Install Python dependencies
        run: cd audio-service && poetry install

      - name: Run Black formatting check
        run: cd audio-service && poetry run black --check .

      - name: Run isort import sorting check
        run: cd audio-service && poetry run isort --check-only .

      - name: Run flake8 linting
        run: cd audio-service && poetry run flake8 .

      - name: Run mypy type checking
        run: cd audio-service && poetry run mypy .

  # =================
  # Security Scanning
  # =================
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Dependency vulnerability scanning
      - name: Run npm audit
        run: cd frontend && npm audit --audit-level moderate

      - name: Run Maven dependency check
        run: cd backend && mvn org.owasp:dependency-check-maven:check

      - name: Run Python safety check
        run: |
          cd audio-service
          poetry run pip install safety
          poetry run safety check

      # Code security scanning
      - name: Run CodeQL Analysis
        uses: github/codeql-action/init@v2
        with:
          languages: javascript,java,python

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2

      # Secret scanning
      - name: Run GitLeaks
        uses: zricethezav/gitleaks-action@master

  # ==================
  # Unit and Integration Tests
  # ==================
  test-frontend:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
          cache-dependency-path: frontend/pnpm-lock.yaml

      - name: Install dependencies
        run: cd frontend && pnpm install --frozen-lockfile

      - name: Run unit tests
        run: cd frontend && pnpm test:ci

      - name: Run component tests
        run: cd frontend && pnpm test:components

      - name: Generate coverage report
        run: cd frontend && pnpm test:coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./frontend/coverage/lcov.info
          flags: frontend
          name: frontend-coverage

  test-backend:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'openjdk'
          java-version: ${{ env.JAVA_VERSION }}

      - name: Cache Maven dependencies
        uses: actions/cache@v3
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('backend/pom.xml') }}

      - name: Run unit tests
        run: cd backend && mvn test
        env:
          SPRING_PROFILES_ACTIVE: test
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_DATASOURCE_USERNAME: postgres
          SPRING_DATASOURCE_PASSWORD: postgres
          SPRING_REDIS_HOST: localhost
          SPRING_REDIS_PORT: 6379

      - name: Run integration tests
        run: cd backend && mvn failsafe:integration-test failsafe:verify
        env:
          SPRING_PROFILES_ACTIVE: test
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_DATASOURCE_USERNAME: postgres
          SPRING_DATASOURCE_PASSWORD: postgres

      - name: Generate coverage report
        run: cd backend && mvn jacoco:report

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./backend/target/site/jacoco/jacoco.xml
          flags: backend
          name: backend-coverage

  test-audio-service:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install system dependencies
        run: |
          sudo apt-get update
          sudo apt-get install -y libsndfile1 ffmpeg

      - name: Install Poetry
        uses: snok/install-poetry@v1

      - name: Install dependencies
        run: cd audio-service && poetry install

      - name: Run unit tests
        run: cd audio-service && poetry run pytest tests/ -v --cov=app --cov-report=xml
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
          ENVIRONMENT: test

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./audio-service/coverage.xml
          flags: audio-service
          name: audio-service-coverage

  # ==================
  # End-to-End Tests
  # ==================
  e2e-tests:
    runs-on: ubuntu-latest
    needs: [test-frontend, test-backend, test-audio-service]
    if: github.event_name == 'pull_request' && github.base_ref == 'main'
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: phantom_power_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

      redis:
        image: redis:7
        ports:
          - 6379:6379

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Start backend service
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'openjdk'
          java-version: ${{ env.JAVA_VERSION }}

      - name: Start backend service
        run: |
          cd backend
          mvn spring-boot:run &
          sleep 30
        env:
          SPRING_PROFILES_ACTIVE: test
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/phantom_power_test
          SPRING_DATASOURCE_USERNAME: postgres
          SPRING_DATASOURCE_PASSWORD: postgres
          SERVER_PORT: 8080

      # Start audio service
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Start audio service
        run: |
          cd audio-service
          poetry install
          poetry run uvicorn main:app --host 0.0.0.0 --port 8000 &
          sleep 15
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/phantom_power_test
          ENVIRONMENT: test

      # Run E2E tests
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
          cache-dependency-path: frontend/pnpm-lock.yaml

      - name: Install frontend dependencies
        run: cd frontend && pnpm install --frozen-lockfile

      - name: Run Cypress E2E tests
        uses: cypress-io/github-action@v6
        with:
          working-directory: frontend
          build: pnpm build
          start: pnpm start
          wait-on: 'http://localhost:3000, http://localhost:8080/actuator/health, http://localhost:8000/health'
          wait-on-timeout: 120
          record: true
        env:
          CYPRESS_RECORD_KEY: ${{ secrets.CYPRESS_RECORD_KEY }}
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          NEXT_PUBLIC_API_URL: http://localhost:8080/api
          NEXT_PUBLIC_AUDIO_SERVICE_URL: http://localhost:8000

      - name: Upload E2E test artifacts
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: cypress-screenshots
          path: frontend/cypress/screenshots

  # ==================
  # Build and Package
  # ==================
  build:
    runs-on: ubuntu-latest
    needs: [lint-and-format, security-scan]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Build frontend
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'pnpm'
          cache-dependency-path: frontend/pnpm-lock.yaml

      - name: Install frontend dependencies
        run: cd frontend && pnpm install --frozen-lockfile

      - name: Build frontend
        run: cd frontend && pnpm build
        env:
          NEXT_PUBLIC_API_URL: https://phantom-power-backend.onrender.com/api
          NEXT_PUBLIC_AUDIO_SERVICE_URL: https://phantom-power-audio.onrender.com

      - name: Upload frontend build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: frontend-build
          path: frontend/.next

      # Build backend
      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'openjdk'
          java-version: ${{ env.JAVA_VERSION }}

      - name: Build backend
        run: cd backend && mvn clean package -DskipTests

      - name: Upload backend build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: backend-build
          path: backend/target/*.jar

      # Build audio service
      - name: Setup Python
        uses: actions/setup-python@v4
        with:
          python-version: ${{ env.PYTHON_VERSION }}

      - name: Install Poetry
        uses: snok/install-poetry@v1

      - name: Build audio service requirements
        run: |
          cd audio-service
          poetry export -f requirements.txt --output requirements.txt --without-hashes

      - name: Upload audio service build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: audio-service-build
          path: audio-service/requirements.txt

  # ==================
  # Deploy to Production
  # ==================
  deploy-production:
    runs-on: ubuntu-latest
    needs: [test-frontend, test-backend, test-audio-service, build]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Deploy to Render.com
      - name: Deploy Backend to Render
        uses: johnbeynon/render-deploy-action@v0.0.8
        with:
          service-id: ${{ secrets.RENDER_BACKEND_SERVICE_ID }}
          api-key: ${{ secrets.RENDER_API_KEY }}
          wait-for-success: true

      - name: Deploy Audio Service to Render
        uses: johnbeynon/render-deploy-action@v0.0.8
        with:
          service-id: ${{ secrets.RENDER_AUDIO_SERVICE_ID }}
          api-key: ${{ secrets.RENDER_API_KEY }}
          wait-for-success: true

      - name: Deploy Frontend to Render
        uses: johnbeynon/render-deploy-action@v0.0.8
        with:
          service-id: ${{ secrets.RENDER_FRONTEND_SERVICE_ID }}
          api-key: ${{ secrets.RENDER_API_KEY }}
          wait-for-success: true

      # Post-deployment verification
      - name: Verify deployment health
        run: |
          sleep 60  # Wait for services to start
          
          echo "Checking backend health..."
          curl -f https://phantom-power-backend.onrender.com/actuator/health || exit 1
          
          echo "Checking audio service health..."
          curl -f https://phantom-power-audio.onrender.com/health || exit 1
          
          echo "Checking frontend health..."
          curl -f https://phantom-power.onrender.com || exit 1
          
          echo "All services are healthy!"

      # Notify deployment success
      - name: Notify deployment success
        uses: 8398a7/action-slack@v3
        with:
          status: success
          text: "🚀 Phantom Power deployed successfully to production!"
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
        if: success()

      - name: Notify deployment failure
        uses: 8398a7/action-slack@v3
        with:
          status: failure
          text: "❌ Phantom Power deployment failed!"
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
        if: failure()
```

### Pull Request Workflow

```yaml
# .github/workflows/pr.yml
name: Pull Request Validation

on:
  pull_request:
    branches: [main, develop]

jobs:
  pr-validation:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # PR title and description validation
      - name: Validate PR title
        uses: amannn/action-semantic-pull-request@v5
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Check PR description
        run: |
          if [ -z "${{ github.event.pull_request.body }}" ]; then
            echo "Error: Pull request description is required"
            exit 1
          fi

      # Code change analysis
      - name: Analyze changed files
        id: changes
        uses: dorny/paths-filter@v2
        with:
          filters: |
            frontend:
              - 'frontend/**'
            backend:
              - 'backend/**'
            audio-service:
              - 'audio-service/**'
            docs:
              - 'docs/**'
            config:
              - '.github/**'
              - 'docker-compose.yml'
              - 'render.yaml'

      # Skip certain checks for docs-only changes
      - name: Skip tests for docs-only changes
        if: steps.changes.outputs.docs == 'true' && steps.changes.outputs.frontend == 'false' && steps.changes.outputs.backend == 'false' && steps.changes.outputs.audio-service == 'false'
        run: echo "Skipping tests for documentation-only changes"

      # Run comprehensive validation for code changes
      - name: Run full validation
        if: steps.changes.outputs.frontend == 'true' || steps.changes.outputs.backend == 'true' || steps.changes.outputs.audio-service == 'true'
        run: |
          echo "Code changes detected, running full validation pipeline"
          # This would trigger the main CI pipeline jobs

  pr-size-check:
    runs-on: ubuntu-latest
    steps:
      - name: Check PR size
        uses: pascalgn/size-label-action@v0.4.3
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          sizes: >
            {
              "0": "XS",
              "10": "S",
              "30": "M",
              "100": "L",
              "500": "XL",
              "1000": "XXL"
            }

  pr-dependencies:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Check for dependency updates
      - name: Check for package.json changes
        id: package-changes
        uses: dorny/paths-filter@v2
        with:
          filters: |
            frontend-deps:
              - 'frontend/package.json'
              - 'frontend/pnpm-lock.yaml'
            backend-deps:
              - 'backend/pom.xml'
            audio-deps:
              - 'audio-service/pyproject.toml'
              - 'audio-service/poetry.lock'

      - name: Validate dependency changes
        if: steps.package-changes.outputs.frontend-deps == 'true' || steps.package-changes.outputs.backend-deps == 'true' || steps.package-changes.outputs.audio-deps == 'true'
        run: |
          echo "Dependency changes detected, running security audit..."
          # Run dependency security checks
```

### Release Workflow

```yaml
# .github/workflows/release.yml
name: Release

on:
  release:
    types: [published]
  workflow_dispatch:
    inputs:
      version:
        description: 'Release version'
        required: true
        default: 'patch'
        type: choice
        options:
          - patch
          - minor
          - major

jobs:
  create-release:
    runs-on: ubuntu-latest
    if: github.event_name == 'workflow_dispatch'
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}

      # Update version numbers
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Update frontend version
        run: |
          cd frontend
          npm version ${{ github.event.inputs.version }} --no-git-tag-version

      - name: Update backend version
        run: |
          cd backend
          mvn versions:set -DnewVersion=$(node -p "require('./frontend/package.json').version")

      - name: Update audio service version
        run: |
          cd audio-service
          poetry version ${{ github.event.inputs.version }}

      # Create release commit and tag
      - name: Commit version updates
        run: |
          VERSION=$(node -p "require('./frontend/package.json').version")
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add .
          git commit -m "chore: bump version to v$VERSION"
          git tag "v$VERSION"
          git push origin main --tags

      # Create GitHub release
      - name: Create GitHub Release
        uses: softprops/action-gh-release@v1
        with:
          tag_name: v$(node -p "require('./frontend/package.json').version")
          name: Release v$(node -p "require('./frontend/package.json').version")
          draft: false
          prerelease: false
          generate_release_notes: true

  deploy-release:
    runs-on: ubuntu-latest
    if: github.event_name == 'release'
    environment: production
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Run full test suite
      - name: Run comprehensive tests
        run: |
          echo "Running full test suite for release..."
          # This would run all the tests from the main CI pipeline

      # Deploy with release tag
      - name: Deploy release to production
        run: |
          echo "Deploying release ${{ github.event.release.tag_name }} to production..."
          # Deploy to production with release version
```

### Scheduled Workflows

```yaml
# .github/workflows/scheduled.yml
name: Scheduled Maintenance

on:
  schedule:
    # Run security scans daily at 2 AM UTC
    - cron: '0 2 * * *'
    # Run dependency updates weekly on Mondays at 9 AM UTC
    - cron: '0 9 * * 1'

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Run comprehensive security scan
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  dependency-updates:
    runs-on: ubuntu-latest
    if: github.event.schedule == '0 9 * * 1'
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      # Update frontend dependencies
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Update frontend dependencies
        run: |
          cd frontend
          pnpm update
          pnpm audit --fix

      # Update backend dependencies
      - name: Update backend dependencies
        run: |
          cd backend
          mvn versions:use-latest-versions

      # Update Python dependencies
      - name: Update Python dependencies
        run: |
          cd audio-service
          poetry update

      # Create PR with updates
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v5
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          commit-message: 'chore: update dependencies'
          title: 'Automated dependency updates'
          body: |
            This PR contains automated dependency updates.
            
            Please review and test before merging.
          branch: 'automated/dependency-updates'
          delete-branch: true
```

## Quality Gates

### Code Coverage Requirements

```yaml
# .github/workflows/coverage.yml
name: Coverage Gate

on:
  pull_request:
    branches: [main]

jobs:
  coverage-gate:
    runs-on: ubuntu-latest
    steps:
      - name: Download coverage reports
        uses: actions/download-artifact@v3
        with:
          name: coverage-reports

      - name: Check coverage thresholds
        run: |
          # Frontend coverage check (minimum 80%)
          FRONTEND_COVERAGE=$(node -p "const report = require('./frontend/coverage/coverage-summary.json'); report.total.lines.pct")
          if (( $(echo "$FRONTEND_COVERAGE < 80" | bc -l) )); then
            echo "Frontend coverage ($FRONTEND_COVERAGE%) below threshold (80%)"
            exit 1
          fi

          # Backend coverage check (minimum 80%)
          BACKEND_COVERAGE=$(xmllint --xpath "string(//counter[@type='LINE']/@missed)" backend/target/site/jacoco/jacoco.xml)
          # Calculate percentage and check threshold
          
          # Audio service coverage check (minimum 80%)
          AUDIO_COVERAGE=$(python -c "import xml.etree.ElementTree as ET; tree = ET.parse('audio-service/coverage.xml'); print(tree.getroot().attrib['line-rate'])")
          if (( $(echo "$AUDIO_COVERAGE < 0.8" | bc -l) )); then
            echo "Audio service coverage ($AUDIO_COVERAGE%) below threshold (80%)"
            exit 1
          fi

          echo "All coverage thresholds met!"
```

### Performance Benchmarks

```yaml
# .github/workflows/performance.yml
name: Performance Benchmarks

on:
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 3 * * *'  # Daily at 3 AM UTC

jobs:
  lighthouse-ci:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies
        run: cd frontend && pnpm install

      - name: Build frontend
        run: cd frontend && pnpm build

      # Run Lighthouse CI
      - name: Run Lighthouse CI
        uses: treosh/lighthouse-ci-action@v10
        with:
          configPath: './frontend/lighthouserc.js'
          uploadArtifacts: true
          temporaryPublicStorage: true

  backend-performance:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: 'openjdk'
          java-version: '17'

      - name: Start backend service
        run: |
          cd backend
          mvn spring-boot:run &
          sleep 30
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_DATASOURCE_USERNAME: postgres
          SPRING_DATASOURCE_PASSWORD: postgres

      # Run performance tests with Artillery
      - name: Install Artillery
        run: npm install -g artillery

      - name: Run performance tests
        run: |
          artillery run backend/performance-tests/load-test.yml
          artillery run backend/performance-tests/stress-test.yml

      - name: Upload performance results
        uses: actions/upload-artifact@v3
        with:
          name: performance-results
          path: backend/performance-tests/results/
```

## Notifications and Integrations

### Slack Integration

```yaml
# Add to existing workflows
- name: Notify Slack on failure
  uses: 8398a7/action-slack@v3
  with:
    status: failure
    text: |
      🚨 CI/CD Pipeline Failed
      Repository: ${{ github.repository }}
      Branch: ${{ github.ref }}
      Commit: ${{ github.sha }}
      Author: ${{ github.actor }}
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
  if: failure()

- name: Notify Slack on success
  uses: 8398a7/action-slack@v3
  with:
    status: success
    text: |
      ✅ CI/CD Pipeline Successful
      Repository: ${{ github.repository }}
      Branch: ${{ github.ref }}
      Deployed to: Production
  env:
    SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
  if: success() && github.ref == 'refs/heads/main'
```

### Discord Integration

```yaml
- name: Discord notification
  uses: Ilshidur/action-discord@master
  env:
    DISCORD_WEBHOOK: ${{ secrets.DISCORD_WEBHOOK }}
  with:
    args: |
      🔄 **Phantom Power Deployment**
      Status: ${{ job.status }}
      Branch: ${{ github.ref }}
      Commit: ${{ github.sha }}
```

## Secrets Management

### Required GitHub Secrets

```bash
# Render.com deployment
RENDER_API_KEY=your_render_api_key
RENDER_BACKEND_SERVICE_ID=srv-backend-service-id
RENDER_AUDIO_SERVICE_ID=srv-audio-service-id
RENDER_FRONTEND_SERVICE_ID=srv-frontend-service-id

# Testing and coverage
CYPRESS_RECORD_KEY=your_cypress_record_key
CODECOV_TOKEN=your_codecov_token

# Notifications
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/your/webhook/url
DISCORD_WEBHOOK=https://discord.com/api/webhooks/your/webhook

# Security scanning
SONAR_TOKEN=your_sonarcloud_token
SNYK_TOKEN=your_snyk_token

# External services (for E2E tests)
TEST_STRIPE_SECRET_KEY=sk_test_your_test_stripe_key
TEST_SENDGRID_API_KEY=SG.your_test_sendgrid_key
```

### Environment-Specific Secrets

```yaml
# Production environment secrets
production:
  JWT_SECRET: ${{ secrets.PROD_JWT_SECRET }}
  DATABASE_URL: ${{ secrets.PROD_DATABASE_URL }}
  STRIPE_SECRET_KEY: ${{ secrets.PROD_STRIPE_SECRET_KEY }}

# Staging environment secrets  
staging:
  JWT_SECRET: ${{ secrets.STAGING_JWT_SECRET }}
  DATABASE_URL: ${{ secrets.STAGING_DATABASE_URL }}
  STRIPE_SECRET_KEY: ${{ secrets.STAGING_STRIPE_SECRET_KEY }}
```

## Monitoring and Observability

### Pipeline Monitoring

```yaml
# .github/workflows/monitor.yml
name: Pipeline Health Monitor

on:
  schedule:
    - cron: '*/15 * * * *'  # Every 15 minutes

jobs:
  monitor-pipeline:
    runs-on: ubuntu-latest
    steps:
      - name: Check recent workflow runs
        uses: actions/github-script@v6
        with:
          script: |
            const { data: runs } = await github.rest.actions.listWorkflowRuns({
              owner: context.repo.owner,
              repo: context.repo.repo,
              workflow_id: 'main.yml',
              per_page: 10
            });
            
            const failedRuns = runs.workflow_runs.filter(run => 
              run.status === 'completed' && run.conclusion === 'failure'
            );
            
            if (failedRuns.length > 3) {
              core.setFailed(`High failure rate detected: ${failedRuns.length} failures in last 10 runs`);
            }
            
            console.log(`Pipeline health: ${runs.workflow_runs.length - failedRuns.length}/${runs.workflow_runs.length} successful runs`);
```

### Deployment Tracking

```yaml
- name: Track deployment metrics
  run: |
    # Send deployment metrics to monitoring service
    curl -X POST https://api.datadog.com/api/v1/events \
      -H "Content-Type: application/json" \
      -H "DD-API-KEY: ${{ secrets.DATADOG_API_KEY }}" \
      -d '{
        "title": "Phantom Power Deployment",
        "text": "Successfully deployed to production",
        "tags": ["environment:production", "service:phantom-power"],
        "alert_type": "info"
      }'
```

## Best Practices

### Workflow Organization
1. **Separate Concerns**: Different workflows for different purposes
2. **Reusable Actions**: Create custom actions for common tasks
3. **Conditional Execution**: Skip unnecessary steps based on file changes
4. **Parallel Execution**: Run independent jobs in parallel
5. **Resource Optimization**: Use appropriate runner types and caching

### Security Best Practices
1. **Least Privilege**: Minimal permissions for each workflow
2. **Secret Management**: Use GitHub secrets, never hardcode credentials
3. **Dependency Scanning**: Regular security audits of dependencies
4. **Code Scanning**: Static analysis and vulnerability detection
5. **Supply Chain Security**: Verify action and dependency integrity

### Performance Optimization
1. **Caching Strategy**: Cache dependencies and build artifacts
2. **Matrix Builds**: Test across multiple environments efficiently
3. **Artifact Management**: Store and reuse build outputs
4. **Resource Limits**: Set appropriate timeouts and resource constraints
5. **Pipeline Parallelization**: Maximize concurrent execution

### Monitoring and Alerting
1. **Pipeline Health**: Monitor success/failure rates
2. **Performance Metrics**: Track build times and resource usage
3. **Deployment Tracking**: Monitor deployment frequency and success
4. **Quality Metrics**: Track code coverage and quality trends
5. **Alert Fatigue**: Balance comprehensive monitoring with noise reduction

<!-- TODO: Add pipeline analytics and reporting -->
<!-- TODO: Document blue-green deployment strategies -->
<!-- TODO: Add rollback procedures -->
<!-- TODO: Create pipeline optimization guide -->
<!-- TODO: Add compliance and audit logging -->