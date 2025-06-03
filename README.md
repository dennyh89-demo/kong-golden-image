# Kong Gateway Golden Images Workshop

## Building Production-Ready Container Images with Enterprise Governance

---

## Agenda

1. **What is a Golden Image?**
2. **Why Build Custom Images vs. Using Pre-built Images?**
3. **Kong's Release & Support Lifecycle**
4. **Kong Binary Distribution via Cloudsmith**
5. **Golden Image Pipeline Architecture**
6. **Implementation: Step-by-Step Process**
7. **Testing & Quality Assurance**
8. **Best Practices & Governance**
9. **Demo & Hands-On**

---

## What is a Golden Image?

### Definition
A **Golden Image** is a standardized, approved, and tested container image that serves as the foundation for deploying applications across your organization.

### Key Characteristics
- **Standardized**: Consistent base configuration across all environments
- **Approved**: Validated by security, compliance, and engineering teams
- **Tested**: Thoroughly validated through automated testing pipelines
- **Versioned**: Proper semantic versioning and change tracking
- **Immutable**: Once created, never modified (new versions are created instead)

### Benefits
- **Security**: Known, patched base images with vulnerability scanning
- **Compliance**: Meets organizational security and regulatory requirements
- **Consistency**: Identical runtime environments across dev, staging, and production
- **Governance**: Centralized control over what runs in production
- **Auditability**: Clear provenance and change history

---

## Why Build Custom Images vs. Pre-built Images?

### The Challenge with Pre-built Images

#### Kong's Official Docker Hub Images
- Built on Kong's chosen base images (Debian, RHEL)
- Fixed dependency versions
- Limited customization options
- May not meet enterprise security requirements
- External dependency on Docker Hub availability

#### Organizational Requirements
- **Security Hardening**: Custom security configurations, removed packages
- **Compliance**: SOC2, FIPS, PCI-DSS requirements
- **Base Image Standards**: Organization-approved base images only
- **Custom Dependencies**: Additional tools, certificates, monitoring agents
- **Air-gapped Environments**: No external registry access

---

## Kong's Release & Support Lifecycle

### Version Support Policy

#### Long-Term Support (LTS) Strategy
- **4 minor versions per year** (March, June, September, December)
- **1 LTS release annually** (March release becomes LTS)
- **3 years of support** per LTS version
- **2-year overlap** between adjacent LTS releases

#### Version Format: `{MAJOR}.{MINOR}.{PATCH}.{ENTERPRISE_PATCH}`
- **Major**: Rare, breaking changes
- **Minor**: New features, released every ~12 weeks
- **Patch**: Bug fixes and security updates
- **Enterprise Patch**: Enterprise-specific fixes

### Current LTS Schedule
| LTS Version | Release Date | End of Support |
|-------------|--------------|----------------|
| 3.10        | March 2025   | March 2028     |
| 3.14        | March 2026   | March 2029     |
| 3.18        | March 2027   | March 2030     |

### Why This Matters for Golden Images
- **Predictable Upgrade Cycles**: Plan image updates around LTS releases
- **Security Patches**: Regular patches require image rebuilds
- **Support Overlap**: Maintain multiple image versions during transitions
- **Governance**: Track which versions are approved for production use

---

## Kong Binary Distribution via Cloudsmith

### Cloudsmith Package Repository
Kong distributes official binaries through **packages.konghq.com** (hosted by Cloudsmith)

### Repository Structure
- **Per Major.Minor Version**: `gateway-310`, `gateway-34`, etc.
- **Multi-format Support**: DEB, RPM packages in single repository
- **Multiple Architectures**: AMD64, ARM64 support
- **Package Types**:
  - `kong` (OSS Gateway)
  - `kong-enterprise-edition` (Enterprise Gateway)

### Downloading Kong Binaries

#### Example: Kong Gateway 3.10 Enterprise
```bash
# Download DEB package
curl -1sLf -O \
  'https://packages.konghq.com/public/gateway-310/deb/ubuntu/pool/noble/main/k/kong-enterprise-edition/kong-enterprise-edition_3.10.0.1_amd64.deb'

# Download RPM package
curl -1sLf -O \
  'https://packages.konghq.com/public/gateway-310/rpm/el/8/x86_64/kong-enterprise-edition-3.10.0.1-1.el8.x86_64.rpm'
```

### Benefits of Using Official Binaries
- **Authenticity**: Cryptographically signed packages
- **Security**: Pre-built with known dependencies
- **Support**: Backed by Kong's support team
- **Reproducibility**: Consistent builds across environments

---

## Golden Image Pipeline Architecture

