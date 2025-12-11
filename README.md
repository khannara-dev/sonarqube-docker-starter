# Local CI/CD Infrastructure: Jenkins + SonarQube

A ready-to-use local CI/CD stack with Jenkins and SonarQube via Docker Compose. Perfect for enterprise-grade pipelines and unlimited code quality analysis without cloud limitations.

## Why Local SonarQube?

| Feature | SonarCloud Free | Local SonarQube |
|---------|-----------------|------------------|
| Private repos | 50 lines limit | **Unlimited** |
| Analysis speed | Cloud latency | **Local speed** |
| Offline work | No | **Yes** |
| Monthly cost | $14+/month | **$0** |
| Data privacy | Cloud | **Your machine** |

## Services Included

| Service | Port | Purpose |
|---------|------|---------|
| **Jenkins** | 8080 | CI/CD automation server |
| **SonarQube** | 9000 | Code quality & security analysis |
| **PostgreSQL** | (internal) | SonarQube database |

## Quick Start

```bash
# Clone this repo
git clone https://github.com/khannara/sonarqube-docker-starter.git
cd sonarqube-docker-starter

# Start all services (first run downloads ~1.5GB)
docker compose up -d

# Wait for startup (~2-3 minutes)
docker compose logs -f
```

## Access

### Jenkins
| Item | Value |
|------|-------|
| **URL** | http://localhost:8080 |
| **Initial Password** | `docker logs jenkins` (first line contains unlock key) |
| **First Login** | Complete setup wizard, install suggested plugins |

### SonarQube
| Item | Value |
|------|-------|
| **URL** | http://localhost:9000 |
| **Default Login** | admin / admin |
| **First Login** | You'll be prompted to change password |

---

## Jenkins Setup

### 1. Initial Setup

After first start, get the initial admin password:
```bash
docker logs jenkins 2>&1 | grep -A 2 "initial"
```

### 2. Install Plugins

During setup wizard, install **suggested plugins**, then add these essential ones:
- **NodeJS Plugin** - For npm/node projects
- **SonarQube Scanner** - Code quality integration
- **Discord Notifier** - Build notifications (optional)

Or install via CLI:
```bash
docker exec jenkins jenkins-plugin-cli --plugins nodejs sonar discord-notifier
docker restart jenkins
```

### 3. Configure NodeJS

1. **Manage Jenkins** → **Tools**
2. Under **NodeJS installations**, click **Add NodeJS**
   - Name: `NodeJS-20`
   - Version: `NodeJS 20.x`
   - Install automatically: ✓
3. Save

### 4. Configure SonarQube Server

1. **Manage Jenkins** → **System**
2. Under **SonarQube servers**, click **Add SonarQube**
   - Name: `SonarQube-Local`
   - Server URL: `http://sonarqube:9000` (Docker internal network)
   - Server authentication token: (create in SonarQube, add as credential)
3. Save

### 5. Configure Credentials

1. **Manage Jenkins** → **Credentials** → **System** → **Global credentials**
2. Add credentials for:
   - `sonarqube-token` (Secret text) - SonarQube analysis token
   - `github-credentials` (Username with password/PAT)
   - `discord-webhook-builds` (Secret text) - Discord webhook URL
   - `npm-token` (Secret text) - GitHub Packages token (if using private packages)

### 6. Create Pipeline Job

1. **New Item** → Enter name → Select **Pipeline**
2. Configure:
   - **Pipeline from SCM**: Git
   - **Repository URL**: `https://github.com/khannara/your-repo.git`
   - **Script Path**: `Jenkinsfile`
3. Save & Build

### Jenkinsfile Template

See `templates/Jenkinsfile.template` for a complete example that mirrors Azure DevOps pipeline patterns.

---

## Generate Analysis Token

1. Go to http://localhost:9000
2. Click your profile -> **My Account**
3. Go to **Security** tab
4. Generate token:
   - Name: `my-scanner`
   - Type: `Global Analysis Token`
   - Expires: Never (or set expiry)
5. **Copy the token** (only shown once!)

## Configure Your Project

Create `sonar-project.properties` in your project root:

