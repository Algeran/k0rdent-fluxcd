# k0rdent FluxCD repo template

## Prerequisites

1. `kubectl` binary installed and added to the PATH
2. This repo uses `task-go` utility to run commands. Please [install](https://taskfile.dev/installation/) it and add it to the PATH
2. kubernetes cluster that will be used as the [management cluster](https://k0rdent.github.io/docs/glossary/#management-cluster). Configure kubectl to connect and use this cluster
3. If you decide to bootstrap FluxCD using this repo (other options described in next steps), you need to [install](https://fluxcd.io/flux/installation/) `flux` CLI and add it to the PATH


## Produced GitOps repo structure

```
/
├── clusters                                            # k0rdent Cluster deployments
├── global-services                                     # k0rdent global beach-head services
├── templates                                           # Custom ClusterTemplates and ServiceTemplates
├── credentials                                         # Platform credentials
├── management                                          # Management cluster configuration
│   └── k0rdent                                         # k0rdent platform configuration
└── flux                                                # Flux CD configurations
    ├── flux-system
    └── k0rdent.yaml
```

## Setup the repo

### Stage 1. Init configs

> [!IMPORTANT]
> The [config.sample.yaml](./config.sample.yaml) file contains variables that are **vital** to the template process.

1. Genertae the `config.yaml` from the [config.sample.yaml](./config.sample.yaml) configuration file:

    ```shell
    task init
    ```

2. Fill out the `config.yaml` configuration file using the comments in that file as a guide.

### Stage 2. Bootstrap or configure FluxCD

> [!IMPORTANT]
> If you already have the installed Flux CD in your management cluster, you can skip this stage

Run the command and wait when Flux is installed to the cluster and configuration is synced to the repo:
    ```shell
    task bootstrap:flux
    ```

### Stage 3. Generate k0rdent configuration

1. Template out all the configuraion files:

    ```shell
    task configure
    ```

2. Push your changes to git:

    ```shell
    git add -A
    git commit -m "chore: initial commit"
    git push
    ```

### Stage 4. Watch the rollout of k0rdent

    Currently only KCM k0rdent component is installed with this repo. When KCM controllers is installed, it starts to bootstrap all the required configuration, providers and components. To monitor the KCM installation you can watch the `Management` type object. By default, it's called `kcm`. 

    > Creation of the Management object can take some time

    ```shell
    kubectl -n kcm-system get management.k0rdent.mirantis.com kcm -o go-template='{{range $key, $value := .status.components}}{{$key}}: {{if $value.success}}{{$value.success}}{{else}}{{$value.error}}{{end}}{{"\n"}}{{end}}'
    ```

    All components in the list must have the value `true`

