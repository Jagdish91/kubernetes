# Why Kustomize?

In real-world Kubernetes deployments, we often deal with multiple environments like development, staging, and production. While the application itself remains the same, environment-specific values such as image versions, replica counts, labels, and annotations tend to differ.

Let’s take a familiar example — the nginx deployment. In many organizations:

- Dev might use a newer version like `nginx:1.22` or even the latest build for testing purposes.
- Staging and Prod generally run a more stable and vetted version like `nginx:1.20`, often identical to ensure consistency.

This parallels how infrastructure environments are managed — you might test newer OS or runtime versions in dev before pushing them to prod.

## What is Kustomize?

Kustomize is a declarative configuration management tool for Kubernetes that allows you to customize raw, template-free YAML files for multiple environments. Instead of using variables or a separate templating language (like in Helm), Kustomize applies overlays and patches to your base configuration — all in pure YAML.

Kustomize works natively with `kubectl` and supports:

- Applying overrides like replica count, image tags, labels, etc.
- Managing variants of Kubernetes objects without duplicating entire YAML files
- Keeping configurations clean, modular, and DRY

With Kustomize, your Kubernetes manifests are:

- Fully declarative
- Structured as base + overlays
- Compatible with existing tooling (`kubectl`, GitOps, CI/CD pipelines)

Kustomize is ideal for teams that want to maintain multiple environment-specific configurations in a scalable, native way — without introducing a templating engine.

## Example: NGINX Deployment Across Dev, Staging, and Prod
Let’s say we want to deploy nginx across three environments with the following variations:

| Environment | Image Tag   | Replicas | Label     |
|-------------|--------------|----------|-----------|
| Dev         | nginx:1.22   | 2        | env: dev |
| Staging     | nginx:1.20   | 3        | env: staging |
| Production  | nginx:1.20   | 5        | env: prod |

## What We Do with Kustomize
The base folder contains the generic YAML manifests: Deployment, Service, etc.
Each overlay (dev, staging, prod) contains a small patch to modify specific fields like replicas, image, or metadata.labels.


# Kustomize Commands Must Run in a Directory with `kustomization.yaml`

Kustomize operates on directories — not individual YAML files. This is because its entire processing model revolves around a file named `kustomization.yaml` which serves as the entry point for defining how resources should be composed or transformed.

## Why This Matters
Every time you use a kustomize-related command — whether via the standalone kustomize CLI or `kubectl kustomize` — it expects to find a `kustomization.yaml` file in the current directory or in the directory you explicitly reference. This file tells Kustomize what resources to pull in and how to transform them.

## What Are Transformers in Kustomize?
In Kustomize, transformers are built-in mechanisms that let you modify Kubernetes resources declaratively — without having to edit the raw YAML files. These are customization directives you define in your `kustomization.yaml` that apply consistent changes across all resources.

Transformers work by targeting well-known fields in Kubernetes manifests and updating them based on the configuration you provide.

## Examples of Transformers
Here are some common transformers you’ll encounter:
- **namePrefix** and **nameSuffix**: Add prefixes or suffixes to resource names (e.g., nginx-deploy becomes dev-nginx-deploy)
- **namespace**: Assigns a namespace to all resources
- **images**: Override container image names and tags
- **replicas**: Change the number of replicas for Deployments or StatefulSets
- **labels and annotations**: Add key-value metadata to resources or selectors
- **commonAnnotations** and **commonLabels**: Apply annotations/labels to all resources uniformly
- **resources**: List the base files or directories containing Kubernetes manifests

## When Transformers Fall Short
Kustomize provides several built-in transformers — like namePrefix, namespace, images, replicas, labels, annotations, and more — to customize Kubernetes resources in a structured way. These are easy to use and declarative, but they have limitations.

### What Transformers Cannot Do
Transformers are great for high-level field substitutions, but they cannot modify arbitrary nested fields inside your manifests. For example:
- You cannot add or change the `resources` block under a container using a transformer
- You cannot modify `containerPort` or add ports inside the container spec
- You cannot remove a field or label that exists in the base manifest
- You cannot inject environment variables into containers unless using `envs:` with external files (which is limited)
- You cannot add volume mounts or change probes (`liveness/readiness`)
- You cannot touch nested fields within `spec.template.spec` beyond what’s exposed by specific transformers
These are deeper, structural changes that require fine-grained control.

## Enter Patching
For these kinds of modifications, patching is the recommended approach. Patches allow you to surgically modify any field in any resource — including deeply nested ones — using standardized mechanisms like RFC 6902 JSON Patch or Strategic Merge.
You are not limited to either-or — you can (and often do) use transformers and patches together in your overlays. For example:
- Use transformers to apply a namespace, label, and image version
- Use patches to inject an environment variable or hardcode a nodePort
This hybrid approach gives you both structure and flexibility — which is essential for large-scale Kubernetes deployments.

## What Are Patches?
Patches are used to make precise, field-level modifications to a Kubernetes object — especially for deeply nested fields or configurations not supported by transformers.
Patches are helpful when:
- The field you want to change is inside nested structures (e.g., container specs)
- The modification is highly specific (e.g., change a single port, env var, or resource limit)
- You want fine-grained control over a specific object
### Categories of Patches in Kustomize
type | Format | Merge Strategy | When to Use 
---|---|---|---
pachesStrategicMerge | YAML | Strategic Merge | Best for typical workloads like Deployment, Service, etc.
pachesJson6902 | JSON patch | RFC 6902 operations | When strategic merge fails or you need precise edits 
paches (generic form) | YAML/JSON | With target field | For complex use cases and fine-grained matching
