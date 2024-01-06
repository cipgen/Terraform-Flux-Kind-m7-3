**Module 7 - Task 3 - Terraform + Flux Interanal cluster Kind**

- Let's add a module responsible for deploying the cluster according to the task.


````console
module "kind_cluster" {
  source = "github.com/den-vasyliev/tf-kind-cluster?ref=cert_auth"
}
````
Terraform initialization

```console
tf init
```

- lists all the resources that are managed by Terraform in the current state.

```console
tf state list
```
- Code validation

```console
tf validate
```

- Applies the defined infrastructure changes.

```console
tf apply
```

**result**
module.flux_bootstrap.flux_bootstrap_git.this
module.github_repository.github_repository.this
module.github_repository.github_repository_deploy_key.this
module.kind_cluster.kind_cluster.this
module.tls_private_key.tls_private_key.this

- Checking Flux system pods' namespace list.

```console
k get ns

NAME                 STATUS   AGE
default              Active   44m
flux-system          Active   44m
kube-node-lease      Active   44m
kube-public          Active   44m
kube-system          Active   44m
local-path-storage   Active   44m


mod7_task3_tf_flux git:(main) ✗ k get po -n flux-system
NAME                                       READY   STATUS    RESTARTS      AGE
helm-controller-8d775f7f6-wc8bx            1/1     Running   2 (11h ago)   12h
kustomize-controller-7b97748758-5cr2h      1/1     Running   2 (11h ago)   12h
notification-controller-7485f4b5dd-4j5f5   1/1     Running   2 (11h ago)   12h
source-controller-69db56c884-vc8pn         1/1     Running   2 (11h ago)   12h

```

- Install the Flux CLI client.

```console
curl -s https://fluxcd.io/install.sh | bash
flux get all
```

- Monitoring logs command

```console
flux logs -f
```

- add manifest demo/ns.yaml --> flux-gitops (create namespace on Kind cluster)

```console
apiVersion: v1
kind: Namespace
metadata:
  name: demo
```

- kbot-git-repository.yaml --> flux-gitops (Add a GitRepository manifest.)

```console
flux create source git kbot \
    --url=https://github.com/cipgen/kbot \
    --branch=main \
    --namespace=demo \
    --export

apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: kbot
  namespace: demo
spec:
  interval: 1m0s
  ref:
    branch: main
  url: https://github.com/cipgen/kbot.git
```

____

add manifest  demo/kbot-helm-release.yaml   --> flux-gitops (Add a HelmRelease manifest.)

```console
flux create helmrelease kbot \
    --namespace=demo \
    --source=GitRepository/kbot \
    --chart="./helm" \
    --interval=1m \
    --export
```

- **kubectl get po -n demo** - Check the creation of pods in the "demo" namespace.
- **kubectl describe pods -n demo** - Getting information about pods in the "demo" namespace.
- **terraform destroy** - Destroying the infrastructure, stopping the cluster's operation.