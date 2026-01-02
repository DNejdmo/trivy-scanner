# Trivy Security Scanner - Helm Chart

[![Trivy](https://img.shields.io/badge/Trivy-0.57.1-blue)](https://github.com/aquasecurity/trivy)
[![Helm](https://img.shields.io/badge/Helm-v3-blue)](https://helm.sh/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

A complete Kubernetes security solution using five automated scanners and sending results to Discord.
## 🚀 Features

- ** CVE Scanning** - Container image vulnerability scanning
- ** Secret Scanning** - Detects hardcoded secrets and credentials
- ** Config Audit** - Kubernetes misconfiguration detection
- ** RBAC Assessment** - Role-based access control analysis
- ** CIS Benchmark** - Cluster compliance according to CIS standards

##  Quick Install

```bash
helm install trivy-scanner ./trivy-scanner-chart \
  --namespace trivy-system \
  --create-namespace \
  --set discord.webhookUrl="YOUR_DISCORD_WEBHOOK_URL"
```

##  Documentation

- **[QUICKSTART.md](trivy-scanner-chart/QUICKSTART.md)** - Get started in 2 minutes with copy-paste examples
- **[README.md](trivy-scanner-chart/README.md)** - Complete technical reference and configuration options

##  What It Does

Runs scheduled security scans on your Kubernetes cluster and sends a compact, color-coded Discord notification with:
- Summary of all findings across 5 security domains
- Severity breakdown (Critical/High/Medium/Low)
- Trend analysis compared to previous scans
- Detailed JSON reports attached

##  Configuration Options

```yaml
# Minimal - only webhook required
discord:
  webhookUrl: "https://discord.com/api/webhooks/..."

# Optional - namespace-specific scanning
scanning:
  targetNamespace: "production"  # Scan only one namespace
  schedule: "0 9 * * 1"          # Monday 09:00 UTC

# Optional - resource limits
resources:
  limits:
    memory: "3Gi"
    cpu: "1000m"
```

##  Security

- Non-root container execution
- Minimal Linux capabilities (CHOWN, DAC_OVERRIDE, SETGID, SETUID)
- Read-only root filesystem support
- Automated RBAC with least-privilege principle

##  Default Schedule

Runs every **Monday at 09:00 UTC** (10:00 CET)

##  Namespace Support

- **All namespaces** (default): Scans entire cluster
- **Single namespace**: Set `scanning.targetNamespace` for focused scanning
- **Multiple namespaces**: Install multiple times with different release names

##  Requirements

- Kubernetes 1.20+
- Helm 3.0+
- Discord webhook URL
- Cluster-admin permissions (for installation)

##  Output Example

Discord notifications include:
- Inline fields for easy scanning (3 per row)
- Color-coded embeds (red for critical issues, green for clean)
- 5 attached JSON reports with full details
- Trend indicators (📈 increased, 📉 decreased, ➡️ stable)

##  Contributing

Issues and pull requests welcome!

##  License

MIT

##  Credits

Built with [Trivy](https://github.com/aquasecurity/trivy) by Aqua Security.
