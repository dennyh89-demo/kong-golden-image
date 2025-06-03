---
marp: true
theme: default
paginate: true
size: 16:9
header: '🏆 Kong Gateway Golden Images Workshop'
footer: 'Kong Inc. © 2025 | Confidential'
style: |
  section {
    font-size: 28px;
  }
  h1 {
    color: #003f5c;
    font-size: 48px;
  }
  h2 {
    color: #2f4b7c;
    font-size: 40px;
  }
  h3 {
    color: #665191;
    font-size: 32px;
  }
  .columns {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 2rem;
  }
  .highlight {
    background: #ffd700;
    padding: 1rem;
    border-radius: 8px;
  }
---

# 🏆 Kong Gateway Golden Images Workshop

## Building Production-Ready Container Images

**Date:** June 3, 2025  
**Presenter:** Platform Engineering Team  
**Duration:** 2 hours  

---

## 🎯 Workshop Agenda

- **🔍 Golden Images Overview** (20 min)
- **🏗️ Kong Architecture & Versioning** (15 min)
- **🐳 Container Best Practices** (25 min)
- **⚙️ Pipeline Implementation** (30 min)
- **🧪 Testing & Security** (20 min)
- **🚀 Production Deployment** (15 min)
- **❓ Q&A Session** (15 min)

---

## 🎯 Learning Objectives

By the end of this workshop, you will:

- ✅ **Understand** golden image concepts and benefits
- ✅ **Design** production-ready Kong container images
- ✅ **Implement** automated build and test pipelines
- ✅ **Apply** security hardening and compliance practices
- ✅ **Deploy** images using GitOps workflows

---

## 🖼️ What Are Golden Images?

<div class="columns">

### Traditional Approach
- 🔄 Install Kong at deployment time
- 📦 Download packages during startup
- ⚡ Variable performance and reliability
- 🔍 Difficult to audit and track changes

### Golden Image Approach
- ✅ Pre-built, tested, and verified images
- 🛡️ Security hardening baked in
- 📋 Compliance and governance ready
- 🚀 Fast, consistent deployments

</div>

---

## 🏆 Golden Image Benefits

<div class="columns">

### **🛡️ Security & Compliance**
- Known patched vulnerabilities
- Regular security scanning
- Hardened base configurations
- Audit trail and compliance

### **📊 Governance & Control**
- Centralized image management
- Approved software versions
- Policy enforcement
- Deployment consistency

</div>

---

## 🏆 Golden Image Benefits (cont.)

<div class="columns">

### **🚀 Operational Efficiency**
- Faster deployment times
- Reduced runtime failures
- Consistent environments
- Simplified troubleshooting

### **💰 Cost Optimization**
- Reduced support overhead
- Less downtime incidents
- Team productivity gains
- Resource optimization

</div>

---

## 🏗️ Kong Gateway Architecture

```
┌─────────────────────────────────────────────┐
│            🌐 Kong Gateway                  │
├─────────────────────────────────────────────┤
│  🔌 Plugin System    │  ⚙️ Configuration   │
│  ├── Rate Limiting   │  ├── kong.conf      │
│  ├── Authentication  │  ├── nginx.conf     │
│  ├── Transformations │  └── Environment    │
│  └── Custom Plugins  │     Variables       │
├─────────────────────────────────────────────┤
│            🚀 OpenResty (Nginx + LuaJIT)   │
├─────────────────────────────────────────────┤
│            🐧 Operating System Base         │
└─────────────────────────────────────────────┘
```

---

## 📅 Kong Gateway Versioning Strategy

### LTS Release Timeline

```
│  2025  │  Mar  │  Jun  │  Sep  │  Dec  │  2026  │  Mar     │
│        │ 3.10  │ 3.11  │ 3.12  │ 3.13  │        │ 3.14     │
│        │ 🏆LTS │       │       │       │        │ 🏆LTS    │
├────────────────────────────────────────────────────────────┤
│        │◄──────── 3 Year Support ────────►│                │
│                  │◄──────── 3 Year Support ────────►      │
│                  │◄── 2 Year Overlap ──►│                  │
```

