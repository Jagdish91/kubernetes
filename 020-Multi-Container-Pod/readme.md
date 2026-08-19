# Multi-Container Pod Kubernetes Project

A comprehensive collection of multi-container pod configurations showcasing essential patterns for Kubernetes CKA certification and production deployment.

## Project Overview

This repository contains three progressively complex multi-container pod configurations demonstrating fundamental Kubernetes orchestration patterns:

1. **Basic Sidecar Pattern** - Simple health monitoring
2. **Intermediate Init Containers** - Sequential dependency checks  
3. **Advanced Production Pattern** - Full monitoring with init containers + sidecars
 
## Files Structure

| File | Description | Difficulty |
|------|-------------|-------------|
| [`task1.yaml`](#task-1-basic-sidecar) | Basic health monitoring sidecar | Beginner |
| [`task2.yaml`](#task-2-intermediate-init-containers) | Sequential dependency checking init containers | Intermediate |
| [`security-monitor.yaml`](#task-3-advanced-production-pattern) | Production-ready monitoring with readiness probes | Advanced |

## Detailed Breakdown

### Task 1: Basic Sidecar
**Pattern**: Sidecar for continuous health monitoring
**Concepts**: `while` loops, `restartPolicy`, container communication
```yaml
# Key features:
# - Main app: nginx on port 80
# - Sidecar: curl container checking health every 5 seconds
# - restartPolicy: Never (one-time execution)
