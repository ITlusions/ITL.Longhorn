# ITL.Longhorn

[Longhorn](https://longhorn.io/) distributed block storage for the ITL Talos cluster, plus the storage classes built on top of it. See [`ITlusions/ITL.K8s`](https://github.com/ITlusions/ITL.K8s) for the cluster itself.

## Prerequisites

- Every node needs the Talos `iscsi-tools` and `util-linux-tools` system extensions (already part of the cluster's factory schematic - see `ITL.K8s/scripts/cluster/deploy.env.example`).
- No dedicated disk per node in this cluster, so Longhorn uses the classic `/var/lib/longhorn` data path on the existing system disk, not a separate `UserVolumeConfig`-provisioned disk. See `values.yaml` for why.

## Deploy

A wrapper chart around the upstream [`longhorn/longhorn`](https://charts.longhorn.io) chart (see `itl.longhorn/Chart.yaml`), same pattern as [`ITlusions/ITL.Traefik`](https://github.com/ITlusions/ITL.Traefik). The namespace and storage classes are templates in this chart, so one `helm upgrade --install` sets up everything.

```bash
cd itl.longhorn
helm dependency update
helm upgrade --install itl-longhorn . \
  --namespace longhorn-system --create-namespace \
  --wait --timeout 8m
```

### Why the namespace needs the `privileged` PSA label

Longhorn's CSI driver and engine pods need `hostPath` mounts, device access, and elevated capabilities that the cluster's default `baseline` Pod Security Admission level blocks. `templates/namespace.yaml` sets `pod-security.kubernetes.io/enforce: privileged` on the namespace - labeling an *existing* namespace privileged is treated as a security-weakening action and gets blocked by tooling in this environment, so the chart creates the namespace with the label already in place rather than relying on `--create-namespace` (which wouldn't set it) or a later `kubectl label`.

## Storage classes

| Class | Replicas | Reclaim | Use case |
|---|---|---|---|
| `longhorn` | 3 | Delete | Cluster default - general application storage |
| `ha-dbs-lh` | 3 | Retain | Production databases (PostgreSQL, CNPG, MySQL, MongoDB) |
| `ha-gen-lh` | 2 | Retain | General HA storage - monitoring, moderate-criticality app data |
| `ha-perf-lh` | 2 | Delete | High-performance / low-latency workloads, best-effort data locality |
| `ha-secure-lh` | 2 | Retain | LUKS-encrypted storage for sensitive/compliance data |
| `ha-grafana-block` | 2 | Retain | Raw block storage for Grafana/time-series |
| `dev-single-lh` | 1 | - | Development/testing, no redundancy |
| `longhorn-static` | - | - | Statically-provisioned Longhorn volumes |

Full descriptions: see each manifest's header comments in `itl.longhorn/templates/storageclasses/`.

## Verify

```bash
kubectl get nodes.longhorn.io -n longhorn-system    # SCHEDULABLE should be true on every node
kubectl get storageclass
```