---

## 📊 Version Support Matrix

| 🏆 LTS Version | 📅 Release Date | 📅 End of Support | 🔧 Support Type |
|----------------|-----------------|-------------------|------------------|
| **3.10** | March 2025 | March 2028 | 🟢 Full Support |
| **3.14** | March 2026 | March 2029 | 🟡 Planned |
| **3.18** | March 2027 | March 2030 | 🟡 Planned |

---

## 🏗️ Version Format & Patch Strategy

### 📦 Version Format
```
{MAJOR}.{MINOR}.{PATCH}.{ENTERPRISE_PATCH}
   3   .   10   .   0   .   1
```

| Component | Frequency | Description |
|-----------|-----------|-------------|
| 🎯 **Major** | Rare | Breaking changes, architectural shifts |
| 🔄 **Minor** | ~12 weeks | New features, backwards compatible |
| 🛠️ **Patch** | As needed | Bug fixes, security updates |
| 🏢 **Enterprise** | As needed | Enterprise-specific fixes |

---

## 📦 Kong Binary Distribution

### 📡 Cloudsmith Package Repository

```
┌─────────────────────────────────────────────────────────────┐
│              packages.konghq.com (Cloudsmith)               │
├─────────────────────────────────────────────────────────────┤
│  📂 gateway-310/     📂 gateway-34/     📂 gateway-legacy/  │
│     ├── 🐧 DEB          ├── 🐧 DEB         ├── 🐧 DEB       │
│     ├── 📦 RPM          ├── 📦 RPM         └── 📦 RPM       │
│     ├── 💻 AMD64        ├── 💻 AMD64                        │
│     └── 🔧 ARM64        └── 🔧 ARM64                        │
└─────────────────────────────────────────────────────────────┘
```

---

## 📥 Download Examples

### 🐧 Ubuntu/Debian Package
```bash
curl -1sLf -O \
  'https://packages.konghq.com/public/gateway-310/deb/ubuntu/pool/noble/main/k/kong-enterprise-edition/kong-enterprise-edition_3.10.0.1_amd64.deb'
```

### 📦 RHEL/CentOS Package
```bash
curl -1sLf -O \
  'https://packages.konghq.com/public/gateway-310/rpm/el/8/x86_64/kong-enterprise-edition-3.10.0.1-1.el8.x86_64.rpm'
```

---

## 🏗️ Golden Image Pipeline Architecture

### 🔄 High-Level Pipeline Flow

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   📄 Source     │    │  🏭 Build       │    │   📦 Registry   │
│   Control       │───▶│  Pipeline       │───▶│   & Store       │
│                 │    │                 │    │                 │
│ 🐳 Dockerfile   │    │ 🔨 Build Image  │    │ 🏪 Container    │
│ 📜 Scripts      │    │ 🧪 Run Tests    │    │    Registry     │
│ 🧪 Tests        │    │ 🔍 Security     │    │ 📊 Vulnerability│
│ ⚙️  Config      │    │    Scan         │    │    Reports      │
│                 │    │ 📤 Publish      │    │ 📋 Image        │
│                 │    │                 │    │    Catalog     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## ⚡ Pipeline Trigger Events

| 🎯 Trigger | 📋 Description | 🔄 Frequency |
|------------|----------------|--------------|
| 📝 **Code Commit** | New commits to main branch | On-demand |
| 🆕 **Kong Release** | New Kong version available | Weekly check |
| 🛡️ **Security Update** | Critical vulnerabilities found | Scheduled |
| ⏰ **Scheduled Build** | Regular rebuilds for freshness | Weekly |

---

## 🏗️ Detailed Pipeline Stages

### 🎯 Stage 1: Source Control Trigger
- 📝 Git Push to main branch
- 🆕 New Kong version detected
- 🛡️ Critical security vulnerability
- ⏰ Scheduled weekly rebuild

### 🔨 Stage 2: Build
- 📥 Download Kong binary from Cloudsmith
- 🐳 Build custom Docker image
- 🛡️ Apply security hardening
- 🏷️ Tag with semantic version

