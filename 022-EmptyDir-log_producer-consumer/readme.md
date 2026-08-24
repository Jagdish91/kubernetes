# Kubernetes Multi-Container Pod with Shared Storage - Log Demo

A hands-on Kubernetes project demonstrating **ephemeral storage (emptyDir)**, **multi-container pods**, **volume sharing**, and **container health probes**.

## 🎯 Project Overview

This project implements a real-world scenario where two containers in the same Kubernetes Pod share temporary storage to process logs:

- **log-producer**: Writes application logs to a shared volume every 10 seconds
- **log-aggregator**: Reads logs from the shared volume, counts entries, and writes summaries every 30 seconds

This demonstrates core Kubernetes concepts essential for DevOps and Cloud Engineers.

## 📚 Learning Objectives

After working through this project, you will understand:

✅ **emptyDir volumes** — temporary storage that lives only as long as the Pod  
✅ **Multi-container Pods** — designing Pods with multiple app containers  
✅ **Volume mounts** — sharing storage between containers  
✅ **Container startup coordination** — ensuring dependent containers wait properly  
✅ **Startup probes** — health checks during container initialization  
✅ **kubectl debugging** — examining logs and verifying shared state  

## 🏗️ Architecture

