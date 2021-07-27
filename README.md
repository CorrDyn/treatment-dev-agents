# Development tools & agents

This repo contains [Skaffold][skaffold] and [Kustomize][kustomize] configurations to be used in local development environments. The following are currently supported:

* Google Cloud Spanner local emulator
* Redis


## Using the agent configurations

Clone the `dev-agents` repository in the directory just above your application configuration, so from within your application directory it's reachable with `ls ../dev-agents`.

### Adding Google Cloud Spanner emulator deployment to your project

The following configuration will deploy a Google Cloud Spanner emulator to the `dev-agents` namespace in your local cluster:

```yaml
apiVersion: skaffold/v2beta18
kind: Config
metadata:
  name: my-agent

requires:
  # Adds the cloud-spanner emulator as a requirement of the project
  - configs: ["cloud-spanner-emulator"]
    path: ../dev-agents

# Normal project build & deploy conig
build:
  artifacts:
  - image: my-agent
    buildpacks:
      builder: "gcr.io/buildpacks/builder"
      env:
        - NODE_ENV=development
  local: {}

deploy:
  kubeContext: minikube
  kustomize:
    paths:
    - .k8s/local-dev
```

Run skaffold with `skaffold dev --port-forward`. The Cloud Spanner emulator will expose ports `9010` and `9020` on your host machine.  To configure the application to use the local emulator, make sure the following entry exists in the `configMapGenerator` of the application kustomize deployment so the Google Cloud libraries override the spanner connection:

```yaml
configMapGenerator:
- name: my-agent
  literals:
  - SPANNER_EMULATOR_HOST="cloud-spanner-emulator.dev-agents.svc:9010"

```

See more details in the [Cloud Spanner Emulator docs][spanner-emulator].

### Adding Redis deployment to your project

The following configuration will deploy a Redis container to the `dev-agents` anmespace in your local cluster:

```yaml
apiVersion: skaffold/v2beta18
kind: Config
metadata:
  name: my-agent

requires:
  # Adds Redis as a requirement of the project
  - configs: ["redis"]
    path: ../dev-agents

# Normal project build & deploy conig
build:
  artifacts:
  - image: my-agent
    buildpacks:
      builder: "gcr.io/buildpacks/builder"
      env:
        - NODE_ENV=development
  local: {}

deploy:
  kubeContext: minikube
  kustomize:
    paths:
    - .k8s/local-dev
```

Run skaffold with `skaffold dev --port-forward`. Reids will run and expose port `6379` on your host machine. To configure the application to use the local cluster Redis, make sure the following entry exists in the `configMapGenerator` of the application kustomize deployment:

```yaml
configMapGenerator:
- name: my-agent
  literals:
  - REDISURI="redis.dev-agents.svc:6379"

```

[spanner-emulator]: https://cloud.google.com/spanner/docs/emulator
[kustomize]: https://kubectl.docs.kubernetes.io/installation/kustomize/
[skaffold]: https://skaffold.dev
