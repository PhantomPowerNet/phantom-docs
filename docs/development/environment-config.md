# Environment Configuration Guide

## Overview

This guide covers Mac-specific environment configuration for Phantom Power development, including IDE setup, secrets management through zsh, and development tools configuration.

## Prerequisites

### Required Tools
- **macOS 12.0+** (Monterey or later)
- **Homebrew** - Package manager for macOS
- **Zsh** (default shell in macOS Catalina+)
- **IntelliJ IDEA Ultimate** - Primary IDE for backend development
- **VS Code/Cursor** - Frontend and general development

## IDE Setup

### IntelliJ IDEA Ultimate Configuration

#### Installation and Setup
```bash
# Install IntelliJ IDEA Ultimate via Homebrew
brew install --cask intellij-idea

# Or download from JetBrains website
# https://www.jetbrains.com/idea/download/?section=mac
```

#### Project Configuration
1. **Import Project**
   - Open IntelliJ IDEA
   - Select "Open or Import"
   - Navigate to `phantom-power/backend` directory
   - Select `pom.xml` and choose "Open as Project"

2. **Java SDK Configuration**
   ```bash
   # Ensure Java 17+ is installed
   brew install openjdk@17
   
   # Link Java for system access
   sudo ln -sfn /opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk \
     /Library/Java/JavaVirtualMachines/openjdk-17.jdk
   ```

3. **IntelliJ Project Settings**
   - **File > Project Structure**
   - **Project SDK**: Select Java 17
   - **Project Language Level**: 17
   - **Modules**: Ensure backend module is configured

#### Essential Plugins
```bash
# Install these IntelliJ plugins:
# - Spring Boot
# - Spring MVC
# - Database Tools and SQL
# - Docker
# - GitToolBox
# - SonarLint
# - CheckStyle-IDEA
# - SpotBugs
```

#### Spring Boot Configuration
1. **Run Configuration**
   - **Run > Edit Configurations**
   - **Add New > Spring Boot**
   - **Main Class**: `com.phantompower.PhantomPowerApplication`
   - **Program Arguments**: `--spring.profiles.active=dev`
   - **Environment Variables**: Load from `.env` file

2. **Database Connection**
   - **Database Tool Window** (View > Tool Windows > Database)
   - **Add PostgreSQL Data Source**
   - **Host**: `localhost`, **Port**: `5432`
   - **Database**: `phantom_power_dev`
   - **User**: `phantom_user`

### VS Code/Cursor Configuration

#### Installation
```bash
# Install VS Code
brew install --cask visual-studio-code

# Or install Cursor (AI-powered VS Code alternative)
brew install --cask cursor
```

#### Essential Extensions
```json
{
  "recommendations": [
    "ms-vscode.vscode-typescript-next",
    "bradlc.vscode-tailwindcss",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "ms-vscode.vscode-json",
    "ms-python.python",
    "ms-python.flake8",
    "ms-python.black-formatter",
    "ms-vscode.vscode-docker",
    "ms-vscode.vscode-git-graph",
    "github.copilot",
    "github.copilot-chat"
  ]
}
```

#### Workspace Settings
```json
{
  "editor.formatOnSave": true,
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "typescript.preferences.importModuleSpecifier": "relative",
  "tailwindCSS.includeLanguages": {
    "typescript": "javascript",
    "typescriptreact": "javascript"
  },
  "python.formatting.provider": "black",
  "python.linting.enabled": true,
  "python.linting.flake8Enabled": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[python]": {
    "editor.defaultFormatter": "ms-python.black-formatter"
  }
}
```

## Zsh Configuration for Secrets Management

### Zsh Profile Setup

