# cloud-vs-edge-using-minikube
This project evaluates Kubernetes performance for deploying an Nginx VNF in cloud-like and edge-like environments using Minikube.

## Executive Performance Summary

This document synthesizes the data gathered during the stress testing of an Nginx Virtual Network Function (VNF). The evaluation contrasts a  Cloud environment against a strictly constrained Edge environment to assess viability, latency, and throughput variations.


---

## Layer-Specific Benchmarking Matrix

The following matrix isolates network performance into raw TCP bandwidth (Layer 4) and Application-level HTTP throughput (Layer 7). 

| Performance Metric | Cloud VM (Minikube - 2 CPU / 4GB) | Edge VM (MicroK8s - 1 CPU / 2GB) | Delta / Variance |
| :--- | :--- | :--- | :--- |
| **L4 Bandwidth (iperf3)** | 127 Gbits/sec | 85.3 Gbits/sec | Cloud +48.8% |
| **L7 Throughput (ab)** | 12,371.32 req/sec | 15,094.48 req/sec | Edge +22.0% |
| **L7 Latency (ab)** | 8.083 ms | 6.625 ms | Edge -1.458 ms |

---

## Analytical Insights: The Throughput Paradox

for initial hardware , the 1-CPU Edge environment outperformed the 2-CPU Cloud environment in Layer 7 Application throughput. This exposes a critical networking phenomenon:

- **The Cloud NAT Penalty:** The Minikube environment operates via an internal Docker network bridge (`192.168.49.2`). Every HTTP request processed during the ApacheBench stress test had to traverse an additional Network Address Translation (NAT) hop. Over 1,000,000 requests, this compounded to an 8.083 ms average latency.
- **Edge Host-Routing Efficiency:** The MicroK8s Edge setup utilized direct host-routing, bypassing virtualization bridge overhead. This resulted in a cleaner network path, yielding a superior 15,094 req/sec at a lower 6.625 ms latency.
- **L4 Compute Dependency:** Conversely, the `iperf3` bandwidth test, which floods the system with raw TCP packets, is heavily CPU-bound. The Cloud VM's secondary CPU core allowed the Linux kernel to process network queues significantly faster, securing a dominant 127 Gbits/sec compared to the Edge's 85.3 Gbits/sec.

---

## Compute Footprint & Constraint Adherence

A primary objective was ensuring the VNF could survive within a rigid **2GB RAM** Edge boundary without triggering an Out-Of-Memory (OOM) eviction.

| Compute State | Cloud VNF Utilization | Edge VNF Utilization | Limit Thresholds |
| :--- | :--- | :--- | :--- |
| **Idle CPU State** | 34m | 96m | 250m |
| **Idle Memory State** | 3 Mi | 2 Mi | 128 Mi |

- **Efficiency Validation:** The Nginx Alpine image proved remarkably efficient, resting at merely 2-3 Mi of memory. 
- **CPU Baseline Shift:** The Edge VM exhibited a higher idle CPU draw (96m vs. Cloud's 34m). This is a direct consequence of running the entire Kubernetes control plane and networking proxy on a single, shared CPU core.
- **Observability Strategy:** To protect the 2GB Edge limit, heavy telemetry stacks (e.g., Prometheus) were intentionally omitted. Native Kubernetes APIs (`kubectl top`) and the lightweight `metrics-server` dashboard were successfully leveraged to capture deployment footprint without threatening node stability.
