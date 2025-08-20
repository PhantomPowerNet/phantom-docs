# Git Workflow

## Overview

Phantom Power uses a structured Git workflow that emphasizes code quality, security, and controlled deployments. Our workflow is based on GitFlow principles with modifications for microservices architecture and deployment/a branch strategy.

## Branch Structure

### Main Branches

#### `main`
- **Purpose**: Production-ready code
- **Protection**: Fully protected, no direct pushes
- **Merges**: Only from `deployment/a` branches via approved PRs
- **CI/CD**: Triggers production deployment pipeline
- **Naming**: Always `main` (default branch)

#### `develop`
- **Purpose**: Integration branch for development
- **Protection**: Protected, requires PR approval
- **Merges**: From feature branches and hotfixes
- **CI/CD**: Triggers staging environment deployment
- **Naming**: Always `develop`

### Deployment Branches

Following the user's specification, we use deployment branches for each application:

#### `deployment/frontend`
- **Purpose**: Frontend application deployment preparation
- **Source**: Merges from `develop` when frontend changes are ready
- **Target**: Merges to `main` for production deployment
- **Lifespan**: Long-lived, updated per release cycle

#### `deployment/backend`
- **Purpose**: Backend API deployment preparation
- **Source**: Merges from `develop` when backend changes are ready
- **Target**: Merges to `main` for production deployment
- **Lifespan**: Long-lived, updated per release cycle

#### `deployment/audio-service`
- **Purpose**: Audio processing service deployment preparation
- **Source**: Merges from `develop` when audio service changes are ready
- **Target**: Merges to `main` for production deployment
- **Lifespan**: Long-lived, updated per release cycle

### Working Branches

Using slash hierarchy naming convention as specified:

#### Feature Branches
- **Pattern**: `feature/description-of-feature`
- **Examples**:
  - `feature/axios-endpoints`
  - `feature/user-authentication`
  - `feature/audio-upload-component`
  - `feature/project-matching-algorithm`
- **Source**: Branched from `develop`
- **Target**: Merged back to `develop`
- **Lifespan**: Deleted after merge

#### Bug Fix Branches
- **Pattern**: `fix/description-of-bug-fix`
- **Examples**:
  - `fix/homepage-render-issue`
  - `fix/audio-analysis-timeout`
  - `fix/user-profile-validation`
  - `fix/database-connection-leak`
- **Source**: Branched from `develop` or `main` (for hotfixes)
- **Target**: Merged to `develop` or directly to `main` (hotfixes)
- **Lifespan**: Deleted after merge

#### Configuration Branches
- **Pattern**: `config/description-of-configuration`
- **Examples**:
  - `config/websockets`
  - `config/docker-optimization`
  - `config/ci-cd-pipeline`
  - `config/database-indexes`
- **Source**: Branched from `develop`
- **Target**: Merged back to `develop`
- **Lifespan**: Deleted after merge

#### Design Branches
- **Pattern**: `design/description-of-design-change`
- **Examples**:
  - `design/navigation-layout`
  - `design/user-dashboard-redesign`
  - `design/mobile-responsive-updates`
  - `design/audio-player-interface`
- **Source**: Branched from `develop`
- **Target**: Merged back to `develop`
- **Lifespan**: Deleted after merge

#### Hotfix Branches
- **Pattern**: `hotfix/critical-issue-description`
- **Examples**:
  - `hotfix/security-vulnerability-patch`
  - `hotfix/payment-processing-error`
  - `hotfix/data-corruption-fix`
- **Source**: Branched from `main`
- **Target**: Merged to both `main` and `develop`
- **Lifespan**: Deleted after merge

## Workflow Process

### 1. Feature Development Workflow

