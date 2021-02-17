# Generate Admin K8s Manifests for MagicAKS

This repo contains high level Fabrikate specification for MagicAKS.

## Steps

Edit users.yaml to specify list of users and groups which have access to cluster. Since MagicAKS is an RBAC enabled cluster, users and groups are specified in Azure Active Directory(AAD). You can get the id's for users and groups from AAD in Azure Portal.

Create a github actions pipeline using the content of this [file](.github/workflows/main.yml). The pipeline runs fabrikate to generate kubenetes manifests and pushes them to k8s admin manifest git repo.

Flux(gitOps) is setup to track admin manifest repo and any changes made to fabrikate definitions will trigeer the actions pipeline and push new changes to admin manifest repo and eventually reflect in the cluster.

Within ``build.sh`` which is executed from the actions pipeline mentioned above you will find

```bash
function generate_rbac_configs() {
    echo "generate rbac configs"
    python rbac-generator.py $1 $2
}

function generate_azmon_configs() {
    echo "generate azure monitor configs"
    python azmonconfig-generator.py $1 $2
}
```
This creates the necessary rbac configs which are then placed in the fabrikate generated folder and hence pushed to the k8s manifest repo during git push.