### High-Level Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Source Code   │    │  Build Pipeline │    │   Image Store   │
│                 │───▶│                 │───▶│                 │
│ • Dockerfile    │    │ • Build Image   │    │ • Container     │
│ • Scripts       │    │ • Run Tests     │    │   Registry      │
│ • Tests         │    │ • Security Scan │    │ • Vulnerability │
│ • Config        │    │ • Publish       │    │   Reports       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Pipeline Stages

1. **Source Control Trigger**
   - New commits to main branch
   - New Kong version releases
   - Scheduled security updates

2. **Build Stage**
   - Download Kong binary from Cloudsmith
   - Build custom Docker image
   - Apply security hardening

3. **Test Stage**
   - Smoke tests (basic functionality)
   - Load tests (performance validation)
   - Security scanning

4. **Publish Stage**
   - Tag with semantic version
   - Push to private container registry
   - Update image catalog

5. **Notification Stage**
   - Notify teams of new image availability
   - Generate changelog
   - Update documentation

---

## Implementation: Step-by-Step Process

### 1. Repository Structure

```
kong-golden-image/
├── .github/
│   └── workflows/
│       └── build-image.yaml
├── dockerfiles/
│   ├── Dockerfile.debian
│   ├── Dockerfile.ubuntu
│   └── Dockerfile.rhel
├── scripts/
│   ├── download-kong.sh
│   ├── security-hardening.sh
│   └── kong-smoke-tests.bats
├── tests/
│   ├── load/
│   │   └── k6/
│   │       └── load.js
│   └── security/
├── configs/
│   ├── kong.conf.template
│   └── nginx.conf.template
└── README.md
```

### 2. Base Dockerfile Example

```dockerfile
# Use organization-approved base image
FROM your-org-registry.com/ubuntu:24.04-hardened

# Metadata
LABEL org.opencontainers.image.title="Kong Gateway Golden Image"
LABEL org.opencontainers.image.description="Production-ready Kong Gateway"
LABEL org.opencontainers.image.version="3.10.0.1-1"
LABEL org.opencontainers.image.vendor="Your Organization"

# Download and install Kong binary
COPY kong-enterprise-edition_3.10.0.1_amd64.deb /tmp/kong.deb

# Install Kong and dependencies
RUN set -ex; \
    apt-get update && \
    apt-get install --yes /tmp/kong.deb && \
    rm -rf /var/lib/apt/lists/* && \
    rm -rf /tmp/kong.deb && \
    \
    # Security hardening
    rm -rf /usr/share/doc/* && \
    rm -rf /usr/share/man/* && \
    \
    # Set proper ownership
    chown kong:0 /usr/local/bin/kong && \
    chown -R kong:0 /usr/local/kong && \
    \
    # Create symlinks
    ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit && \
    ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua && \
    ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx && \
    \
    # Verify installation
    kong version

# Copy custom configurations
COPY configs/kong.conf.template /etc/kong/kong.conf.template
COPY scripts/docker-entrypoint.sh /docker-entrypoint.sh

# Set proper permissions
RUN chmod +x /docker-entrypoint.sh

# Switch to non-root user
USER kong

# Expose standard Kong ports
EXPOSE 8000 8443 8001 8444 8002 8445 8003 8446 8004 8447

# Health check
HEALTHCHECK --interval=10s --timeout=10s --retries=10 \
    CMD kong health

# Default command
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["kong", "docker-start"]
```

### 3. GitHub Actions Workflow

```yaml
name: Build Kong Golden Image

on:
  push:
    branches: [main]
  schedule:
    # Weekly security updates
    - cron: '0 2 * * 1'
  workflow_dispatch:
    inputs:
      kong_version:
        description: 'Kong version to build'
        required: true
        default: '3.10.0.1'

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
      
    - name: Login to registry
      uses: docker/login-action@v3
      with:
        registry: your-org-registry.com
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
        
    - name: Download Kong binary
      run: |
        chmod +x scripts/download-kong.sh
        ./scripts/download-kong.sh ${{ github.event.inputs.kong_version || '3.10.0.1' }}
        
    - name: Build image
      uses: docker/build-push-action@v5
      with:
        context: .
        file: dockerfiles/Dockerfile.ubuntu
        platforms: linux/amd64,linux/arm64
        push: false
        tags: |
          your-org-registry.com/kong-gateway:${{ github.event.inputs.kong_version || '3.10.0.1' }}
          your-org-registry.com/kong-gateway:latest
        cache-from: type=gha
        cache-to: type=gha,mode=max
        
    - name: Run smoke tests
      run: |
        docker run --rm -d --name kong-test \
          your-org-registry.com/kong-gateway:${{ github.event.inputs.kong_version || '3.10.0.1' }}
        sleep 30
        chmod +x scripts/kong-smoke-tests.bats
        ./scripts/kong-smoke-tests.bats
        docker stop kong-test
        
    - name: Run security scan
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: your-org-registry.com/kong-gateway:${{ github.event.inputs.kong_version || '3.10.0.1' }}
        format: 'sarif'
        output: 'trivy-results.sarif'
        
    - name: Run load tests
      run: |
        docker run --rm -d --name kong-load-test \
          -p 8000:8000 \
          your-org-registry.com/kong-gateway:${{ github.event.inputs.kong_version || '3.10.0.1' }}
        sleep 30
        npx k6 run tests/load/k6/load.js
        docker stop kong-load-test
        
    - name: Push image
      if: success()
      uses: docker/build-push-action@v5
      with:
        context: .
        file: dockerfiles/Dockerfile.ubuntu
        platforms: linux/amd64,linux/arm64
        push: true
        tags: |
          your-org-registry.com/kong-gateway:${{ github.event.inputs.kong_version || '3.10.0.1' }}
          your-org-registry.com/kong-gateway:latest
```

