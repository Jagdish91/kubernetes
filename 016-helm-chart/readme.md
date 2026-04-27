# Why Helm?

In Kubernetes, as your application evolves from a simple service to a production-grade system, the number of Kubernetes resources grows rapidly. You're not just dealing with a few Deployments anymore — you're managing:

- StatefulSets, ConfigMaps, Secrets
- Ingresses, PVCs, StorageClasses
- Network Policies, Autoscalers
- Even Custom Resources (CRs) used by tools like cert-manager, Prometheus, or Vault

Managing these manually across environments becomes increasingly unmanageable.

Even with Kustomize overlays, you're still dealing with raw YAMLs that must be copied, patched, and tracked. As application complexity increases, you need a better way to package, install, upgrade, rollback, and share your deployments.

## What Makes Helm Special (Compared to Kustomize)?

Kustomize can be used to manage per-environment overlays — like changing image tags, labels, or replica counts between dev, stage, and prod. It lets you organize your manifests and avoid copy-pasting YAMLs.

But Kustomize stops at configuration management — it doesn’t handle installation, upgrades, rollbacks, or packaging.

This is where Helm goes beyond:

- 🔁 Templating using Go templates (if, range, reusable helpers)
- 📦 Versioned packaging of applications into Helm Charts
- 📜 Release tracking and rollback support
- 🚀 Install, upgrade, delete operations via a single command
- 🌍 Chart repositories to distribute your application internally or externally
- 🔧 Hooks and tests for lifecycle events

Helm is not just about generating YAML — it’s a full application lifecycle manager for Kubernetes, much like how apt manages software packages on Debian.

## What is Helm?
Helm is a package manager for Kubernetes applications. Just like:

- apt is used to install packages on Debian-based Linux systems
- Homebrew is used on macOS
- Chocolatey is used on Windows

Helm is used to install, upgrade, rollback, uninstall, and distribute applications on Kubernetes.

You can use Helm to deploy:

- Your own custom applications  
- Third-party tools like Prometheus, Grafana, or Cert-Manager  
- Even Kubernetes Operators and controllers  

But Helm isn’t just a one-time install tool. It supports the full lifecycle of an application on Kubernetes — install,
uprade,
ollback,
delete — and even allows you to package your app into a Helm Chart,
which others can easily install with a single command.

In other words,
Helm for Kubernetes is what apt is for Debian — you can install,
remove,
and upgrade software using versioned packages,
with built-in support for values and configuration.

## Why Helm Needs to Understand Your Application 
Kubernetes doesn’t actually understand your application. To Kubernetes,
your app is just a collection of resources — Deployments,
Services,
ConfigMaps,
persistent volume claims (PVCs),
and Secrets. These are deployed and managed independently,
and Kubernetes doesn't know how they relate to one another.
This is where Helm adds intelligence.
Helm groups all your Kubernetes objects into a single logical unit called a **Helm Chart**. When you install that chart,
Helm tracks it as one cohesive **Release**, treating your entire app as a unified package.
Because of this grouping:

a) Helm understands your application as a whole—not just as individual YAMLs.
b) It can manage the entire lifecycle of the app:  
a. ✅ Install all components together  
b. 🔄 Upgrade them with new values or templates  
c. ⬅️ Rollback to a previous state if something breaks  
d. ❌ Uninstall cleanly and remove all related resources  
This application-level awareness makes Helm far more powerful than just running `kubectl apply -f`.
It’s not just about deploying YAMLs — it’s about managing your app’s full lifecycle with safety,
trepeatability,and control.

# Helm Core Concepts

Helm revolves around three key concepts:

## 1. Chart
A **Chart** is a Helm package. It contains everything you need to deploy an application, service, or tool on Kubernetes — including:

- Resource templates (Deployments, Services, ConfigMaps, etc.)
- A default configuration file (`values.yaml`)
- Chart metadata (`Chart.yaml`)
- Optional test hooks, helper templates, and dependencies

Think of a chart as a reusable, versioned blueprint of your app.

## 2. Repository
A **Repository** is a place where charts are stored and shared — similar to how `apt` downloads Debian packages from an APT repository.

- Helm repositories host Helm Charts (not packages).
- You can add multiple repos locally, search them, and install charts from them.
- When you create a Helm bundle for your app, it’s called a Helm Chart, not a Helm package.

When you want to install applications using Helm, the first thing you need is access to Helm Charts — these are versioned, packaged definitions of Kubernetes applications.

These charts are hosted on Helm Repositories. Some of the most popular public repositories include:

| Repository | Description |
| --- | --- |
| Bitnami | A trusted source with charts for Prometheus, Grafana, MySQL, NGINX, and many more |
| TrueCharts | Community-maintained charts, often used with TrueNAS and similar platforms |
| Others | Stakater, Grafana, Jetstack, and many more depending on your needs |

## 3. Release
A **Release** is a running instance of a Helm chart in your Kubernetes cluster. Every time you install a chart, Helm creates a release — and you provide a unique name for that release.

> 🔍 *Note:* The commands shown in this section are included for completeness. We will walk through each of these commands and their outputs in detail during the demo section of this lecture.

For example:
```bash
ghelm install app1-prod ./my-app-chart```
You’re installing the chart `my-app-chart`. The release name is `app1-prod`.

### Helm Tracks Release Revisions
Each release maintains a full revision history — tracking every install, upgrade, or rollback.

To view a release's revision history:
```bash
ghelm history app1-prod-test```
You’ll see a table of all revisions: their status, timestamps, and notes.

To rollback to a specific revision:
```bash
ghelm rollback app1-prod-test 2```
