# MyApp Helm Chart - Learning Guide

A minimal Helm chart for learning Helm and Kubernetes basics.

## Chart Structure

```
myapp-chart/
├── Chart.yaml          # Chart metadata (name, version, description)
├── values.yaml         # Default configuration values
├── templates/          # Kubernetes manifest templates (Go templating)
│   ├── _helpers.tpl    # Reusable template snippets
│   ├── deployment.yaml # Deployment manifest
│   ├── service.yaml    # Service manifest
│   ├── ingress.yaml    # Ingress (optional, enabled via values)
│   └── NOTES.txt       # Post-install instructions
└── README.md
```

## Quick Start

```bash
# Validate the chart
helm lint myapp-chart

# Dry-run to see rendered manifests (without installing)
helm install myapp myapp-chart --dry-run --debug

# Install the chart
helm install myapp myapp-chart

# List releases
helm list

# Upgrade with new values
helm upgrade myapp myapp-chart --set replicaCount=3

# Uninstall
helm uninstall myapp
```

## Overriding Values

**Method 1: Command line**
```bash
helm install myapp myapp-chart --set replicaCount=2 --set image.tag=1.26
```

**Method 2: Values file**
```bash
# Create custom-values.yaml with your overrides, then:
helm install myapp myapp-chart -f custom-values.yaml
```

## Key Concepts to Explore

1. **`.Values`** - Access values from `values.yaml` (e.g. `{{ .Values.replicaCount }}`)
2. **`.Chart`** - Chart metadata from `Chart.yaml` (e.g. `{{ .Chart.Name }}`)
3. **`.Release`** - Release info (e.g. `{{ .Release.Name }}` - the name you gave when installing)
4. **Helpers** - `{{ include "myapp-chart.fullname" . }}` reuses logic from `_helpers.tpl`
5. **Conditionals** - `{{- if .Values.ingress.enabled }}` in ingress.yaml

## Useful Commands

| Command | Purpose |
|---------|---------|
| `helm lint` | Check chart for issues |
| `helm template` | Render templates locally |
| `helm get values myapp` | Show values used for a release |
| `helm history myapp` | Show upgrade history |
