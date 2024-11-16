# Usage FluxCD

After configuring, to check the validity we will use this command_
```
kustomize build helm/repositories/<cluster-name>
```

## Steps:
1. Create a new folder with the cluster-name in cluster folder.
    ```
    mkdir clusters/<cluster-name>

    <!-- eg. -->
    mkdir clusters/kind-test-cluster
    ```

2. Copy everything from clusters/docker-desktop/flux-system and paste in new cluster folder.
    ```
    cp -r clusters/docker-desktop/flux-system clusters/<new-cluster>

    <!-- eg. -->
    cp -r clusters/docker-desktop/flux-system clusters/kind-test-cluster
    ```

3. Replace everything named `docker-desktop` to your current cluster name. 

    **e.g** => `docker-desktop` to `kind-test-cluster`

4. Switch to the correct kube context. To check and switch_
    ```
    #check current context
    kubectl config current-context 

    #see all contexts
    kubectl config get-contexts

    #switch context
    kubectl config use-context <context-name> 
    ```

5. Apply the changes for new cluster_
    ```
    kubectl apply -f clusters/<new-cluster-name>/flux-system

    <!-- eg.  -->
    kubectl apply -f clusters/kind-test-cluster/flux-system
    ```
    Will take some time to spin up all the pods. 

6. Create some folder named same with the cluster name in_
    - apps/
    - infrastructure/
    - helm3/releases/

    Or you can just simply copy the docker-desktop folder and paste and change the name.

7. Change the path of `apps.yaml`, `helm.yaml` and `infrastructure.yaml` files from the new cluster eg. `clusters/kind-test-cluster` folder.

8. Check if all the configs are okay_
    ```
    kustomize build ./helm3/releases/kind-test-cluster
    kustomize build ./apps/kind-test-cluster
    kustomize build ./infrastructure/kind-test-cluster
    ```

9. Push all changes to the repository. Remember, after pushing, the changes will reflect in the cluster if the `gotk-sync.yaml` branch refers to the same branch you are pushing. 

10. Wait for some time, [till `gotk-sync.yaml` mentioned kustomization interval time] and check the changes applied or not.