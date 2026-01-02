# Trivy Scanner - Helm Chart Quick Start

## TL;DR - Installera på 2 minuter

**Bash/Linux/macOS:**
```bash
# 1. Skapa en config-fil med din webhook URL
cat > my-values.yaml <<EOF
discord:
  webhookUrl: "https://discord.com/api/webhooks/DIN_WEBHOOK_ID/DIN_TOKEN"
EOF

# 2. Installera
helm install trivy-scanner ./trivy-scanner-chart \
  --namespace trivy-system \
  --create-namespace \
  --values my-values.yaml
```

**PowerShell/Windows:**
```powershell
# 1. Skapa en config-fil med din webhook URL
@"
discord:
  webhookUrl: "https://discord.com/api/webhooks/DIN_WEBHOOK_ID/DIN_TOKEN"
"@ | Out-File -FilePath my-values.yaml -Encoding utf8

# 2. Installera
helm install trivy-scanner ./trivy-scanner-chart `
  --namespace trivy-system `
  --create-namespace `
  --values my-values.yaml
```

**Testa installation (fungerar i både bash och PowerShell):**
```powershell
# 3. Testa (körs direkt, väntar inte på schema)
$jobName = "trivy-manual-$(Get-Date -Format 'yyyyMMdd-HHmm')"
kubectl create job --from=cronjob/trivy-combined-scan $jobName -n trivy-system
Start-Sleep -Seconds 10
kubectl logs -f -n trivy-system -l job-name=$jobName
```

**Det är allt!** 🎉 Scanningen körs varje måndag kl 09:00 UTC automatiskt.

---

## Vad händer vid installation?

Helm skapar automatiskt:
- ✅ Namespace `trivy-system`
- ✅ ServiceAccount med rätt RBAC-permissions
- ✅ Secret med din Discord webhook URL
- ✅ CronJob som kör varje måndag 09:00 UTC
- ✅ 5 ConfigMaps för trend-tracking (skapas vid första körningen)

**Du behöver INTE:**
- ❌ Skapa namespace manuellt
- ❌ Konfigurera RBAC manuellt
- ❌ Editera YAML-filer
- ❌ Köra flera `kubectl apply`

---

## Vanliga scenarion

### Scenario 1: "Jag vill bara testa"
```bash
# Minimal installation
helm install trivy-scanner ./trivy-scanner-chart \
  --namespace trivy-system \
  --create-namespace \
  --set discord.webhookUrl="https://..."
  
# Trigga direkt med unikt namn
$jobName = "trivy-test-$(Get-Date -Format 'HHmmss')"
kubectl create job --from=cronjob/trivy-combined-scan $jobName -n trivy-system
Start-Sleep -Seconds 10
kubectl logs -f -n trivy-system -l job-name=$jobName
```

### Scenario 2: "Jag vill bara scanna production namespace"
```bash
# Skapa config
cat > my-values.yaml <<EOF
discord:
  webhookUrl: "https://..."
scanning:
  targetNamespace: "production"  # <-- Detta är nyckeln
EOF

helm install trivy-scanner ./trivy-scanner-chart -f my-values.yaml
```

### Scenario 3: "Jag vill scanna flera namespaces"
```bash
# Installation 1: Production (måndag 08:00)
helm install trivy-prod ./trivy-scanner-chart \
  --set discord.webhookUrl="https://..." \
  --set scanning.targetNamespace="production" \
  --set scanning.schedule="0 8 * * 1"

# Installation 2: Staging (onsdag 10:00)
helm install trivy-staging ./trivy-scanner-chart \
  --set discord.webhookUrl="https://..." \
  --set scanning.targetNamespace="staging" \
  --set scanning.schedule="0 10 * * 3"
```

### Scenario 4: "Mitt kluster har begränsat minne"
```yaml
# my-values.yaml
discord:
  webhookUrl: "https://..."
resources:
  limits:
    memory: "1.5Gi"  # Default är 3Gi
    cpu: "500m"
```

---

## Namespace-scanning förklarat

**Default (ingen targetNamespace):**
- Scannar alla pods i alla namespaces
- Scannar alla configs i alla namespaces
- Scannar alla secrets i alla namespaces
- CIS Benchmark (alltid cluster-wide)

**Med targetNamespace="production":**
- ✅ Scannar ENDAST pods i `production` namespace
- ✅ Scannar ENDAST configs i `production` namespace
- ✅ Scannar ENDAST secrets i `production` namespace
- ℹ️ CIS Benchmark (fortfarande cluster-wide, men det är OK)

**Varför är det användbart?**
- Börja smått - få ett överskådligt resultat
- Isolera kritiska namespaces
- Undvik överväldigande antal vulnerabilities första gången
- Scanna dev/test/prod separat med olika scheman

---

## Uppgradering

```bash
# Uppgradera till ny version (behåller alla settings)
helm upgrade trivy-scanner ./trivy-scanner-chart --reuse-values

# Ändra schedule
helm upgrade trivy-scanner ./trivy-scanner-chart \
  --reuse-values \
  --set scanning.schedule="0 10 * * 1"

# Byt från "hela klustret" till "endast production"
helm upgrade trivy-scanner ./trivy-scanner-chart \
  --reuse-values \
  --set scanning.targetNamespace="production"
```

---

## Felsökning

**"Inget händer":**
```bash
# Kontrollera CronJob
kubectl get cronjob -n trivy-system

# Trigga manuellt med unikt namn
$jobName = "trivy-debug-$(Get-Date -Format 'HHmmss')"
kubectl create job --from=cronjob/trivy-combined-scan $jobName -n trivy-system
Start-Sleep -Seconds 10
kubectl logs -f -n trivy-system -l job-name=$jobName
```

**"OOMKilled":**
```bash
# Öka memory
helm upgrade trivy-scanner ./trivy-scanner-chart \
  --reuse-values \
  --set resources.limits.memory="4Gi"
```

**"Inget Discord-meddelande":**
```bash
# Testa webhook direkt
kubectl get secret discord-webhook -n trivy-system -o jsonpath='{.data.url}' | base64 -d
# Kopiera URL och testa i webbläsare eller curl
```

---

## Avinstallation

```bash
helm uninstall trivy-scanner --namespace trivy-system

# Radera även namespace om du vill
kubectl delete namespace trivy-system
```

---

## Alla tillgängliga konfigurationer

Se `values.yaml` för fullständig lista. Här är de viktigaste:

| Parameter | Beskrivning | Default |
|-----------|-------------|---------|
| `discord.webhookUrl` | **REQUIRED** Discord webhook URL | - |
| `scanning.targetNamespace` | Specifikt namespace (tom = alla) | `""` |
| `scanning.schedule` | Cron schema | `"0 9 * * 1"` |
| `scanning.timeout` | Timeout per scanning-steg | `"10m"` |
| `resources.limits.memory` | Max memory | `"3Gi"` |
| `resources.limits.cpu` | Max CPU | `"1000m"` |
| `rbac.create` | Skapa RBAC automatiskt | `true` |

---

## Hjälp och Support

- 📖 Fullständig guide: Se `INSTALLATION.md` i `trivy-deployment-package/`
- 📋 Exempel: Se `values-example.yaml`
- 🐛 Issues: GitHub Issues
- 📝 Trivy docs: https://aquasecurity.github.io/trivy/
