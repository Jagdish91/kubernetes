# Kubernetes Multi-Container Monitoring Pod - Task 3

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

## 📁 File Structure