---

## Testing & Quality Assurance

### 1. Smoke Tests (BATS)

```bash
#!/usr/bin/env bats

@test "Kong binary is installed and executable" {
    run docker exec kong-test kong version
    [ "$status" -eq 0 ]
    [[ "$output" == *"Kong"* ]]
}

@test "Kong can start successfully" {
    run docker exec kong-test kong health
    [ "$status" -eq 0 ]
}

@test "Kong Admin API is responsive" {
    run curl -f http://localhost:8001/status
    [ "$status" -eq 0 ]
}

@test "Kong Proxy is responsive" {
    run curl -f http://localhost:8000
    [ "$status" -eq 0 ]
}

@test "Essential plugins are available" {
    run docker exec kong-test kong config -c /etc/kong/kong.conf init
    [ "$status" -eq 0 ]
    
    run curl -s http://localhost:8001/plugins/available
    [[ "$output" == *"rate-limiting"* ]]
    [[ "$output" == *"key-auth"* ]]
    [[ "$output" == *"cors"* ]]
}
```

### 2. Load Tests (K6)

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up
    { duration: '5m', target: 100 }, // Sustained load
    { duration: '2m', target: 0 },   // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<1000'], // 95% of requests under 1s
    http_req_failed: ['rate<0.01'],    // Error rate under 1%
  },
};

export default function () {
  // Test Kong Admin API
  let adminResponse = http.get('http://localhost:8001/status');
  check(adminResponse, {
    'Admin API status is 200': (r) => r.status === 200,
    'Admin API response time < 500ms': (r) => r.timings.duration < 500,
  });

  // Test Kong Proxy
  let proxyResponse = http.get('http://localhost:8000');
  check(proxyResponse, {
    'Proxy status is 404 (no routes configured)': (r) => r.status === 404,
    'Proxy response time < 200ms': (r) => r.timings.duration < 200,
  });

  sleep(1);
}
```

### 3. Security Scanning

```yaml
# Security scan configuration
security:
  tools:
    - trivy          # Vulnerability scanning
    - hadolint       # Dockerfile linting
    - dockle         # Container image linter
    - grype          # Alternative vulnerability scanner
    
  policies:
    max_critical: 0
    max_high: 5
    max_medium: 20
    
  exemptions:
    - CVE-2024-12345  # False positive
    - CVE-2024-67890  # Accepted risk with mitigation
```

---

## Best Practices & Governance

### 1. Version Management

#### Semantic Versioning Strategy
```
{KONG_VERSION}-{ORG_PATCH}.{BUILD_NUMBER}

Examples:
- 3.10.0.1-1.245   (Kong 3.10.0.1, org patch 1, build 245)
- 3.10.0.1-2.250   (Kong 3.10.0.1, org patch 2, build 250)
```

#### Tagging Strategy
- **Latest**: `kong-gateway:latest`
- **Version**: `kong-gateway:3.10.0.1-1.245`
- **Major.Minor**: `kong-gateway:3.10-latest`
- **Environment**: `kong-gateway:3.10.0.1-1.245-prod`

### 2. Security Hardening

```bash
#!/bin/bash
# security-hardening.sh

# Remove unnecessary packages
apt-get remove --purge -y \
  curl \
  wget \
  nano \
  vim \
  ssh \
  telnet

