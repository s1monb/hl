# Homelab

## Todos

- [ ] Cluster up and running with Cilium (kube-proxy replacement)
- [ ] Argo CD configured for GitOps within this repo (Might change to a separate repo)

## Overview

`talos/` - Talos k8s configurations

## Talos setup

When you are running the `talosctl gen config`-command, you need to add the `--config-patch @patch.yaml` flag to the it. You might also want to export the kubeconfig file to `$HOME/.kube/config` instead of `.`. Other than that, follow [this guide](https://www.talos.dev/v1.7/talos-guides/install/virtualized-platforms/proxmox/)

### Cilium installation

This might take some time to complete, so be patient.

```bash
cilium install \
    --helm-set=ipam.mode=kubernetes \
    --helm-set=kubeProxyReplacement=true \
    --helm-set=securityContext.capabilities.ciliumAgent="{CHOWN,KILL,NET_ADMIN,NET_RAW,IPC_LOCK,SYS_ADMIN,SYS_RESOURCE,DAC_OVERRIDE,FOWNER,SETGID,SETUID}" \
    --helm-set=securityContext.capabilities.cleanCiliumState="{NET_ADMIN,SYS_ADMIN,SYS_RESOURCE}" \
    --helm-set=cgroup.autoMount.enabled=false \
    --helm-set=cgroup.hostRoot=/sys/fs/cgroup \
    --helm-set=k8sServiceHost=localhost \
    --helm-set=k8sServicePort=7445
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
export CONTROL_PLANE_IP="192.168.1.118"

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