#### Configure ~/.zshrc
```bash
# Open zsh configuration
code ~/.zshrc

# Add Phantom Power environment section
cat >> ~/.zshrc << 'EOF'

# =================================
# Phantom Power Development Config
# =================================

# Node.js and pnpm
export PATH="/opt/homebrew/bin:$PATH"
export PNPM_HOME="$HOME/.local/share/pnpm"
export PATH="$PNPM_HOME:$PATH"

# Java Development
export JAVA_HOME="/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home"
export PATH="$JAVA_HOME/bin:$PATH"

# Python Development
export PATH="/opt/homebrew/opt/python@3.11/bin:$PATH"

# Docker
export DOCKER_BUILDKIT=1
export COMPOSE_DOCKER_CLI_BUILD=1

# Load Phantom Power secrets
if [[ -f "$HOME/.phantom_secrets" ]]; then
  source "$HOME/.phantom_secrets"
fi

# Development aliases
alias pp-start="cd ~/Developer/phantom-power && docker-compose up -d"
alias pp-stop="cd ~/Developer/phantom-power && docker-compose down"
alias pp-logs="cd ~/Developer/phantom-power && docker-compose logs -f"
alias pp-backend="cd ~/Developer/phantom-power/backend && mvn spring-boot:run"
alias pp-frontend="cd ~/Developer/phantom-power/frontend && pnpm dev"
alias pp-audio="cd ~/Developer/phantom-power/audio-service && source venv/bin/activate && uvicorn main:app --reload"

EOF
```

### Secrets Management

#### Create ~/.phantom_secrets
```bash
# Create secrets file with proper permissions
touch ~/.phantom_secrets
chmod 600 ~/.phantom_secrets

# Add environment variables
cat >> ~/.phantom_secrets << 'EOF'
# =================================
# Phantom Power Secrets
# =================================

# Database Configuration
export DATABASE_URL="postgresql://phantom_user:phantom_pass@localhost:5432/phantom_power_dev"
export SPRING_DATASOURCE_URL="jdbc:postgresql://localhost:5432/phantom_power_dev"
export SPRING_DATASOURCE_USERNAME="phantom_user"
export SPRING_DATASOURCE_PASSWORD="phantom_pass"

# JWT Configuration
export JWT_SECRET="your_very_long_jwt_secret_key_at_least_256_bits_for_production_use"
export JWT_EXPIRATION="86400000"

# API Keys and External Services
export OPENAI_API_KEY="sk-your-openai-api-key-here"
export STRIPE_SECRET_KEY="sk_test_your_stripe_secret_key_here"
export STRIPE_PUBLISHABLE_KEY="pk_test_your_stripe_publishable_key_here"
export SENDGRID_API_KEY="SG.your_sendgrid_api_key_here"

# Audio Processing
export AUDIO_SERVICE_URL="http://localhost:8000"
export LIBROSA_CACHE_DIR="$HOME/.cache/librosa"

# File Storage (Development)
export FILE_STORAGE_TYPE="local"
export FILE_STORAGE_PATH="/tmp/phantom-uploads"
export FILE_MAX_SIZE="52428800"

# Redis Configuration
export REDIS_URL="redis://localhost:6379"

# Email Configuration
export MAIL_HOST="smtp.gmail.com"
export MAIL_PORT="587"
export MAIL_USERNAME="your-dev-email@gmail.com"
export MAIL_PASSWORD="your-app-specific-password"

# Development Configuration
export SPRING_PROFILES_ACTIVE="dev"
export NODE_ENV="development"
export NEXT_PUBLIC_API_URL="http://localhost:8080/api"
export NEXT_PUBLIC_WS_URL="ws://localhost:8080/ws"
export NEXT_PUBLIC_AUDIO_SERVICE_URL="http://localhost:8000"

# Logging and Debug
export LOG_LEVEL="DEBUG"
export SPRING_JPA_SHOW_SQL="true"
export SPRING_JPA_PROPERTIES_HIBERNATE_FORMAT_SQL="true"

EOF
```

#### Apply Configuration
```bash
# Reload zsh configuration
source ~/.zshrc

# Verify environment variables are loaded
echo $JWT_SECRET
echo $DATABASE_URL
```

### Environment-Specific Configurations

#### Development Environment
```bash
# ~/.phantom_secrets_dev
export SPRING_PROFILES_ACTIVE="dev"
export LOG_LEVEL="DEBUG"
export SPRING_JPA_SHOW_SQL="true"
export NEXT_PUBLIC_DEV_MODE="true"
```

#### Testing Environment
```bash
# ~/.phantom_secrets_test
export SPRING_PROFILES_ACTIVE="test"
export DATABASE_URL="postgresql://phantom_user:phantom_pass@localhost:5432/phantom_power_test"
export LOG_LEVEL="WARN"
export SPRING_JPA_SHOW_SQL="false"
```

