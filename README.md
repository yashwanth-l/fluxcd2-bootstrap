# FluxCD2 Bootstrap

This repository contains a set of files and configurations to help you bootstrap your Kubernetes cluster using FluxCD v2. FluxCD is a popular GitOps tool that automates the deployment and management of Kubernetes resources using Git as the source of truth.

## Getting Started

To get started with FluxCD2 Bootstrap, follow these steps:

- Clone this repository to your local machine:

```sh
git clone https://github.com/yashwanth-l/fluxcd2-bootstrap.git
```

- Install FluxCD CLI (flux) by following the official documentation: [https://fluxcd.io/docs/cmd/](https://fluxcd.io/docs/cmd/)

- Configure your Kubernetes cluster credentials by setting the `kubeconfig` file path as an environment variable:

```sh
export KUBECONFIG=/path/to/your/kubeconfig
```

- Bootstrap your cluster to install fluxcd, in this scenario Github(Personal Account):

```sh
export GITHUB_TOKEN=********
flux bootstrap github \
    --owner=<user> \
    --repository=repository name> \
    --private=false \
    --personal=true \
    --path="./clusters/my-cluster" \
    --branch=main \
    --author-email="flux-github-repo-key@gmail.com" \
    --author-name="flux-github-repo-key(FLUX2)" \
    --commit-message-appendix="[ci skip]" \
    --context=kind-k8s-local \
    --components-extra=image-reflector-controller,image-automation-controller \
    --secret-name=flux-github-key \
    --token-auth
```

The outputs looks like below

```sh
► connecting to github.com
► cloning branch "main" from Git repository "https://github.com/*******.git"
✔ cloned repository
► generating component manifests
# Warning: 'patchesJson6902' is deprecated. Please use 'patches' instead. Run 'kustomize edit fix' to update your Kustomization automatically.
✔ generated component manifests
✔ component manifests are up to date
► installing components in "flux-system" namespace
✔ installed components
✔ reconciled components
► determining if source secret "flux-system/flux-github-key" exists
► generating source secret
► applying source secret "flux-system/flux-github-key"
✔ reconciled source secret
► generating sync manifests
✔ generated sync manifests
✔ sync manifests are up to date
► applying sync manifests
✔ reconciled sync configuration
◎ waiting for Kustomization "flux-system/flux-system" to be reconciled
✔ Kustomization reconciled successfully
► confirming components are healthy
✔ helm-controller: deployment ready
✔ image-automation-controller: deployment ready
✔ image-reflector-controller: deployment ready
✔ kustomize-controller: deployment ready
✔ notification-controller: deployment ready
✔ source-controller: deployment ready
✔ all components are healthy
```

At this point the the repo contains the below:

```sh
└── clusters
    └── flux-kind-k8s
        └── flux-system
            ├── gotk-components.yaml
            ├── gotk-sync.yaml
            ├── kustomization.yaml
```

- This can be _customized_ to your needs to install any components, as I do in my Cluster as described below

  - **gitrepositories:** Contains FLUXCD's GitRepositories CRD based manifests

  - **helmrepositories:** Contains FLUXCD's HelmRepositories CRD based manifests

  - **helmreleases:** Contains FLUXCD's HelmRelease CRD based manifests

  - **kustomizations:** Contains FLUXCD's Kustomization(_not to be confused with Kubernetes Kustomization!_) CRD based manifests

  - **namespaces:** Contains the namespaces to be installed in the cluster

  - **secrets:** Contains the secrets to be installed in the cluster, via [sops-age](https://fluxcd.io/flux/guides/mozilla-sops/#encrypting-secrets-using-age) feature

    - Since I used [age-keygen](https://github.com/FiloSottile/age#installation) below steps were also executed

    ```sh
    age-keygen -o $HOME/age.agekey

    cat sops-age-key.txt |
        kubectl create secret generic flux-sops-age \
        --namespace=flux-system \
        --from-file=$HOME/age.agekey=/dev/stdin \
        --dry-run=client
    ```

    - The location of the files to be decrypted should have something like described below for our use-case

    ```sh
    ❯ cat .sops.yaml
    keys:
    - &some-alias public-key-from-output-of-age-keygen

    creation_rules:
    - encrypted_regex: '^(data|stringData)$'
      key_groups:
      - age:
        - *some-alias
    ```

    - Encryption and Decryption of the files can be done as below

    ```sh
    # Encryption
    SOPS_AGE_KEY_FILE=$HOME/age.agekey \
    sops \
        --encrypt \
        --in-place \
        --verbose \
        <someFile>

    # Decryption
    SOPS_AGE_KEY_FILE=$HOME/age.agekey \
    sops \
        --decrypt \
        --in-place \
        --verbose \
        <someFile>
    ```

  - **shard1:** This is a feature of FluxCD which can be used when you use flux to deploy tons of applications as explained [here](https://fluxcd.io/flux/cheatsheets/sharding/)

## Support

If you have any questions or need assistance, please open an [issue](https://github.com/yashwanth-l/fluxcd2-bootstrap/issues).
