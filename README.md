# Chaos Engineering as Code with k6 and Chaos Mesh

> "Treat Failure as a Feature. Hope is not an strategy"

Welcome to the companion repository for the Extremadura Digital Day 2026 session.

Modern cloud-native systems are expected to survive failures. Pods crash. Nodes disappear. Networks become unreliable. Dependencies slow down.

Yet most teams still validate their systems only under normal operating conditions.

In this session, we explore a different approach:

**What if resilience testing became part of your automated testing strategy?**

Instead of treating Chaos Engineering as a separate activity, we will use:

- k6 to generate traffic and validate user experience
- xk6-kubernetes to interact with Kubernetes
- Chaos Mesh to inject failures
- SLOs and thresholds to verify resilience

The result is a fully automated experiment where load generation, fault injection, and validation happen from the same test suite.

---

## The Story

Imagine a typical production service. Everything looks healthy:

- Dashboards are green
- Deployments are successful
- Response times are acceptable

But there is one important question nobody has answered:

> What happens when something goes wrong?

Will users notice? Will the application recover? Will our SLOs still be respected?

The only way to know is to test failure intentionally. That is where Chaos Engineering begins.

---

## From Load Testing to Resilience Testing

Traditional performance testing asks:

> How fast is the system?

Chaos Engineering asks:

> How does the system behave when things break?

In this workshop we combine both perspectives. We will:

1. Generate load with k6
2. Establish a steady state
3. Inject failures with Chaos Mesh
4. Observe the impact
5. Validate resilience using automated assertions

This transforms a performance test into a resilience test.

---

## Architecture

The demo environment consists of:

- Kubernetes cluster (Minikube)
- Sample Kubernetes applications
- Chaos Mesh
- Prometheus
- Grafana
- k6 with xk6-kubernetes

```text
                   +----------------+
                   |       k6       |
                   | Load + Chaos   |
                   +--------+-------+
                            |
                            |
                            v
--------------------------------------------------
|                 Kubernetes Cluster               |
|                                                  |
|  +-----------+      +-----------+                |
|  | Service A | ---> | Service B |                |
|  +-----------+      +-----------+                |
|                                                  |
|         Chaos Mesh injects failures              |
--------------------------------------------------
```

---

## Chaos as Code

A key idea of this session is that chaos experiments should be versioned and reproducible. Instead of clicking buttons in a UI, experiments are defined as code.

Example:

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: network-delay
spec:
  action: delay
```

These manifests live in source control and can be executed repeatedly across environments.

---

## Repository Structure

```text
.
├── docs/
├── experiments/
├── k6/
├── kubernetes-sample-apps/
├── minikube/
├── scripts/
└── README.md
```

### `docs/`
Workshop documentation, architecture notes, and the chaos hypothesis.

### `experiments/`
Chaos Engineering experiments and supporting artifacts for resilience testing.

### `k6/`
Performance and resilience test scripts, including load, resource stress, and pod disruption scenarios.

### `kubernetes-sample-apps/`
Sample Kubernetes applications and manifests for demo deployments.

### `minikube/`
Local Minikube cluster setup and teardown scripts.

### `scripts/`
Helper scripts for installing Chaos Mesh, configuring Helm, and working with k6.

---

## Experiments

### Experiment 1: Network Latency

**Question**

Can our service tolerate additional latency?

**Chaos**

- Inject network delay

**Validation**

- Response time remains below SLO

---

### Experiment 2: Pod Failure

**Question**

Can Kubernetes recover automatically?

**Chaos**

- Inject pod failures

**Validation**

- Error rate remains acceptable

---

### Experiment 3: CPU Stress

**Question**

How does the application behave under resource pressure?

**Chaos**

- Saturate CPU resources

**Validation**

- User experience remains acceptable

---

### Experiment 4: Chaos Workflow

**Question**

What happens when multiple failures occur together?

**Chaos**

- Network delay
- Pod failure
- Resource stress

**Validation**

- System continues to satisfy resilience objectives

---

## Running the Workshop

### Create Cluster

```bash
./minikube/00-create_cluster.sh
```

### Install Chaos Mesh

```bash
./scripts/03_add_helm_repo.sh
./scripts/04_helm_install_chaos_mesh.sh
```

### Deploy Demo Application

```bash
kubectl apply -k kubernetes-sample-apps/bookinfo-example/kustomize
```

### Run Baseline Test

```
while true; do curl -I http://localhost:50768; sleep 1; done
```

```bash
k6 run k6/basic-load.js
```

### Run Chaos Experiment

```bash
k6 run k6/pod-kill.js
```

---

## Success Criteria

The goal is not to avoid failure. The goal is to understand failure.

A successful experiment is one that teaches us something about our system. Whether the experiment passes or fails, we learn. And every lesson makes the system more resilient.

---

## Key Takeaways

By the end of this session you will know how to:

- Use Chaos Mesh on Kubernetes
- Drive experiments from k6
- Automate Chaos Engineering workflows
- Validate resilience using SLOs
- Treat chaos experiments as code
- Integrate resilience testing into your delivery pipelines


---

## Resources

- Chaos Mesh: https://chaos-mesh.org
- Chaos Mesh Playground: https://github.com/superorbital/chaos-mesh-playground
- k6: https://k6.io
- xk6-kubernetes: https://github.com/grafana/xk6-kubernetes

---

## Remember

A green dashboard does not prove resilience. A successful chaos experiment does.

Happy breaking things! 🚀
