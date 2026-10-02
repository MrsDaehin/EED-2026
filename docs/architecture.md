# System Architecture: EDD 2026

This document details the architectural layout for the EDD 2026 demonstration environment. The system models a resilient microservices application subjected to automated performance testing and deliberate fault injection.

---

## 1. System Infrastructure Overview

The entire environment runs locally inside a virtualized **3-Node Minikube Cluster**. The architecture isolates the target application domain from the testing control planes, while allowing **Chaos Mesh** to orchestrate targeted disruptions directly across the node boundaries.

```
+------------------------------------------------------------------------------------+
| MINIKUBE CLUSTER (3-Node Topology: minikube, minikube-m02, minikube-m03)           |
|                                                                                    |
|  =========================== APPLICATION NAMESPACE =============================   |
|  [Node 1 / 2 / 3 Distribution]                                                     |
|                                                                                    |
|       +-------------------+       +-------------------+       +-----------------+  |
|       |   ProductPage     +------>|     Details       |       |     Reviews     |  |
|       |   (Frontend App)  |       |   (Backend App)   |       |  (Backend App)  |  |
|       +---------^---------+       +-------------------+       +-----------------+  |
|                 |                                                                  |
|  ===============|================= TESTING WORKSPACES ============================  |
|                 |                                                                  |
|   +-------------+-----------+                             +--------------------+   |
|   |    k6 Load Injector     |                             |     Chaos Mesh     |   |
|   | (Synthetic Traffic Gen) |                             | (CRD Orchestrator) |   |
|   +-------------------------+                             +---------+----------+   |
|                                                                     |              |
|                                    Injects Faults (Pod/Network/JVM) |              |
|                                    v v v v v v v v v v v v v v v v  v              |
+------------------------------------------------------------------------------------+
```

---

## 2. Core Architectural Components

### 2.1 Target Applications (`kubernetes-sample-apps`)
The application plane consists of 3 specific services inspired by the **BookInfo** microservices ecosystem, deployed declaratively via the configurations located in `/kubernetes-sample-apps`:

* **ProductPage App:** The entry-point service. It orchestrates the user interface by calling the downstream metadata and evaluation services to build the final web display.
* **Details App:** A backend service that manages and surfaces structural book metadata (e.g., pages, publisher, year).
* **Reviews App:** A backend service responsible for retrieving and formatting book reviews and ratings.

### 2.2 Load Generation Plane
* **k6 Injector:** A dedicated runner executing script-driven scenario configurations. It targets the public-facing endpoint of the **ProductPage App** to establish a baseline traffic pattern, measuring key metrics like request duration, error rates, and HTTP exceptions during steady and un-steady states.

### 2.3 Fault Injection Plane
* **Chaos Mesh:** A cloud-native Chaos Engineering platform installed cluster-wide. It operates via Custom Resource Definitions (CRDs) to systematically intercept and disrupt the BookInfo components. 

---

## 3. Lifecycle of a Chaos Experiment

The demonstration maps out a repeatable loop of validation under duress, executing across the cluster nodes in a distinct sequence:

1. **Establish Steady State (k6 Routine Init):** The k6 injector initiates sustained synthetic user traffic targeting the ProductPage app, mapping out initial latency and throughput baselines.
2. **Deploy Chaos Manifesto (Chaos Mesh Apply):** A targeted YAML configuration (e.g., PodChaos or NetworkChaos) is applied directly to the cluster via `kubectl`.
3. **Disruption Injection (Cluster Interception):** Chaos Mesh agents modify node network routing or terminate targeted application pods (like the Reviews or Details apps) across the Minikube workers.
4. **Resilience Evaluation (k6 Verification):** The k6 injector captures the instant error blast-radius, measuring if the frontend handles downstream app failure gracefully or crashes cascadingly.

---

## 4. Key Deployment Configurations

### 4.1 Multi-Node Scheduling Strategy
Because Minikube is provisioned explicitly with **3 nodes**, application workloads and Chaos Mesh agent daemons are distributed across distinct virtual boundaries (`minikube`, `minikube-m02`, and `minikube-m03`). This enables realistic testing of cross-node network latency and structural node partition failures.

### 4.2 Blast Radius Controls
To ensure safety during live presentations, Chaos Mesh experiments are explicitly scoped using Kubernetes namespace selectors. The fault injectors are strictly limited to hitting pods matching labels inside the `kubernetes-sample-apps` execution space.