```bash
# 1. Start from updated develop branch
git checkout develop
git pull origin develop

# 2. Create feature branch with slash hierarchy naming
git checkout -b feature/axios-endpoints

# 3. Work on feature with frequent commits
git add .
git commit -m "feat(api): add axios interceptor for authentication

Implement request/response interceptors to handle JWT tokens
automatically. This reduces boilerplate code across API calls
and centralizes error handling.

Closes #123"

# 4. Push branch and keep it updated
git push -u origin feature/axios-endpoints

# 5. Regularly sync with develop
git fetch origin
git rebase origin/develop  # or merge if preferred

# 6. Create Pull Request when ready
# - Target: develop
# - Include comprehensive description
# - Link related issues
# - Request appropriate reviewers
```

### 2. Pull Request Process

#### Creating a Pull Request
```markdown
## Pull Request Template

### Description
Brief description of changes and why they were made.

### Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to not work as expected)
- [ ] Documentation update
- [ ] Configuration change
- [ ] Design/UI update

### Related Issues
Closes #123
Related to #456

### Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] Accessibility testing (if UI changes)
- [ ] Performance impact assessed

### Screenshots (if applicable)
[Include before/after screenshots for UI changes]

### Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Documentation updated
- [ ] Tests added/updated
- [ ] No security vulnerabilities introduced
```

#### Review Process
1. **Automated Checks**: All CI/CD pipelines must pass
2. **Code Review**: At least one team member approval required
3. **Security Review**: Required for sensitive changes
4. **Testing**: Verify test coverage meets requirements (80%)
5. **Documentation**: Ensure documentation is updated

### 3. Deployment Workflow

#### Preparing for Deployment
```bash
# 1. When features are ready for deployment, merge to appropriate deployment branch
git checkout deployment/frontend
git pull origin deployment/frontend
git merge develop --no-ff

# 2. Test deployment branch thoroughly
# Run full test suite, integration tests, and manual QA

# 3. Create deployment PR to main
git push origin deployment/frontend
# Create PR: deployment/frontend -> main
```

#### Production Deployment
```bash
# 1. After deployment PR approval, merge to main
# This triggers production deployment via CI/CD

# 2. Tag the release
git checkout main
git pull origin main
git tag -a v1.2.0 -m "Release version 1.2.0

Features:
- Audio analysis improvements
- User profile enhancements
- Performance optimizations

Bug fixes:
- Homepage rendering issue
- Authentication timeout fix"
git push origin v1.2.0

# 3. Update deployment branch
git checkout deployment/frontend
git merge main --ff-only
git push origin deployment/frontend
```

### 4. Hotfix Workflow

```bash
# 1. Create hotfix branch from main
git checkout main
git pull origin main
git checkout -b hotfix/security-vulnerability-patch

# 2. Implement critical fix
git add .
git commit -m "fix(security): patch XSS vulnerability in user input

Implement proper input sanitization to prevent XSS attacks
in user-generated content. This addresses the security
vulnerability reported in issue #789.

Security Impact: High
Tested: Manual testing and security scan passed

Closes #789"

# 3. Create PR to main for immediate deployment
git push -u origin hotfix/security-vulnerability-patch
# Create PR: hotfix/security-vulnerability-patch -> main

# 4. After merge to main, also merge to develop
git checkout develop
git pull origin develop
git merge main --no-ff
git push origin develop

# 5. Clean up hotfix branch
git branch -d hotfix/security-vulnerability-patch
git push origin --delete hotfix/security-vulnerability-patch
```

## Commit Standards

### Commit Message Format

Following Conventional Commits specification:

```
type(scope): brief description

Longer description explaining the change in detail.
Include reasoning for the change and contrast with
previous behavior if applicable.

Breaking Change: Description of breaking change (if applicable)
Closes #123
Related #456
```

### Commit Types
- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes (formatting, missing semicolons, etc.)
- **refactor**: Code changes that neither fix bugs nor add features
- **perf**: Performance improvements
- **test**: Adding or modifying tests
- **chore**: Changes to build process, dependencies, etc.
- **ci**: Changes to CI/CD configuration
- **security**: Security-related changes