#### Production Environment
```bash
# ~/.phantom_secrets_prod (for local production testing)
export SPRING_PROFILES_ACTIVE="prod"
export LOG_LEVEL="INFO"
export DATABASE_URL="postgresql://user:pass@prod-db:5432/phantom_power"
export JWT_SECRET="production_jwt_secret_min_256_bits"
```

## Development Tools Configuration

### Homebrew Package Management

#### Essential Packages
```bash
# Install essential development tools
brew install git
brew install node@18
brew install pnpm
brew install openjdk@17
brew install maven
brew install python@3.11
brew install poetry
brew install postgresql@15
brew install redis
brew install docker
brew install docker-compose

# Audio processing dependencies
brew install libsndfile
brew install ffmpeg
brew install sox

# Development utilities
brew install curl
brew install wget
brew install jq
brew install tree
brew install htop
brew install ripgrep
brew install fzf
```

#### Cask Applications
```bash
# Install GUI applications
brew install --cask intellij-idea
brew install --cask visual-studio-code
brew install --cask cursor
brew install --cask docker
brew install --cask postman
brew install --cask tableplus
brew install --cask figma
brew install --cask slack
brew install --cask discord
```

### Git Configuration

#### Global Git Settings
```bash
# Configure Git with your information
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Configure merge behavior
git config --global pull.rebase false
git config --global merge.tool vscode

# Helpful aliases
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --decorate --all"
```

#### SSH Key Setup
```bash
# Generate SSH key for GitHub
ssh-keygen -t ed25519 -C "your.email@example.com"

# Add SSH key to ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copy public key to clipboard
pbcopy < ~/.ssh/id_ed25519.pub

# Add to GitHub: Settings > SSH and GPG keys > New SSH key
```

### Database Tools

#### PostgreSQL Configuration
```bash
# Install and start PostgreSQL
brew install postgresql@15
brew services start postgresql@15

# Create development database
createdb phantom_power_dev
createdb phantom_power_test

# Create user and grant permissions
psql postgres -c "CREATE USER phantom_user WITH PASSWORD 'phantom_pass';"
psql postgres -c "GRANT ALL PRIVILEGES ON DATABASE phantom_power_dev TO phantom_user;"
psql postgres -c "GRANT ALL PRIVILEGES ON DATABASE phantom_power_test TO phantom_user;"
```

#### Redis Configuration
```bash
# Install and start Redis
brew install redis
brew services start redis

# Test Redis connection
redis-cli ping
```

## Service-Specific Configuration

### Frontend (Next.js) Configuration

#### Package Manager Setup
```bash
# Install pnpm globally
npm install -g pnpm

# Configure pnpm
pnpm config set registry https://registry.npmjs.org/
pnpm config set store-dir ~/.pnpm-store
```

#### Environment Files
```bash
# Create frontend environment file
cat > ~/Developer/phantom-power/frontend/.env.local << 'EOF'
# API Configuration
NEXT_PUBLIC_API_URL=http://localhost:8080/api
NEXT_PUBLIC_WS_URL=ws://localhost:8080/ws
NEXT_PUBLIC_AUDIO_SERVICE_URL=http://localhost:8000

# Authentication
NEXT_PUBLIC_JWT_SECRET_KEY=your_jwt_secret_key_here

# File Upload
NEXT_PUBLIC_MAX_FILE_SIZE=50000000
NEXT_PUBLIC_UPLOAD_URL=http://localhost:8080/api/upload

# Development
NEXT_PUBLIC_DEV_MODE=true
NEXT_PUBLIC_LOG_LEVEL=debug

# External Services
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_your_stripe_key_here
EOF
```

### Backend (Spring Boot) Configuration

#### Maven Configuration
```bash
# Configure Maven settings
mkdir -p ~/.m2
cat > ~/.m2/settings.xml << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<settings>
  <profiles>
    <profile>
      <id>phantom-power-dev</id>
      <properties>
        <spring.profiles.active>dev</spring.profiles.active>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
      </properties>
    </profile>
  </profiles>
  
  <activeProfiles>
    <activeProfile>phantom-power-dev</activeProfile>
  </activeProfiles>
</settings>
EOF
```