---

## 🏗️ Detailed Pipeline Stages (cont.)

### 🧪 Stage 3: Testing
- 💨 Smoke tests (basic functionality)
- 📊 Load tests (performance validation)
- 🔍 Security scanning (vulnerabilities)
- 📋 Compliance checks (policies)

### 📤 Stage 4: Publish & Notify
- 🏪 Push to private container registry
- 📋 Update image catalog
- 📧 Notify teams of availability
- 📝 Generate changelog

---

## 📁 Repository Structure

```
kong-golden-image/
├── 📁 .github/workflows/
│   └── 📄 build-image.yaml          # 🚀 GitHub Actions
├── 📁 dockerfiles/
│   ├── 📄 Dockerfile.debian         # 🐧 Debian base
│   ├── 📄 Dockerfile.ubuntu         # 🟠 Ubuntu base
│   └── 📄 Dockerfile.rhel           # 🎩 RHEL base
├── 📁 scripts/
│   ├── 📄 download-kong.sh          # 📥 Binary downloader
│   ├── 📄 security-hardening.sh     # 🛡️ Security config
│   └── 📄 kong-smoke-tests.bats     # 💨 Basic tests
├── 📁 tests/load/k6/
│   └── 📄 load.js                   # 📊 Performance tests
├── 📁 configs/
│   ├── 📄 kong.conf.template        # ⚙️ Kong config
│   └── 📄 nginx.conf.template       # 🌐 Nginx config
└── 📄 README.md                     # 📚 Documentation
```

---

## 🐳 Dockerfile Best Practices

### Production-Ready Example

```dockerfile
# 🏢 Use organization-approved base image
FROM your-org-registry.com/ubuntu:24.04-hardened

# 🏷️ Comprehensive metadata
LABEL org.opencontainers.image.title="Kong Gateway Golden Image" \
      org.opencontainers.image.version="3.10.0.1-1" \
      org.opencontainers.image.vendor="Your Organization"

# 📥 Copy Kong binary (downloaded separately)
COPY kong-enterprise-edition_3.10.0.1_amd64.deb /tmp/kong.deb

# 🔧 Multi-stage installation with cleanup
RUN set -ex; \
    apt-get update && \
    apt-get install --yes /tmp/kong.deb && \
    rm -rf /var/lib/apt/lists/* /tmp/kong.deb
```

---

## 🐳 Dockerfile Best Practices (cont.)

```dockerfile
# 👤 Security: Switch to non-root user
USER kong

# 🌐 Expose Kong ports
EXPOSE 8000 8443 8001 8444

# ❤️ Health monitoring
HEALTHCHECK --interval=10s --timeout=10s --retries=10 \
    CMD kong health

# 🚀 Container startup
ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["kong", "docker-start"]
```

---

## ⚙️ GitHub Actions Workflow

### 🚀 Complete CI/CD Pipeline

```yaml
name: 🏭 Build Kong Golden Image

on:
  push:
    branches: [main]
  schedule:
    - cron: '0 2 * * 1'  # Weekly security updates
  workflow_dispatch:

env:
  REGISTRY: your-org-registry.com
  IMAGE_NAME: kong-gateway
```

---

## ⚙️ GitHub Actions Workflow (cont.)

```yaml
jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
    - name: 📥 Checkout code
      uses: actions/checkout@v4
      
    - name: 🐳 Set up Docker Buildx
      uses: docker/setup-buildx-action@v3
      
    - name: 📥 Download Kong binary
      run: ./scripts/download-kong.sh
      
    - name: 🏗️ Build and load image
      run: docker build -t kong-test .
```

---

## 🧪 Testing Strategy

### 💨 Smoke Tests with BATS