### Scope Examples
- **api**: Backend API changes
- **ui**: Frontend UI components
- **auth**: Authentication/authorization
- **audio**: Audio processing functionality
- **db**: Database-related changes
- **config**: Configuration changes
- **deps**: Dependency updates

### Commit Message Examples

```bash
# Good commit messages
git commit -m "feat(auth): add JWT refresh token rotation

Implement automatic refresh token rotation for enhanced security.
Tokens are rotated on each refresh request and old tokens are
invalidated after 24 hours.

Breaking Change: Updates authentication flow, clients must handle
token rotation properly.

Closes #234"

git commit -m "fix(audio): resolve memory leak in analysis service

Fix memory leak caused by temporary files not being cleaned up
after audio analysis completion. Add proper cleanup in finally
block and implement monitoring for disk usage.

Closes #456"

git commit -m "docs(api): update user endpoints documentation

Add comprehensive examples for user profile management endpoints.
Include request/response examples and error handling scenarios.

Related #789"
```

## Branch Protection Rules

### Main Branch Protection
```yaml
# GitHub branch protection settings for main
protection_rules:
  main:
    required_status_checks:
      strict: true
      contexts:
        - "ci/frontend-tests"
        - "ci/backend-tests"
        - "ci/audio-service-tests"
        - "ci/security-scan"
        - "ci/code-quality"
    enforce_admins: true
    required_pull_request_reviews:
      required_approving_review_count: 2
      dismiss_stale_reviews: true
      require_code_owner_reviews: true
      restrict_pushes: true
    restrictions:
      users: []
      teams: ["core-team"]
    allow_force_pushes: false
    allow_deletions: false
```

### Develop Branch Protection
```yaml
# GitHub branch protection settings for develop
protection_rules:
  develop:
    required_status_checks:
      strict: true
      contexts:
        - "ci/frontend-tests"
        - "ci/backend-tests"
        - "ci/audio-service-tests"
        - "ci/lint-and-format"
    enforce_admins: false
    required_pull_request_reviews:
      required_approving_review_count: 1
      dismiss_stale_reviews: true
    allow_force_pushes: false
    allow_deletions: false
```

### Deployment Branch Protection
```yaml
# GitHub branch protection settings for deployment branches
protection_rules:
  "deployment/*":
    required_status_checks:
      strict: true
      contexts:
        - "ci/full-test-suite"
        - "ci/integration-tests"
        - "ci/security-scan"
        - "ci/performance-tests"
    enforce_admins: true
    required_pull_request_reviews:
      required_approving_review_count: 2
      require_code_owner_reviews: true
    restrictions:
      teams: ["deploy-team", "core-team"]
    allow_force_pushes: false
    allow_deletions: false
```

## Code Owners

### CODEOWNERS File
```bash
# .github/CODEOWNERS
# Global owners
* @phantom-power/core-team

# Frontend code
/frontend/ @phantom-power/frontend-team @phantom-power/core-team
/frontend/src/components/ui/ @phantom-power/design-team

# Backend code
/backend/ @phantom-power/backend-team @phantom-power/core-team
/backend/src/main/java/com/phantompower/security/ @phantom-power/security-team

# Audio service
/audio-service/ @phantom-power/audio-team @phantom-power/core-team

# Infrastructure and deployment
/.github/workflows/ @phantom-power/devops-team @phantom-power/core-team
/docker-compose.yml @phantom-power/devops-team
/render.yaml @phantom-power/devops-team

# Documentation
/docs/ @phantom-power/core-team
/README.md @phantom-power/core-team

# Database migrations
/backend/src/main/resources/db/migration/ @phantom-power/backend-team @phantom-power/dba-team

# Security-sensitive files
/backend/src/main/java/com/phantompower/config/SecurityConfig.java @phantom-power/security-team
/frontend/src/lib/auth/ @phantom-power/security-team
```

## Release Management

