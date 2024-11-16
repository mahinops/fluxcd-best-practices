# Usage FluxCD

After configuring, to check the validity we will use this command_
```
kustomize build helm/repositories/<cluster-name>
```

## Steps:
- Create a new folder with the cluster-name in cluster folder.
```
mkdir clusters/<cluster-name>

<!-- eg. -->
mkdir clusters/kind-test-cluster
```

- Copy everything from clusters/docker-desktop/flux-system and paste in new cluster folder.
```
cp -r clusters/docker-desktop/flux-system clusters/<new-cluster>

<!-- eg. -->
cp -r clusters/docker-desktop/flux-system clusters/kind-test-cluster
```

- Replace everything named `docker-desktop` to your current cluster name. 
**e.g** => `docker-desktop` to `kind-test-cluster`

- Switch to the correct kube context. To check and switch_
```
#check current context
kubectl config current-context 

#see all contexts
kubectl config get-contexts

#switch context
kubectl config use-context <context-name> 
```

- Apply the changes for new cluster_
```
kubectl apply -f clusters/<new-cluster-name>/flux-system

<!-- eg.  -->
kubectl apply -f clusters/kind-test-cluster/flux-system
```
Will take some time to spin up all the pods. 

- Create some folder named same with the cluster name in_
  - apps/
  - infrastructure/
  - helm3/releases/

  Or you can just simply copy the docker-desktop folder and paste and change the name.

- Change the path of `apps.yaml`, `helm.yaml` and `infrastructure.yaml` files from the new cluster eg. `clusters/kind-test-cluster` folder.

- Check if all the configs are okay_
```
kustomize build ./helm3/releases/kind-test-cluster
kustomize build ./apps/kind-test-cluster
kustomize build ./infrastructure/kind-test-cluster
```

- Push all changes to the repository. Remember, after pushing, the changes will reflect in the cluster if the `gotk-sync.yaml` branch refers to the same branch you are pushing. 