```bash
#!/usr/bin/env bats
# 🧪 Kong Golden Image Smoke Tests

@test "🔧 Kong binary is installed and executable" {
    run docker exec kong-test kong version
    [ "$status" -eq 0 ]
    [[ "$output" == *"Kong"* ]]
}

@test "❤️ Kong health check passes" {
    run docker exec kong-test kong health
    [ "$status" -eq 0 ]
}

@test "🌐 Admin API is responsive" {
    run curl -f -s http://localhost:8001/status
    [ "$status" -eq 0 ]
}
```

---

## 📊 Load Testing with K6

### ⚡ Performance Validation

```javascript
import http from 'k6/http';
import { check } from 'k6';

export let options = {
  stages: [
    { duration: '30s', target: 10 },   // 🚀 Warm up
    { duration: '1m', target: 50 },    // 📈 Ramp up
    { duration: '2m', target: 100 },   // 🏃‍♂️ Peak load
  ],
  thresholds: {
    'http_req_duration': ['p(95)<1000'],  // 95% under 1s
    'http_req_failed': ['rate<0.01'],     // <1% error rate
  },
};
```

---

## 📊 Load Testing with K6 (cont.)

```javascript
export default function () {
  // 🔧 Test Admin API
  let adminResponse = http.get('http://localhost:8001/status');
  check(adminResponse, {
    '✅ Admin API status 200': (r) => r.status === 200,
    '⚡ Admin API under 500ms': (r) => r.timings.duration < 500,
  });
  
  // 🚪 Test Proxy
  let proxyResponse = http.get('http://localhost:8000');
  check(proxyResponse, {
    '✅ Proxy responding': (r) => r.status === 404, // No routes = 404 OK
    '⚡ Proxy under 200ms': (r) => r.timings.duration < 200,
  });
}
```

---

## 🔍 Security Scanning Pipeline

### 🛡️ Multi-Layer Security Approach

<div class="columns">

### Container Image Scanning
- **Trivy** - Vulnerability detection
- **Grype** - SBOM analysis
- **Clair** - Continuous monitoring

### Dockerfile Best Practices
- **Hadolint** - Dockerfile linting
- **Dockle** - Image security checks
- **CIS Benchmarks** - Compliance

</div>

---

## 🚨 Security Thresholds

| 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low |
|-------------|---------|-----------|--------|
| **0** allowed | **≤5** allowed | **≤20** allowed | No limit |
| Build fails | Requires approval | Warning only | Informational |

---

## 🏆 Version Management Strategy

### 🏷️ Semantic Versioning
```
{KONG_VERSION}-{ORG_PATCH}.{BUILD_NUMBER}

Examples:
🏷️ 3.10.0.1-1.245   (Kong 3.10.0.1, org patch 1, build 245)
🏷️ 3.10.0.1-2.250   (Kong 3.10.0.1, org patch 2, build 250)
🏷️ 3.10.1.0-1.255   (Kong 3.10.1.0, org patch 1, build 255)
```

---

## 🎯 Tagging Strategy

### 📦 Production Tags
- `kong-gateway:latest` - 🆕 Most recent stable
- `kong-gateway:3.10.0.1-1.245` - 🔒 Specific version
- `kong-gateway:3.10-latest` - 📌 Latest in series
- `kong-gateway:3.10.0.1-1.245-prod` - 🏭 Production certified

### 🧪 Development Tags
- `kong-gateway:3.10.0.1-1.245-dev` - 🔬 Development
- `kong-gateway:3.10.0.1-1.245-staging` - 🎭 Staging
- `kong-gateway:3.10.0.1-1.245-test` - 🧪 Testing

---

## 🛡️ Security Hardening Checklist

| ✅ Security Control | 🎯 Implementation | 📊 Status |
|---------------------|-------------------|-----------|
| **Non-root user** | USER kong directive | ✅ Required |
| **Minimal base image** | Distroless/Alpine preferred | ✅ Implemented |
| **No package manager** | Remove apt/yum after install | ✅ Required |
| **Read-only filesystem** | --read-only flag | 🟡 Recommended |
| **No shell access** | Remove bash/sh | 🟡 Optional |
| **Secrets via env/volume** | Never embed in image | ✅ Required |

---

## ⚙️ Configuration Management

