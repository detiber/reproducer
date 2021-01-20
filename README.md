# Reproducer

## Prerequisites

### Standalone tools

- [kind](https://kind.sigs.k8s.io/) (v0.8.0+, Tested with v0.9.0)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) (tested with v1.18.8)
- [tilt](https://tilt.dev) (v0.17.8+, tested with v0.17.9)

## Setup

- Bring up a new kind cluster

```sh
kind create cluster
```

- Start tilt

```sh
tilt up
```

## Bring up the worker VM

```sh
kubectl create -f deploy/kind/worker.yaml
```

## What is expected

Worker vm is created and has an interface attached to a macvtap network via l2 connectivity

## What happens

virt-launcher pod fails to create with the following error:

```sh
  Warning  Failed          12s   kubelet, kind-control-plane  Error: failed to generate container "3059716e18ca6af8de9c131d0fb190fc24f6ef2079b0215a62907e0138c910bc" spec: failed to generate spec: lstat /dev/tap14: no such file or director
```

- Shelling into the kind container (`docker exec -it kind-control-plane bash`), /dev/tap14 does not exist
- On the host machine, /dev/tap14 is present

It appears that either macvtap-cni is creating the tap device on the physical host rather than in the kind-control-plane container where the kubelet (and the device plugin) are running.

## Teardown

```sh
kind delete cluster
```