#### Application Properties
```bash
# Create backend environment file
cat > ~/Developer/phantom-power/backend/.env << 'EOF'
# Database Configuration
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/phantom_power_dev
SPRING_DATASOURCE_USERNAME=phantom_user
SPRING_DATASOURCE_PASSWORD=phantom_pass
SPRING_JPA_HIBERNATE_DDL_AUTO=update

# Server Configuration
SERVER_PORT=8080
SPRING_PROFILES_ACTIVE=dev

# JWT Configuration
JWT_SECRET=your_very_long_jwt_secret_key_at_least_256_bits
JWT_EXPIRATION=86400000

# Audio Service Integration
AUDIO_SERVICE_URL=http://localhost:8000
AUDIO_SERVICE_TIMEOUT=30000

# File Storage
FILE_STORAGE_TYPE=local
FILE_STORAGE_PATH=./uploads
FILE_MAX_SIZE=52428800

# Logging
LOGGING_LEVEL_COM_PHANTOMPOWER=DEBUG
LOGGING_LEVEL_ORG_SPRINGFRAMEWORK_WEB=DEBUG
EOF
```

### Audio Service (Python) Configuration

#### Python Environment Setup
```bash
# Install Python and Poetry
brew install python@3.11
brew install poetry

# Configure Poetry
poetry config virtualenvs.in-project true
poetry config virtualenvs.prefer-active-python true
```

#### Virtual Environment Configuration
```bash
# Navigate to audio service directory
cd ~/Developer/phantom-power/audio-service

# Create and activate virtual environment
python3.11 -m venv venv
source venv/bin/activate

# Install dependencies
poetry install

# Create environment file
cat > .env << 'EOF'
# Service Configuration
PORT=8000
HOST=0.0.0.0
DEBUG=true
ENVIRONMENT=development

# Database Connection
DATABASE_URL=postgresql://phantom_user:phantom_pass@localhost:5432/phantom_power_dev

# Audio Processing
MAX_AUDIO_FILE_SIZE=52428800
SUPPORTED_FORMATS=mp3,wav,flac,m4a,aac
TEMP_STORAGE_PATH=./temp

# Analysis Configuration
LIBROSA_BACKEND=soundfile
MADMOM_MODELS_PATH=./models
ANALYSIS_TIMEOUT=300

# Performance
WORKER_PROCESSES=4
WORKER_CONNECTIONS=1000

# Logging
LOG_LEVEL=DEBUG
LOG_FORMAT=json
EOF
```

## Docker Configuration

### Docker Desktop Setup
```bash
# Install Docker Desktop
brew install --cask docker

# Configure Docker settings
# - Resources: Memory 8GB, CPU 4 cores
# - Features: Enable Kubernetes (optional)
# - File Sharing: Add project directory
```

### Docker Development Environment
```bash
# Create docker-compose override for development
cat > ~/Developer/phantom-power/docker-compose.override.yml << 'EOF'
version: '3.8'

services:
  postgres:
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backend/src/main/resources/db/seed:/docker-entrypoint-initdb.d
    environment:
      POSTGRES_DB: phantom_power_dev
      POSTGRES_USER: phantom_user
      POSTGRES_PASSWORD: phantom_pass

  redis:
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "8080:8080"
    volumes:
      - ./backend:/app
    environment:
      SPRING_PROFILES_ACTIVE: dev
    depends_on:
      - postgres
      - redis

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      NODE_ENV: development

  audio-service:
    build:
      context: ./audio-service
      dockerfile: Dockerfile.dev
    ports:
      - "8000:8000"
    volumes:
      - ./audio-service:/app
    environment:
      ENVIRONMENT: development

volumes:
  postgres_data:
  redis_data:
EOF
```

## Verification and Testing

