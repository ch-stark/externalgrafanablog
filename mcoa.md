# The Next Era of Fleet Observability: Introducing the MultiCluster Observability Addon (MCOA)

Observability is evolving in Red Hat Advanced Cluster Management for Kubernetes (RHACM). To meet the growing demands of hybrid cloud environments and edge deployments, RHACM has introduced the MultiCluster Observability Addon (MCOA). MCOA is a new component designed to orchestrate OpenShift Observability across a massive fleet of managed clusters (spokes), automating the collection and forwarding of metrics, logs, and traces to central stores.

While MCOA first appeared as a preview, its metrics collection capabilities reached General Availability (GA) in RHACM 2.15, bringing a fundamental architectural shift to how fleets are monitored.

## The Architectural Shift: Moving to Upstream Standards

Previously, RHACM observability relied on custom workloads on the managed clusters, specifically the endpoint operator and the metrics collector. MCOA replaces these custom solutions with established, open-source upstream alternatives: the Prometheus Agent and the Prometheus Operator. This shift means your fleet observability is now powered by standard community-driven tools rather than proprietary components, delivering stronger performance and a smoother experience.

## Core Benefits of MCOA

**GitOps-Native Configurability:** The legacy method of configuring metrics via a single custom allowlist ConfigMap has been deprecated. Instead, MCOA utilizes standard upstream Kubernetes APIs like ScrapeConfig, PrometheusRule, and PrometheusAgent Custom Resources. This gives you direct control over how metrics are collected using standard APIs that fit seamlessly into modern CI/CD and GitOps pipelines.

**Granular Fleet Management:** MCOA fully integrates with the Open Cluster Management (OCM) Placement API and the ClusterManagementAddon. This enables platform teams to apply fine-grained observability configurations to specific subsets of clusters (like edge nodes vs. core data centers) based on their unique resource needs.

**Scalability via Sharded Federation:** MCOA drastically increases metric federation scalability by using distinct ScrapeConfigs. Each configuration is independently federated from the in-cluster Prometheus, reducing the strain on the system and preventing bottlenecks as metric cardinality grows across your fleet.

**Edge Resiliency and Remote Write:** Utilizing a highly performant, standard remote write implementation, MCOA improves payload management when sending metrics to the hub. Crucially for edge deployments, it offers robust network partition resiliency. The Prometheus Agent buffers data locally, protecting against network outages between the spoke and hub for up to two hours without data loss.

## Important Changes to Alert Routing

A significant behavioral change to be aware of is that alert forwarding from spoke clusters to the Hub Alertmanager is no longer configured automatically. MCOA requires users to make an intentional choice about alert routing to prevent unwanted noise and conflicts. Forwarding can easily be enabled fleet-wide by deploying a standard RHACM Configuration Policy that injects the routing configurations into your clusters.

## Migrating to MCOA

Transitioning your metrics stack to MCOA is designed to be straightforward and conflict-free. When you enable the MCOA metrics capabilities in your MultiClusterObservability custom resource on the hub (by setting `capabilities.platform.metrics.default.enabled: true`), the system automatically removes the legacy endpoint operators and metrics collectors.

If you rely on custom metrics, you do not need to rewrite your configurations manually. The Allowlist Migration Tool CLI (`allowlist-migration`) parses your old `observability-metrics-custom-allowlist` ConfigMap and automatically generates the equivalent MCOA-compliant ScrapeConfig and PrometheusRule manifests for you to deploy.

## Summary
MCOA represents a strategic leap forward, turning isolated cluster monitoring into a unified, resilient, and highly configurable intelligence engine that maximizes the return on your hybrid cloud investments.
MCOA represents a strategic leap forward, turning isolated cluster monitoring into a unified, resilient, and highly configurable intelligence engine that maximizes the return on your hybrid cloud investments.