### Release Planning
1. **Sprint Planning**: Define features for upcoming release
2. **Feature Freeze**: No new features after deployment branch creation
3. **Testing Phase**: Comprehensive testing on deployment branches
4. **Release Preparation**: Documentation updates, changelog generation
5. **Production Deployment**: Merge deployment branches to main
6. **Post-Release**: Monitor, hotfixes if needed, retrospective

### Versioning Strategy

Following Semantic Versioning (SemVer):

- **Major Version (X.0.0)**: Breaking changes
- **Minor Version (0.X.0)**: New features, backward compatible
- **Patch Version (0.0.X)**: Bug fixes, backward compatible

### Release Tags
```bash
# Version tags
v1.0.0    # Major release
v1.1.0    # Minor release with new features
v1.1.1    # Patch release with bug fixes

# Pre-release tags
v1.2.0-rc.1    # Release candidate
v1.2.0-beta.1  # Beta release
v1.2.0-alpha.1 # Alpha release
```

### Changelog Generation
```bash
# Generate changelog from commit messages
git log --oneline --grep="feat" --grep="fix" --grep="BREAKING" \
  v1.0.0..HEAD --pretty=format:"%s" > CHANGELOG.md

# Or use automated tools
npx conventional-changelog-cli -p angular -i CHANGELOG.md -s
```

## Git Hooks

### Pre-commit Hook
```bash
#!/bin/sh
# .git/hooks/pre-commit

echo "Running pre-commit checks..."

# Run linting
echo "Checking code style..."
cd frontend && pnpm lint:check
if [ $? -ne 0 ]; then
  echo "Frontend linting failed. Please fix issues before committing."
  exit 1
fi

cd ../backend && mvn checkstyle:check
if [ $? -ne 0 ]; then
  echo "Backend checkstyle failed. Please fix issues before committing."
  exit 1
fi

cd ../audio-service && poetry run flake8 .
if [ $? -ne 0 ]; then
  echo "Audio service linting failed. Please fix issues before committing."
  exit 1
fi

# Check for secrets
echo "Scanning for secrets..."
gitleaks protect --verbose --redact --staged
if [ $? -ne 0 ]; then
  echo "Potential secrets detected. Please review and fix."
  exit 1
fi

# Run tests on changed files
echo "Running tests on changed files..."
# Add test execution logic here

echo "Pre-commit checks passed!"
exit 0
```

### Commit Message Hook
```bash
#!/bin/sh
# .git/hooks/commit-msg

commit_regex='^(feat|fix|docs|style|refactor|perf|test|chore|ci|security)(\(.+\))?: .{1,50}'

if ! grep -qE "$commit_regex" "$1"; then
    echo "Invalid commit message format!"
    echo "Format: type(scope): description"
    echo "Example: feat(auth): add JWT refresh token rotation"
    exit 1
fi

# Check for proper line length
if [ $(head -n1 "$1" | wc -c) -gt 72 ]; then
    echo "Commit message first line too long (max 72 characters)"
    exit 1
fi

exit 0
```

## Troubleshooting

### Common Git Issues

#### Merge Conflicts
```bash
# When merge conflicts occur
git status  # See conflicted files

# Resolve conflicts manually, then
git add <resolved-files>
git commit -m "resolve: merge conflicts from develop"

# Or abort merge
git merge --abort
```

#### Revert Changes
```bash
# Revert last commit
git revert HEAD

# Revert specific commit
git revert <commit-hash>

# Create revert PR
git push origin feature/revert-problematic-change
```

#### Clean Up Local Branches
```bash
# Delete merged local branches
git branch --merged | grep -v "\*\|main\|develop" | xargs -n 1 git branch -d

# Clean up remote tracking branches
git remote prune origin

# Force delete unmerged branch (use with caution)
git branch -D feature/abandoned-feature
```

### Branch Synchronization
```bash
# Sync fork with upstream
git remote add upstream https://github.com/phantom-power/phantom-power.git
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# Update all deployment branches
for branch in deployment/frontend deployment/backend deployment/audio-service; do
  git checkout $branch
  git merge main --ff-only
  git push origin $branch
done
```

