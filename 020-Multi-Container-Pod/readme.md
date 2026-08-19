# Kubernetes Multi-Container Monitoring Pod 

A production-ready Kubernetes multi-container pod configuration demonstrating advanced orchestration patterns for the CKA certification journey.

## 📋 Overview

This YAML configuration creates a sophisticated monitoring pod with:
- **Init Container** for pre-flight dependency checking
- **Multi-Container Sidecar Pattern** for continuous health monitoring  
- **Production Features** including readiness probes, resource limits, and structured logging

## 🚀 Features

### Init Container (`pre-flight`)
- ✅ Verifies external API (`https://kubernetes.io`) reachability
- ✅ Structured logging with attempt counter
- ✅ Retry logic with configurable intervals
- ✅ Graceful timeout handling

### Main Containers

#### `web-server` (Primary App)
- ✅ Nginx web server with custom readiness probe
- ✅ Resource constraints: CPU 250m, Memory 128Mi
- ✅ Proper port configuration (port 80)
- ✅ Health check via HTTP GET probe

#### `traffic-analyzer` (Monitoring Sidecar)
- ✅ Continuous health monitoring every 6 seconds
- ✅ Real-time status logging (responding/not responding)
- ✅ Shared localhost networking pattern
- ✅ Proper resource limits matching primary container

### Production Features
- ✅ Kubernetes namespace isolation (`prod-security`)
- ✅ Production labels (`role=monitoring`, `critical=yes`)
- ✅ Resource management with limits and requests
- ✅ Restart policy: `OnFailure` for production resilience

## 🔧 Prerequisites

- Kubernetes cluster (Minikube, Kind, EKS, AKS, GKE)
- `kubectl` configured with cluster access
- Namespace `prod-security` created

## 🚀 Deployment

1. **Create namespace** (if not exists):
   ```bash
   kubectl create namespace prod-security

# Init container logs
kubectl logs security-monitor -c pre-flight -n prod-security

# Web server logs
kubectl logs security-monitor -c web-server -n prod-security  

# Traffic analyzer logs (monitoring sidecar)
kubectl logs security-monitor -c traffic-analyzer -n prod-security


🎯 Key Learning Points
----------------------

### 1\. Init Containers Pattern

Init containers run to completion BEFORE main containers start, perfect for:

*   Dependency checking (databases, APIs, services)
    
*   Data initialization
    
*   Security validations
    

### 2\. Sidecar Monitoring Pattern

The traffic-analyzer demonstrates classic sidecar usage:

*   Shared network namespace (localhost)
    
*   Non-blocking, continuous health checks
    
*   Separation of concerns (app vs monitoring)

### 3\. Production Readiness

yaml

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   readinessProbe:  httpGet:    path: /    port: 80  periodSeconds: 5   `

*   Ensures traffic only routes to healthy pods
    
*   Crucial for zero-downtime deployments
    
*   Part of Kubernetes' self-healing capabilities
    

### 4\. Resource Management
---yaml

Plain textANTLR4BashCC#CSSCoffeeScriptCMakeDartDjangoDockerEJSErlangGitGoGraphQLGroovyHTMLJavaJavaScriptJSONJSXKotlinLaTeXLessLuaMakefileMarkdownMATLABMarkupObjective-CPerlPHPPowerShell.propertiesProtocol BuffersPythonRRubySass (Sass)Sass (Scss)SchemeSQLShellSwiftSVGTSXTypeScriptWebAssemblyYAMLXML`   resources:  limits:    cpu: "250m"     # 0.25 CPU cores    memory: "128Mi" # 128 Mebibytes (NOT "128m"!)   `

*   Prevents resource starvation
    
*   Enables fair scheduling
    
*   Essential for multi-tenant clusters

