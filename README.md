# Generate Admin K8s Manifests for MagicAKS

This repo contains high level [Fabrikate](https://github.com/microsoft/fabrikate) specification for MagicAKS.

## Steps

Edit users.yaml to specify list of users and groups which have access to cluster. Since MagicAKS is an RBAC enabled cluster, users and groups are specified in Azure Active Directory(AAD). You can get the id's for users and groups from AAD in Azure Portal.

[Create](https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository) a repository secret named ACCESS_TOKEN which holds github [Personal Access Token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) with repo permissions.

Fork the [repo](https://github.com/magicaks/k8smanifests) to create an admin manifest repo for yourself.

Change **REPO** in [file](.github/workflows/main.yml) to point to your new admin manifest repo created above.

Commit the changes made to origin. This will trigger an actions pipeline run. The pipeline runs fabrikate to generate kubenetes manifests and pushes them to k8s admin manifest git repo. Check the output of the actions pipeline and ensure everything ran well. If the run is successful you should see changes applied to your admin manifest repo.

MagicAKS sets up Flux(gitOps) to track admin manifest repo. Any changes made to fabrikate definitions will trigger the actions pipeline and push new changes to admin manifest repo and eventually reflect in the cluster.

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