## Best Practices

### Branch Management
1. **Keep branches focused**: One feature/fix per branch
2. **Use descriptive names**: Follow slash hierarchy naming convention
3. **Regular updates**: Sync with develop/main frequently
4. **Clean up**: Delete merged branches promptly
5. **Small PRs**: Easier to review and merge

### Commit Practices
1. **Atomic commits**: Each commit should be a complete, logical change
2. **Clear messages**: Explain what and why, not just what
3. **Regular commits**: Don't let changes accumulate
4. **Test before commit**: Ensure code works before committing
5. **Sign commits**: Use GPG signing for security

### Collaboration
1. **Communicate**: Discuss significant changes with team
2. **Review thoroughly**: Provide constructive feedback
3. **Test others' code**: Verify changes work as expected
4. **Document decisions**: Record important architectural choices
5. **Share knowledge**: Help team members understand changes

### Security
1. **Never commit secrets**: Use environment variables
2. **Review dependencies**: Check for vulnerabilities
3. **Sign commits**: Verify commit authenticity
4. **Audit changes**: Review security-sensitive modifications
5. **Monitor access**: Regularly review repository permissions

## Automation Scripts

### Branch Creation Script
```bash
#!/bin/bash
# scripts/create-branch.sh

set -e

if [ $# -ne 2 ]; then
  echo "Usage: $0 <type> <description>"
  echo "Types: feature, fix, config, design, hotfix"
  echo "Example: $0 feature axios-endpoints"
  exit 1
fi

TYPE=$1
DESCRIPTION=$2
BRANCH_NAME="$TYPE/$DESCRIPTION"

# Validate branch type
case $TYPE in
  feature|fix|config|design)
    BASE_BRANCH="develop"
    ;;
  hotfix)
    BASE_BRANCH="main"
    ;;
  *)
    echo "Invalid branch type: $TYPE"
    exit 1
    ;;
esac

echo "Creating branch: $BRANCH_NAME from $BASE_BRANCH"

# Switch to base branch and pull latest
git checkout $BASE_BRANCH
git pull origin $BASE_BRANCH

# Create and switch to new branch
git checkout -b $BRANCH_NAME

echo "Branch created successfully: $BRANCH_NAME"
echo "Don't forget to push when you're ready: git push -u origin $BRANCH_NAME"
```

### Deployment Script
```bash
#!/bin/bash
# scripts/deploy-service.sh

set -e

if [ $# -ne 1 ]; then
  echo "Usage: $0 <service>"
  echo "Services: frontend, backend, audio-service"
  exit 1
fi

SERVICE=$1
DEPLOYMENT_BRANCH="deployment/$SERVICE"

echo "Preparing deployment for $SERVICE"

# Switch to deployment branch
git checkout $DEPLOYMENT_BRANCH
git pull origin $DEPLOYMENT_BRANCH

# Merge from develop
echo "Merging latest changes from develop..."
git merge develop --no-ff -m "prepare: merge develop for $SERVICE deployment"

# Run service-specific tests
echo "Running tests for $SERVICE..."
case $SERVICE in
  frontend)
    cd frontend && pnpm test:ci && pnpm build
    ;;
  backend)
    cd backend && mvn test && mvn package -DskipTests
    ;;
  audio-service)
    cd audio-service && poetry run pytest && poetry build
    ;;
esac

echo "Tests passed. Ready to push deployment branch."
echo "Run: git push origin $DEPLOYMENT_BRANCH"
echo "Then create PR: $DEPLOYMENT_BRANCH -> main"
```

<!-- TODO: Add Git workflow visualization diagrams -->
<!-- TODO: Document emergency rollback procedures -->
<!-- TODO: Add branch metrics and monitoring setup -->
<!-- TODO: Create automated branch cleanup workflows -->