### 🎛️ Environment-Specific Configuration

```bash
#!/bin/bash
# 🎛️ Dynamic configuration injection

# 🌍 Environment detection
ENVIRONMENT=${ENVIRONMENT:-development}
echo "🎯 Configuring for environment: $ENVIRONMENT"

# 📝 Template substitution
envsubst < /etc/kong/kong.conf.template > /etc/kong/kong.conf

# 🔌 Dynamic plugin configuration
export KONG_PLUGINS="${KONG_PLUGINS:-bundled}"
if [ "$ENVIRONMENT" = "production" ]; then
    export KONG_PLUGINS="$KONG_PLUGINS,rate-limiting-advanced"
fi
```

---

## 🎯 Configuration Template Example

```yaml
# 📄 kong.conf.template
# 🌍 Environment: ${ENVIRONMENT}

# 🗄️ Database Configuration
database = ${KONG_DATABASE:-postgres}
pg_host = ${KONG_PG_HOST:-postgres}
pg_port = ${KONG_PG_PORT:-5432}
pg_database = ${KONG_PG_DATABASE:-kong}
pg_user = ${KONG_PG_USER:-kong}
pg_password = ${KONG_PG_PASSWORD}

# 🌐 Network Configuration
proxy_listen = ${KONG_PROXY_LISTEN:-0.0.0.0:8000, 0.0.0.0:8443 ssl}
admin_listen = ${KONG_ADMIN_LISTEN:-0.0.0.0:8001, 0.0.0.0:8444 ssl}
```

---

## 📊 Monitoring & Observability

### 📈 Built-in Monitoring Capabilities

<div class="columns">

### ❤️ Health Checks
- Endpoint: `/status`
- Interval: 30s
- Timeout: 5s
- Retries: 3

### 📏 Metrics Collection
- **Prometheus**: `/metrics`
- **StatsD**: UDP metrics
- **Structured Logs**: JSON format

</div>

---

## 🏗️ Multi-Architecture Support

### 🌍 Cross-Platform Building

```yaml
platforms:
  - linux/amd64     # 💻 Intel/AMD 64-bit
  - linux/arm64     # 🔧 ARM 64-bit (Apple Silicon, AWS Graviton)
```

### ⚡ Build Optimization

```dockerfile
FROM --platform=$BUILDPLATFORM alpine:latest AS downloader
ARG TARGETARCH
ARG KONG_VERSION=3.10.0.1

RUN case "$TARGETARCH" in \
    "amd64") ARCH_SUFFIX="_amd64" ;; \
    "arm64") ARCH_SUFFIX="_arm64" ;; \
    *) echo "Unsupported architecture: $TARGETARCH" && exit 1 ;; \
    esac
```

---

## 🚀 Implementation Roadmap

### 📅 8-Week Implementation Plan

| Week | 🎯 Focus | 📋 Deliverables |
|------|----------|-----------------|
| **1-2** | 🏗️ Foundation | Repository setup, Basic Dockerfile, GitHub Actions |
| **3-4** | 🧪 Testing & Security | Smoke tests, Load tests, Security scanning |
| **5-6** | 🏭 Production Readiness | Multi-arch support, Build optimization |
| **7-8** | 🚀 Advanced Features | Automated updates, Monitoring integration |

---

## 🎯 Success Metrics

| 📊 Metric | 🎯 Target | 🏆 Goal |
|-----------|-----------|---------|
| **Build Success Rate** | >95% | 99% |
| **Security Scan Pass Rate** | >90% | 95% |
| **Image Build Time** | <10 min | <5 min |
| **Image Size** | <300MB | <250MB |
| **Deploy Time** | <2 min | <1 min |
| **Zero-day Response** | <24h | <4h |

---

## 🎯 Key Takeaways

### ✅ Golden Image Benefits

<div class="columns">

### 🛡️ Security & Compliance
- Known patches
- Vulnerability scanning
- Hardened base
- Audit trails

### 📋 Governance & Efficiency
- Centralized control
- Faster deployments
- Consistent environments
- Cost savings