### Environment Verification Script
```bash
# Create verification script
cat > ~/Developer/phantom-power/verify-environment.sh << 'EOF'
#!/bin/bash

echo "= Phantom Power Environment Verification"
echo "========================================"

# Check required tools
echo "=æ Checking required tools..."
command -v node >/dev/null 2>&1 && echo " Node.js: $(node --version)" || echo "L Node.js not found"
command -v pnpm >/dev/null 2>&1 && echo " pnpm: $(pnpm --version)" || echo "L pnpm not found"
command -v java >/dev/null 2>&1 && echo " Java: $(java --version | head -1)" || echo "L Java not found"
command -v mvn >/dev/null 2>&1 && echo " Maven: $(mvn --version | head -1)" || echo "L Maven not found"
command -v python3 >/dev/null 2>&1 && echo " Python: $(python3 --version)" || echo "L Python not found"
command -v poetry >/dev/null 2>&1 && echo " Poetry: $(poetry --version)" || echo "L Poetry not found"
command -v docker >/dev/null 2>&1 && echo " Docker: $(docker --version)" || echo "L Docker not found"
command -v psql >/dev/null 2>&1 && echo " PostgreSQL: $(psql --version)" || echo "L PostgreSQL not found"

# Check environment variables
echo ""
echo "= Checking environment variables..."
[[ -n "$JWT_SECRET" ]] && echo " JWT_SECRET is set" || echo "L JWT_SECRET not set"
[[ -n "$DATABASE_URL" ]] && echo " DATABASE_URL is set" || echo "L DATABASE_URL not set"
[[ -n "$SPRING_DATASOURCE_URL" ]] && echo " SPRING_DATASOURCE_URL is set" || echo "L SPRING_DATASOURCE_URL not set"

# Check database connectivity
echo ""
echo "=Ä  Testing database connection..."
if psql "$DATABASE_URL" -c "SELECT 1;" >/dev/null 2>&1; then
    echo " Database connection successful"
else
    echo "L Database connection failed"
fi

# Check Docker services
echo ""
echo "=3 Checking Docker services..."
if docker ps >/dev/null 2>&1; then
    echo " Docker is running"
    docker-compose -f docker-compose.yml ps
else
    echo "L Docker is not running"
fi

echo ""
echo "<‰ Environment verification complete!"
EOF

chmod +x ~/Developer/phantom-power/verify-environment.sh
```

### Run Verification
```bash
# Run environment verification
cd ~/Developer/phantom-power
./verify-environment.sh
```

## Troubleshooting

### Common Issues

#### Zsh Configuration Not Loading
```bash
# Check if .zshrc is sourced
echo $SHELL  # Should show /bin/zsh

# Force reload zsh configuration
source ~/.zshrc

# Check if secrets file exists and is readable
ls -la ~/.phantom_secrets
cat ~/.phantom_secrets | head -5
```

#### Java Version Issues
```bash
# List all Java versions
/usr/libexec/java_home -V

# Set Java version explicitly
export JAVA_HOME=$(/usr/libexec/java_home -v 17)

# Verify Java version
java -version
javac -version
```

#### Database Connection Issues
```bash
# Check PostgreSQL is running
brew services list | grep postgresql

# Start PostgreSQL if not running
brew services start postgresql@15

# Test connection manually
psql -h localhost -p 5432 -U phantom_user -d phantom_power_dev
```

#### Python Virtual Environment Issues
```bash
# Recreate virtual environment
cd ~/Developer/phantom-power/audio-service
rm -rf venv
python3.11 -m venv venv
source venv/bin/activate
pip install --upgrade pip
poetry install
```

### Performance Optimization

#### IntelliJ IDEA Performance
```bash
# Edit IntelliJ VM options
# Help > Edit Custom VM Options
-Xmx4g
-Xms2g
-XX:ReservedCodeCacheSize=1g
-XX:InitialCodeCacheSize=64m
```

#### Docker Performance
```bash
# Configure Docker Desktop resources
# Docker Desktop > Settings > Resources
# - CPUs: 4-6 cores
# - Memory: 8-12 GB
# - Swap: 1 GB
# - Disk: 60+ GB
```

## Security Considerations

### Secrets Management Best Practices
1. **Never commit secrets to version control**
2. **Use different secrets for each environment**
3. **Rotate secrets regularly**
4. **Use strong, unique passwords**
5. **Enable file permissions on secret files (600)**

### File Permissions
```bash
# Secure secrets files
chmod 600 ~/.phantom_secrets*
chmod 600 ~/.ssh/id_*

# Verify permissions
ls -la ~/.phantom_secrets
ls -la ~/.ssh/
```

<!-- TODO: Add IDE-specific debugging configuration -->
<!-- TODO: Document performance monitoring setup -->
<!-- TODO: Add automated environment setup script -->
<!-- TODO: Create environment troubleshooting checklist -->