# Remove package manager caches
rm -rf /var/lib/apt/lists/*
rm -rf /var/cache/apt/*

# Remove documentation and man pages
rm -rf /usr/share/doc/*
rm -rf /usr/share/man/*
rm -rf /usr/share/info/*

# Set proper file permissions
chmod 755 /usr/local/bin/kong
chmod -R 755 /usr/local/kong

# Remove SUID bits
find / -perm -4000 -exec chmod u-s {} \; 2>/dev/null || true
find / -perm -2000 -exec chmod g-s {} \; 2>/dev/null || true

# Create non-root user if not exists
groupadd -r kong || true
useradd -r -g kong -s /bin/false kong || true
```

### 3. Configuration Management

```bash
# Environment-specific configuration injection
envsubst < /etc/kong/kong.conf.template > /etc/kong/kong.conf

# Dynamic plugin loading
KONG_PLUGINS="${KONG_PLUGINS:-bundled,custom-plugin}"

# Secrets management
KONG_DATABASE_PASSWORD="${DATABASE_PASSWORD:-$(cat /run/secrets/db_password)}"
```

### 4. Monitoring & Observability

```yaml
# Built-in monitoring capabilities
monitoring:
  healthcheck:
    endpoint: /status
    interval: 30s
    timeout: 5s
    
  metrics:
    prometheus: enabled
    statsd: enabled
    
  logging:
    level: info
    format: json
    outputs:
      - stdout
      - /var/log/kong/kong.log
```

### 5. Multi-Architecture Support

```yaml
# Build for multiple architectures
platforms:
  - linux/amd64
  - linux/arm64
  
# Architecture-specific optimizations
buildargs:
  - ARCH=${TARGETARCH}
  - VARIANT=${TARGETVARIANT}
```

---

## Implementation Workflow

### Phase 1: Foundation (Week 1-2)
1. **Repository Setup**
   - Create golden image repository
   - Set up GitHub Actions workflow
   - Configure container registry access

2. **Basic Pipeline**
   - Implement Dockerfile for primary base image
   - Create download script for Kong binaries
   - Set up basic smoke tests

### Phase 2: Testing & Security (Week 3-4)
1. **Enhanced Testing**
   - Implement comprehensive smoke tests
   - Add load testing with K6
   - Create security scanning pipeline

2. **Hardening**
   - Apply security hardening scripts
   - Implement vulnerability scanning
   - Add compliance checking

### Phase 3: Production Readiness (Week 5-6)
1. **Multi-Architecture**
   - Add ARM64 support
   - Optimize build times with caching
   - Implement parallel builds

2. **Governance**
   - Create approval workflows
   - Add automated notifications
   - Implement change tracking

### Phase 4: Advanced Features (Week 7-8)
1. **Automation**
   - Automated Kong version detection
   - Scheduled security updates
   - Integration with change management

2. **Monitoring**
   - Build success/failure tracking
   - Performance metrics collection
   - Usage analytics

---

## Key Takeaways

### Benefits of Golden Images
✅ **Security**: Controlled, scanned, and approved base images  
✅ **Compliance**: Meet organizational and regulatory requirements  
✅ **Consistency**: Identical runtime environments across all stages  
✅ **Governance**: Centralized control over production deployments  
✅ **Support**: Better troubleshooting with known configurations  

### Implementation Success Factors
🎯 **Start Simple**: Begin with basic functionality, iterate  
🎯 **Automate Everything**: Reduce manual intervention and errors  
🎯 **Test Thoroughly**: Comprehensive validation before production  
🎯 **Document Well**: Clear processes for maintenance and updates  
🎯 **Monitor Continuously**: Track performance and security metrics  

### Next Steps
1. **Pilot Project**: Start with non-production environments
2. **Team Training**: Ensure teams understand the new process
3. **Gradual Rollout**: Phase migration from existing images
4. **Continuous Improvement**: Regular review and optimization

---

## Questions & Discussion

### Common Questions

**Q: How often should we rebuild golden images?**  
A: At minimum for security patches, ideally monthly, or when new Kong versions are released.

**Q: Can we include custom plugins in golden images?**  
A: Yes, custom plugins can be pre-installed, but consider the trade-off between image size and deployment flexibility.

**Q: How do we handle Kong Enterprise licenses in images?**  
A: Never embed licenses in images. Use environment variables or mounted secrets at runtime.

**Q: What's the impact on deployment speed?**  
A: Initial build time increases, but deployment speed improves due to pre-built, cached images.

---

## Resources & Links

### Documentation
- [Kong Gateway Installation Guide](https://docs.konghq.com/gateway/latest/install/)
- [Building Custom Docker Images](https://docs.konghq.com/gateway/latest/install/docker/build-custom-images/)
- [Kong Support Policy](https://docs.konghq.com/gateway/latest/support-policy/)

### Tools & Repositories
- [Kong Cloudsmith Packages](https://packages.konghq.com/)
- [Docker Official Images](https://github.com/docker-library/official-images)
- [Container Security Best Practices](https://cheatsheetseries.owasp.org/cheatsheets/Docker_Security_Cheat_Sheet.html)

### Community
- [Kong Community Forum](https://discuss.konghq.com/)
- [Kong GitHub Discussions](https://github.com/Kong/kong/discussions)
- [Kong Academy](https://education.konghq.com/)

---

**Thank you for attending the Kong Gateway Golden Images Workshop!**

*For questions or support, contact: [your-email@organization.com]*