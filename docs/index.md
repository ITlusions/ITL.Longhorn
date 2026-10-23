# ITL.Longhorn documentation index

Documents are grouped into numbered folders. Each file is named `x.y-title.md`, where `x` is the folder number, matching the convention used in [`ITlusions/ITL.K8s`](https://github.com/ITlusions/ITL.K8s).

## 1. Longhorn

- [1.1 Architecture](1-longhorn/1.1-architecture.md): what the chart deploys, the Talos prerequisites, and the data-path decision.
- [1.2 Deployment](1-longhorn/1.2-deployment.md): installing and upgrading the chart, and the namespace privileged-PSA caveat.
- [1.3 Storage classes](1-longhorn/1.3-storage-classes.md): the eight storage classes this chart ships, their settings, and how to pick one.
- [1.4 UI access](1-longhorn/1.4-ui-access.md): the Longhorn UI Ingress, the IP allowlist, and why there's no login.

## Manifests

These aren't documents, but the docs refer to them.

- `itl.longhorn/templates/storageclasses/`: the storage class manifests.
- `itl.longhorn/templates/namespace.yaml`: the `longhorn-system` namespace with its privileged PSA label.
- `itl.longhorn/templates/ui-ingress.yaml`: the ACL Middleware and Ingress for the Longhorn UI.
- `itl.longhorn/values.yaml`: the Talos-specific data-path override.
