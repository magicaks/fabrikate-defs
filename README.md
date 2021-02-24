# MagicAKS Fabrikate high-level definitions

This repository contains [Fabrikate](https://github.com/microsoft/fabrikate) high-level definitions (HLD) for [MagicAKS](https://github.com/magicaks/magicaks).

MagicAKS sets up [Flux (GitOps)](https://fluxcd.io/) to track [the Kubernetes (K8s) manifest repository](https://github.com/magicaks/k8smanifests) ("manifest repo" for short). Any changes made to Fabrikate definitions here will trigger the [GitHub Actions](https://docs.github.com/en/actions) pipeline ([`.github/workflows/main.yml`](./.github/workflows/main.yml)) and push new changes to the manifest repo and those changes will eventually be reflected in the cluster.

The [`build.sh`](./build.sh) script, executed by the pipeline, creates the necessary [role-based access control (RBAC)](https://docs.microsoft.com/en-us/azure/role-based-access-control/overview) configuration, which is then placed in the Fabrikate generated folder and pushed to the manifest repo.

## Getting started

Execute the following steps to initialize Flux (GitOps) for your cluster:

1. Edit the [`users.yaml`](./users.yaml) file to specify the list of users and groups that have access to the cluster

    > **Important:** User and group object IDs are specific to an AAD tenant. Make sure to retrieve the user and group object IDs from the AAD tenant that governs the RBAC access to the cluster.

    * Since MagicAKS is a RBAC enabled cluster, users and groups are defined in [Azure Active Directory (AAD)](https://docs.microsoft.com/en-us/azure/active-directory/fundamentals/active-directory-whatis). You can retrieve the object IDs of users and groups from AAD in [Azure Portal](https://portal.azure.com) or by [command line tools](https://docs.microsoft.com/en-us/azure/healthcare-apis/find-identity-object-ids). Examples using Azure CLI:
        * User object ID:

            ```bash
            az ad user show --id "<user principal name>" --query objectId --out tsv
            ```

        * Group object ID:

            ```bash
            az ad group show --group "<group name>" --query objectId --out tsv
            ```

1. Create a secret for this repository containing an access token so that the GitOps process can monitor repositories and update manifests
    1. [Create a personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) with **repo** scope (full control of private repositories)
        > **Note:** Make sure to copy the access token value once created, because you cannot access it again.
    1. [Create a repository secret](https://docs.github.com/en/actions/reference/encrypted-secrets#creating-encrypted-secrets-for-a-repository), named `ACCESS_TOKEN`, for this repository using the value of the personal access token
1. Duplicate or fork [the manifest repo](https://github.com/magicaks/k8smanifests) to create one for yourself
1. Change the value of the `REPO` variable in the last step of [the `main.yml` pipeline file](.github/workflows/main.yml) to point to your new manifest repo created in the previous step
1. Make sure the build script and the Fabrikate executable have execute permissions set so that the GitHub Actions pipeline can run them:

    ```bash
    git update-index --chmod=+x build.sh
    git update-index --chmod=+x bin/fab
    ```

    > **Note:** These changes too need to be committed (`git commit`).

1. Commit and push the changes made
    * This will trigger the GitHub Actions pipeline, which runs Fabrikate to generate the K8s manifests and pushes them to the manifest repo
    * Check the output of the pipeline to ensure everything ran well; if the run was successful, you should see changes applied to your manifest repo