</div>

---

## 🚀 Next Steps & Action Items

### 📋 Immediate Actions (This Week)
- 📁 Create golden image repository
- 🔐 Set up container registry access
- 🐳 Write basic Dockerfile
- 📥 Test Kong binary download process
- 🚀 Set up basic GitHub Actions workflow

### 📅 Short-term Goals (Next Month)
- 🧪 Implement comprehensive test suite
- 🔍 Add security scanning pipeline
- 📊 Create monitoring dashboards
- 👥 Train development teams

---

## ❓ Q&A Session

### 🤔 Common Questions

**Q: How often should we rebuild golden images?**
A: 📅 Minimum for security patches, 📈 Monthly recommended, 🚨 24h for critical vulnerabilities

**Q: Can we include custom plugins?**
A: ✅ Yes! Pre-installing reduces deployment time, but consider image size trade-offs

**Q: How do we handle Kong Enterprise licenses?**
A: 🚫 Never embed in images. Use environment variables or mounted secrets

---

## 📚 Essential Resources

### 📖 Documentation
- **[Kong Gateway Docs](https://docs.konghq.com/gateway/latest/)** - Complete Kong documentation
- **[Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)** - Official Docker guidelines
- **[Container Security Guide](https://kubernetes.io/docs/concepts/security/)** - K8s security best practices

### 🛠️ Tools & Repositories
- **[Kong Cloudsmith](https://packages.konghq.com/)** - Official Kong packages
- **[Kong Docker Images](https://github.com/Kong/docker-kong)** - Reference implementations

---

## 📞 Stay Connected

**📧 Workshop Follow-up**
- 📬 Email: `golden-images@yourorg.com`
- 💬 Slack: `#kong-golden-images`
- 📅 Office Hours: Fridays 2-3 PM

**🌟 Share Your Success**
- 📱 Social: Tag `@KongHQ` and `#KongGoldenImages`
- 📝 Blog: Share your implementation story
- 🎤 Speak: Present at Kong meetups and conferences

---

## 📚 Quick Reference

### 🔧 Essential Commands

```bash
# 📥 Download Kong Binary
curl -1sLf -O "https://packages.konghq.com/public/gateway-310/deb/ubuntu/pool/noble/main/k/kong-enterprise-edition/kong-enterprise-edition_3.10.0.1_amd64.deb"

# 🐳 Build Image
docker build -t kong-golden:3.10.0.1 .

# 🧪 Run Tests
docker run --rm kong-golden:3.10.0.1 kong version
./scripts/kong-smoke-tests.bats

# 🔍 Security Scan
trivy image kong-golden:3.10.0.1

# 📤 Push to Registry
docker push your-registry.com/kong-golden:3.10.0.1
```

---

## 📋 Release Checklist

<div class="highlight">

**Golden Image Release Checklist:**
- □ Kong binary downloaded and verified
- □ Dockerfile passes hadolint validation
- □ Image builds successfully for all architectures
- □ Smoke tests pass (BATS)
- □ Load tests meet performance thresholds (K6)
- □ Security scan shows no critical vulnerabilities
- □ Configuration templates validated
- □ Documentation updated
- □ Registry push successful
- □ Teams notified of new image availability

</div>

---

# 🎉 Workshop Complete!

## 🏆 Congratulations!

**🏅 You're now a Kong Golden Image Expert!**

### 📚 What you've learned:
- 🖼️ Golden Image fundamentals
- 🏗️ Production-ready pipeline design
- 🧪 Comprehensive testing strategies
- 🔍 Security and compliance best practices
- 🚀 Real-world implementation roadmap

### 🎯 Your next mission:
**Build your organization's first Kong Golden Image!**

---

## 🙏 Thank You!

**🚀 Now go forth and build golden images that sparkle!** ✨

*For ongoing support and questions:*
- 📧 golden-images-support@yourorg.com
- 💬 #kong-golden-images Slack channel
- 📚 Internal Wiki: wiki.yourorg.com/kong-golden-images

**Kong Inc. © 2025 | Confidential**