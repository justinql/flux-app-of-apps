# Install FluxCD
## Install CLI
curl -s https://fluxcd.io/install.sh | sudo bash

## Bootrap Cluster
```bash
export GITHUB_TOKEN=
```

```bash
flux bootstrap github \
  --token-auth \
  --owner=justinql \
  --repository=flux-app-of-apps \
  --branch=main \
  --path=clusters/home \
  --personal
```

### Air-Gap
```
flux bootstrap git \
  --registry=registry.internal/fluxcd \
  --url=ssh://git@<host>/<org>/<repository> \
  --branch=<my-branch> \
  --private-key-file=<path/to/private.key> \
  --password=<key-passphrase> \
  --path=clusters/my-cluster
```

## Usefule

https://fluxcd.io/flux/guides/repository-structure/
```
├── apps
│   ├── base
│   ├── production 
│   └── staging
├── infrastructure
│   ├── base
│   ├── production 
│   └── staging
└── clusters
    ├── production
    └── staging
```
https://github.com/fluxcd/flux2-kustomize-helm-example

### See if it synced or errors: 

```bash
flux get kustomizations --watch
```
### Force to re-sync:
```
flux reconcile kustomization infra-controllers
```

### Troubleshoot:
```
flux -n minio get helmreleases
```

RBAC assuming no outside access to git. If there is, use multi-tenant model (main git repo referes to external git repos for apps, add extra rules to prevent one app from managing another)
https://fluxcd.io/flux/installation/configuration/multitenancy/
https://github.com/fluxcd/flux2-multi-tenancy

## Managing Secrets
https://fluxcd.io/flux/guides/mozilla-sops/


## Other
### Velero
#### CLI
https://velero.io/docs/v1.15/basic-install/#install-the-cli
```bash
wget https://github.com/vmware-tanzu/velero/releases/download/v1.15.2/velero-v1.15.2-linux-amd64.tar.gz
tar xf velero-v1.15.2-linux-amd64.tar.gz
sudo install velero-v1.15.2-linux-amd64/velero /usr/local/bin/velero
```
