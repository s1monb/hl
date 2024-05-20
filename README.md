# Homelab

## Plans

- [x] Cluster up and running with Cilium (kube-proxy replacement)
- [ ] Argo CD configured for GitOps within this repo (Might change to a separate repo)

## Overview

`talos/` - Talos k8s configurations
`infra/` - Kubernetes tools and configuration

## Talos setup

### Installation

When you are running the `talosctl gen config`-command, you need to add the `--config-patch @patch.yaml` flag to the it. You might also want to export the kubeconfig file to `$HOME/.kube/config` instead of `.`.

Other than that, follow [this guide](https://www.talos.dev/v1.7/talos-guides/install/virtualized-platforms/proxmox/). Then go to the [next section](#cilium-installation).

#### Cilium installation

This might take some time to complete, so be patient.

```bash
kubectl kustomize --enable-helm ./infra/cilium | kubectl apply -f -
```

### Removing nodes

To remove the node from the cluster, you run the following two commands:

```bash
talosctl -n <IP.of.node.to.remove> reset

kubectl delete node <nodename>
```

### Updating the cluster

The way to update the cluster, is to swap out the nodes with new updated ones.

#### Control Plane-nodes:

Simply run:

```bash
talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file _out/controlplane.yaml
```

> Keep in mind that if you replace the control-plane, you have to run the commands below to update the kubeconfig-file.

```bash
# Set it to the new IP shown in the console of newly installed controlplane-node
export CONTROL_PLANE_IP="ip.address.to.your.control.plane"

# Point to the talosconfig-file (used by talosctl)
export TALOSCONFIG="_out/talosconfig"
talosctl config endpoint $CONTROL_PLANE_IP
talosctl config node $CONTROL_PLANE_IP

# Export the new kubectl config:
talosctl kubeconfig $HOME/.kube/config
```

#### Worker-nodes

Simply run:

```bash
talosctl apply-config --insecure --nodes $WORKER_IP --file _out/worker.yaml
```

## ArgoCD setup

Install ArgoCD from the `./infra/argocd` kustomize-files:

```bash
kubectl kustomize --enable-helm ./infra/argocd | kubectl apply -f -
```

With the cilium configuration we have, your argocd-server service should have been assigned its own IP on your LAN. Make sure you can reach your argo login-screen with this IP (port 80).

Assuming you have gotten to the login-screen, you have to get the credentials from argocd. There is an autogenerated password for you within the cluster. Just query the kubernetes with the query below and log in with it (username: `admin`)

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}"
```

Now I would suggest you change the password and store it somewhere else.
