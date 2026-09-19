---
tags: [cloud, containerisation, kubernetes, cni, networking, calico, cilium]
type: cheatsheet
source: compiled reference (CNI / Calico / Cilium)
last-verified: 2026-09-16
---

# CNI

## Up
- [[Kubernetes]]

The **Container Network Interface (CNI)** is the plugin standard that gives every pod an IP and wires up pod-to-pod networking. Kubernetes ships no networking itself — you install a CNI plugin (**Calico**, **Cilium**, Flannel, Weave, or a cloud CNI). The plugin also enforces **NetworkPolicy**.

---

## What a CNI does
- Assigns each pod an IP (IPAM) and connects it to the cluster network so **any pod can reach any pod** (the K8s networking model).
- Programs routes/overlays (VXLAN/IP-in-IP) or native routing (BGP/eBPF).
- Enforces **NetworkPolicy** (default-allow until you add policies).

```yaml
# NetworkPolicy — default-deny ingress, then allow app→db
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: { name: db-allow-app, namespace: prod }
spec:
  podSelector: { matchLabels: { app: db } }
  policyTypes: [Ingress]
  ingress:
    - from: [{ podSelector: { matchLabels: { app: web } } }]
      ports: [{ protocol: TCP, port: 5432 }]
```

Standard NetworkPolicy is namespaced and L3/L4 only; Calico/Cilium add richer CRDs.

---

## Calico
- Modes: **overlay** (VXLAN/IP-in-IP) or **native routing via BGP** (no overlay, high performance); optional **eBPF** dataplane.
- Strong, mature **network policy** — `GlobalNetworkPolicy`, `NetworkPolicy` (Calico CRD) with ordering, DNS/domain rules, host-endpoint protection.
```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/<ver>/manifests/tigera-operator.yaml
calicoctl get networkpolicy -A
```

## Cilium
- **eBPF-based** dataplane (in-kernel, no iptables scaling limits) — high performance + deep visibility.
- **L7-aware policy** (HTTP/gRPC/Kafka methods & paths, DNS), transparent encryption, **Hubble** for flow observability, cluster mesh, and can replace kube-proxy.
```bash
cilium install ; cilium status ; cilium hubble ui
```
```yaml
# CiliumNetworkPolicy — L7 (only allow GET /api)
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy
spec:
  endpointSelector: { matchLabels: { app: api } }
  ingress:
    - toPorts: [{ ports: [{ port: "80" }], rules: { http: [{ method: GET, path: /api }] } }]
```

---

## Choosing

| | Calico | Cilium |
|---|---|---|
| Dataplane | iptables/BGP or eBPF | **eBPF** (kernel) |
| Policy depth | L3/L4 (+ global, host) | **L3–L7** (HTTP/DNS/Kafka) |
| Observability | Basic | **Hubble** flow maps |
| Extras | BGP peering, host protection | kube-proxy replacement, encryption, cluster mesh, service-mesh-lite |
| Pick when | Solid, proven policy + BGP | eBPF performance, L7 policy, observability |

Flannel = simplest (overlay, **no** NetworkPolicy). Managed clusters often default to a cloud CNI (AWS VPC CNI, Azure CNI) — you can swap to Calico/Cilium for policy.

---

## Tips
- The CNI is chosen **at cluster creation** and is disruptive to change later — pick deliberately.
- NetworkPolicy is **default-allow**; adopt a **default-deny** baseline per namespace, then allow explicitly (a policy engine like [[Kyverno & OPA Gatekeeper]] can auto-generate it).
- Cilium's eBPF + Hubble overlaps some [[Service Mesh]] features (L7 policy, mTLS) without sidecars.
- Debug: `kubectl get networkpolicy -A`, check the CNI pods in `kube-system`, and confirm pods actually got IPs.
