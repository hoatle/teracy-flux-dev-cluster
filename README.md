# teracy-flux-dev-cluster

GitOps for teracy-dev-k8s local cluster

## Prerequisites

- `teracy-dev-k8s` cluster is installed, see: https://github.com/teracyhq-incubator/teracy-dev-k8s
- `kubectl` is installed and configured to work on the target cluster
- `helm (v3)` is installed, see: https://helm.sh/docs/intro/install/
- `fluxctl` is installed, see: https://docs.fluxcd.io/en/1.18.0/references/fluxctl.html#installing-fluxctl

## Installation

This is for cluster-wide management, so flux requires full cluster permission.

- Fork and clone this repo so that you can continue editing on the config repo, for example:

```bash
$ cd ~/k8s-dev/workspace
$ git clone git@github.com:hoatle/teracy-flux-dev-cluster.git
```

- Add fluxcd helm repository and install flux custom resource definition:

```bash
$ helm repo add fluxcd https://charts.fluxcd.io
$ kubectl apply -f https://raw.githubusercontent.com/fluxcd/helm-operator/master/deploy/crds.yaml
```

- Install Flux:

```bash
$ kubectl create namespace flux-system
$ export FLUX_GIT_REPO=git@github.com:hoatle/teracy-flux-dev-cluster # replace by your repo here
$ helm upgrade -i flux fluxcd/flux \
--set git.url=$FLUX_GIT_REPO \
--set git.branch=teracy-k8s-dev \
--namespace flux-system
```

- Install Flux helm operator:

```bash
$ helm upgrade -i helm-operator fluxcd/helm-operator \
--set git.ssh.secretName=flux-git-deploy \
--set helm.versions=v3 \
--namespace flux-system
```

- Get the Flux public key:

```bash
$ fluxctl identity --k8s-fwd-ns flux-system
```

- Open GitHub, navigate to your fork, go to Setting > Deploy keys, click on Add deploy key, give
  it a Title, check Allow write access, paste the Flux public key and click Add key.


## Backup and Restore

- Don't forget to back up the private key:

```bash
$ kubectl -n flux-system get secrets flux-git-deploy -o yaml > .secrets/flux-git-deploy.yaml
```

- And restore this private key and flux-system when needed:

```bash
$ kubectl create ns flux-system # if any
$ kubectl delete secrets flux-git-deploy -n flux-system # delete if any
$ kubectl apply -f .secrets/flux-git-deploy.yaml -n flux-system
$ helm upgrade -i flux fluxcd/flux \
--set git.url=$FLUX_GIT_REPO \
--set git.branch=teracy-k8s-dev \
--set git.secretName=flux-git-deploy \
--namespace flux-system
$ helm upgrade -i helm-operator fluxcd/helm-operator \
--set git.ssh.secretName=flux-git-deploy \
--set helm.versions=v3 \
--namespace flux-system
```

## Tips

- See Flux logs:

```bash
$ kubectl -n flux-system logs -f deployment/flux
```

- Flux sync manually to trigger Flux sync immediately:

```bash
$ fluxctl sync --k8s-fwd-ns flux-system
```


## References

- https://docs.fluxcd.io/en/latest/tutorials/get-started-helm.html#install-flux
