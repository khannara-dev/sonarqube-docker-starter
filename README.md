# SonarQube Docker Starter

A ready-to-use local SonarQube setup with Docker Compose. Perfect for unlimited code quality analysis of private repositories without SonarCloud's line limits.

## Why Local SonarQube?

| Feature | SonarCloud Free | Local SonarQube |
|---------|-----------------|------------------|
| Private repos | 50 lines limit | **Unlimited** |
| Analysis speed | Cloud latency | **Local speed** |
| Offline work | No | **Yes** |
| Monthly cost | $14+/month | **$0** |
| Data privacy | Cloud | **Your machine** |

## Quick Start

```bash
# Clone this repo
git clone https://github.com/khannara/sonarqube-docker-starter.git
cd sonarqube-docker-starter

# Start SonarQube (first run downloads ~800MB)
docker compose up -d

# Wait for startup (~1-2 minutes)
docker compose logs -f sonarqube
# Look for: "SonarQube is operational"
```

## Access

| Item | Value |
|------|-------|
| **URL** | http://localhost:9000 |
| **Default Login** | admin / admin |
| **First Login** | You'll be prompted to change password |

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
docker stats sonarqube sonarqube-db
```

## Resource Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| **RAM** | 2GB | 4GB |
| **Disk** | 1GB | 5GB |
| **CPU** | 2 cores | 4 cores |

Docker Desktop should have at least 4GB RAM allocated.

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