```properties
# Project identification
sonar.projectKey=my-project-key
sonar.projectName=My Project
sonar.projectVersion=1.0.0

# Source configuration
sonar.sources=src
sonar.exclusions=**/*.test.ts,**/*.test.tsx,**/__tests__/**,**/node_modules/**,**/*.d.ts

# Test configuration
sonar.tests=src
sonar.test.inclusions=**/*.test.ts,**/*.test.tsx,**/__tests__/**

# Coverage (from Jest)
sonar.javascript.lcov.reportPaths=coverage/lcov.info

# TypeScript
sonar.typescript.tsconfigPath=tsconfig.json

# Encoding
sonar.sourceEncoding=UTF-8
```

## Run Analysis

### Option 1: Install SonarScanner CLI

```bash
# Install globally
npm install -g sonar-scanner

# Or use npx
npx sonar-scanner
```

### Option 2: Run with explicit parameters

```bash
sonar-scanner \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=YOUR_TOKEN_HERE
```

### Option 3: Add npm scripts

```json
{
  "scripts": {
    "sonar": "sonar-scanner -Dsonar.host.url=http://localhost:9000 -Dsonar.token=$SONAR_TOKEN"
  }
}
```

## Environment Variables (Optional)

```bash
# Linux/macOS (~/.bashrc or ~/.zshrc)
export SONAR_HOST_URL="http://localhost:9000"
export SONAR_TOKEN="your-token-here"
```

```powershell
# Windows PowerShell (User level)
[Environment]::SetEnvironmentVariable('SONAR_HOST_URL', 'http://localhost:9000', 'User')
[Environment]::SetEnvironmentVariable('SONAR_TOKEN', 'your-token-here', 'User')
```

## CI/CD Integration

### GitHub Actions

```yaml
- name: SonarQube Analysis
  run: |
    npm install -g sonar-scanner
    sonar-scanner \
      -Dsonar.host.url=${{ secrets.SONAR_HOST_URL }} \
      -Dsonar.token=${{ secrets.SONAR_TOKEN }}
```

### Azure DevOps

```yaml
- script: |
    npm install -g sonar-scanner
    sonar-scanner \
      -Dsonar.host.url=$(SONAR_HOST_URL) \
      -Dsonar.token=$(SONAR_TOKEN)
  displayName: 'SonarQube Analysis'
  env:
    SONAR_TOKEN: $(SONAR_TOKEN)
```

**Note:** For CI/CD to access your local SonarQube, you'll need to expose it via:
- Cloudflare Tunnel (recommended)
- ngrok
- VPN
- Public IP with firewall rules

## Managing the Instance

```bash
# Stop (preserves data)
docker compose stop

# Start
docker compose start

# Restart
docker compose restart

# View logs
docker compose logs -f

# Full teardown (WARNING: removes all data)
docker compose down -v

# View resource usage
docker stats jenkins sonarqube sonarqube-db
```

## Resource Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **RAM** | 4GB | 8GB |
| **Disk** | 2GB | 10GB |
| **CPU** | 2 cores | 4 cores |

Docker Desktop should have at least 6GB RAM allocated (Jenkins ~1GB + SonarQube ~2-3GB + PostgreSQL ~256MB).

## Quality Gate Configuration

Configure quality gates at: http://localhost:9000/quality_gates

**Recommended gate conditions:**
- Coverage on new code: > 80%
- Duplicated lines on new code: < 3%
- Maintainability rating: A
- Reliability rating: A
- Security rating: A
- Security hotspots reviewed: 100%

## Troubleshooting

### SonarQube won't start
Ensure Docker Desktop has at least 4GB RAM allocated.

### Port 9000 in use
Change port in docker-compose.yml:
```yaml
ports:
  - "9001:9000"  # Access at http://localhost:9001
```

### Reset admin password
```bash
docker exec -it sonarqube-db psql -U sonarqube -d sonarqube -c \
  "UPDATE users SET crypted_password='\$2a\$12\$uCkkXmhW5ThVK8mpBvnXOOJRLd64LJeHTeCkSuB3lfaR2N0AYBaSi', salt=null, hash_method='BCRYPT' WHERE login='admin';"
docker compose restart sonarqube
# Password is now: admin
```

### Elasticsearch max virtual memory error
```bash
# Linux only
sudo sysctl -w vm.max_map_count=524288
```

## License

MIT License - Use freely in your projects!

## Contributing

PRs welcome! This template is designed to help developers get started with local code quality analysis quickly.

## Author

Created by [Khannara Phay](https://khannara.dev) - Senior SDET / Dev Lead
