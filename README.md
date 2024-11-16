# fluxcd-best-practices


Flux Bootstrap Command

```
export GITHUB_TOKEN=<github-token>

<!-- check compatibility -->
flux check --pre


flux bootstrap github \
  --token-auth \
  --owner=mahinops \
  --repository=fluxcd-best-practices \
  --branch=main \
  --path=clusters/docker-desktop \
  --personal #skip this for organization

<!-- output structure  -->
├── clusters
│   └── docker-desktop
│       └── flux-system
│           ├── gotk-components.yaml
│           ├── gotk-sync.yaml
│           └── kustomization.yaml




```


## Check

```
kubectl get kustomization
kubectl get gitrepository
```


Tool Installation for Advance Config_
- kustomize

Install
```
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
sudo mv kustomize /usr/local/bin
```

Use
```
kustomize build apps/staging
```

Check live change

```
watch kubectl get ns
```