# Trivy Scanner Helm Chart

## Snabbstart

### Installation

**Minimal installation (endast webhook URL krävs):**

*Bash/Linux/macOS:*
```bash
helm install trivy-scanner ./trivy-scanner-chart \
  --namespace trivy-system \
  --create-namespace \
  --set discord.webhookUrl="https://discord.com/api/webhooks/YOUR_WEBHOOK_ID/YOUR_TOKEN"
```

*PowerShell/Windows:*
```powershell
helm install trivy-scanner ./trivy-scanner-chart `
  --namespace trivy-system `
  --create-namespace `
  --set discord.webhookUrl="https://discord.com/api/webhooks/YOUR_WEBHOOK_ID/YOUR_TOKEN"
```

Allt annat använder defaults (schema: måndag 09:00, scannar hela klustret).

---

## Konfiguration

### Skapa din egen `my-values.yaml`:

```yaml
# =============================================================================
# MINIMAL CONFIGURATION - Endast webhook URL krävs
# =============================================================================
discord:
  webhookUrl: "https://discord.com/api/webhooks/1234567890/abc123xyz"

# =============================================================================
# OPTIONAL: Avancerad konfiguration
# =============================================================================

# Scanna endast ett specifikt namespace (istället för hela klustret)
# scanning:
#   targetNamespace: "production"  # Endast production
#   schedule: "0 8 * * 1"          # Måndag kl 08:00 istället
```

**Installera med din config:**
```bash
helm install trivy-scanner ./trivy-scanner-chart \
  --namespace trivy-system \
  --create-namespace \
  --values my-values.yaml
```

---

## Exempel-användning

### 1. Scanna hela klustret (default)
```bash
helm install trivy-scanner ./trivy-scanner-chart \
  --set discord.webhookUrl="https://..."
```

### 2. Scanna endast ett namespace
```bash
helm install trivy-scanner ./trivy-scanner-chart \
  --set discord.webhookUrl="https://..." \
  --set scanning.targetNamespace="production"
```

### 3. Ändra schema
```bash
helm install trivy-scanner ./trivy-scanner-chart \
  --set discord.webhookUrl="https://..." \
  --set scanning.schedule="0 8 * * 1"  # Måndag 08:00
```

### 4. Minska resurser
```bash
helm install trivy-scanner ./trivy-scanner-chart \
  --set discord.webhookUrl="https://..." \
  --set resources.limits.memory="2Gi"
```

---

## Uppgradering

```bash
# Uppgradera till ny version (behåller webhook URL)
helm upgrade trivy-scanner ./trivy-scanner-chart \
  --reuse-values

# Uppgradera och ändra schedule
helm upgrade trivy-scanner ./trivy-scanner-chart \
  --reuse-values \
  --set scanning.schedule="0 10 * * 1"
```

---

## Rollback

```bash
# Lista historik
helm history trivy-scanner

# Rollback till förra versionen
helm rollback trivy-scanner
```

---

## Avinstallation

```bash
helm uninstall trivy-scanner --namespace trivy-system
```

---

## Alla tillgängliga värden

Se `values.yaml` för fullständig lista. Endast **discord.webhookUrl** är obligatorisk!

**Defaults:**
- Schedule: Måndag 09:00 UTC
- Target: Hela klustret
- Memory: 3Gi max
- CPU: 1000m max
- RBAC: Skapas automatiskt
- Trivy: latest version

---

## Manuell testning

```bash
# Trigga scan manuellt med unikt namn och följ loggar
$jobName = "trivy-manual-$(Get-Date -Format 'yyyyMMdd-HHmm')"
kubectl create job --from=cronjob/trivy-combined-scan $jobName -n trivy-system
Start-Sleep -Seconds 10
kubectl logs -f -n trivy-system -l job-name=$jobName
```

---

## FAQ

**Q: Hur får jag webhook URL?**  
A: Discord → Server Settings → Integrations → Webhooks → New Webhook

**Q: Kan jag scanna flera namespaces?**  
A: Nej, antingen hela klustret eller ETT namespace. För flera, installera flera charts med olika namn:
```bash
helm install trivy-prod ./trivy-scanner-chart --set scanning.targetNamespace="production"
helm install trivy-staging ./trivy-scanner-chart --set scanning.targetNamespace="staging"
```

**Q: Varför behöver Trivy root-rättigheter?**  
A: För att installera scanning-verktyg (curl, kubectl, jq) via `apk add`.

**Q: Fungerar det med Minikube/K3s?**  
A: Ja! CIS Benchmark kan ge "N/A" i dev-miljöer, men övriga scanners fungerar.
