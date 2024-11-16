# Usage FluxCD

After configuring, to check the validity we will use this command_
```
kustomize build helm/repositories/<cluster-name>
```

## Steps:
1. Create a new folder with the cluster-name in cluster folder.
    ```
    mkdir -p clusters/<new-cluster> && cp clusters/docker-desktop/{apps.yaml,helm.yaml,infrastructure.yaml} clusters/<new-cluster>/

    <!-- eg. -->
    mkdir -p clusters/kind-test && cp clusters/docker-desktop/{apps.yaml,helm.yaml,infrastructure.yaml} clusters/kind-test/

    ```

2. In `apps.yaml`, `helm.yaml` and `infrastructure.yaml` files from the new cluster folder eg. `clusters/kind-test` folder change the path from `docker-desktop` to `kind-test` or by new cluster name.

    **e.g** => `docker-desktop` to `kind-test`

3. Switch to the correct kube context. To check and switch_
    ```
    #check current context
    kubectl config current-context 

    #see all contexts
    kubectl config get-contexts

    #switch context
    kubectl config use-context <context-name> 
    ```

4. Create some folder named same with the cluster name in_
    - apps/
    - infrastructure/
    - helm3/releases/

    Or you can just simply copy the docker-desktop folder and paste and change the name.


5. Check if all the configs are okay_
    ```
    kustomize build ./helm3/releases/kind-test
    kustomize build ./apps/kind-test
    kustomize build ./infrastructure/kind-test
    ```

6. Push all changes to the repository.

7. Apply command_
```
export GITHUB_TOKEN=<token-here>
flux bootstrap github \
  --token-auth \
  --owner=<github-username> \
  --repository=<repository-name> \
  --branch=main \
  --path=clusters/docker-desktop \
  --personal 
```

For my case I am applying the following command_

```
flux bootstrap github \
  --token-auth \
  --owner=mahinops \
  --repository=fluxcd-best-practices \
  --branch=main \
  --path=clusters/kind-test \
  --personal 
```