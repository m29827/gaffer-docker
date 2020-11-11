# Building and deploying the Gaffer Operator in kind

## Pre-requisite Utilities
Installation of the Operator SDK (https://sdk.operatorframework.io) and Operator Package Manager (https://github.com/operator-framework/operator-registry) utilities are required to build and package the Gaffer Operator.

## Local Docker Repository Configuration
To generate and use Operator Lifecycle Manager components access to a docker repository is required, the settings below can be used for local testing use:
```bash
export DOCKER_REGISTRY_PORT=5000
export DOCKER_REGISTRY="localhost:${DOCKER_REGISTRY_PORT}"
```

## Create kind cluster
Provision a kind cluster with access to a local docker repository:
```bash
./scripts/kind-with-registry.sh "kind-registry" "${DOCKER_REGISTRY_PORT}"
```
Then follow the [instructions here](../../docs/kind-deployment.md), excluding the step already performed to provision the kind cluster.

## Build and Package Gaffer Operator
Generate and publish the gaffer helm operator artifacts using this script:
```bash
./scripts/generate-helm-operator.sh "${DOCKER_REGISTRY}"
```

## Install Operator Lifecycle Manager:
```bash
operator-sdk olm install
```

## Configure Operator Lifecycle Manager
Create a Catalog Source which points to the operator index image, an operator group and subscription:
```bash
OPERATOR_INDEX_IMAGE="${DOCKER_REGISTRY}/gchq/gaffer/gaffer-operator-index:${GAFFER_VERSION}"

kubectl apply -f - << EOF
apiVersion: operators.coreos.com/v1alpha1
kind: CatalogSource
metadata:
  name: gaffer-operator-catalog
  namespace: olm
spec:
  sourceType: grpc
  image: ${OPERATOR_INDEX_IMAGE}
  displayName: Gaffer Operator Catalog
EOF

# Create Operator Group
kubectl apply -f - << EOF
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: gaffer-operator-group
  namespace: default
EOF

# Create subscription
kubectl apply -f - << EOF
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: gaffer-operator-subscription
  namespace: default
spec:
  channel: alpha
  name: gaffer-operator
  source: gaffer-operator-catalog
  sourceNamespace: olm
EOF
```

## Deploy a graph using the examples:
```bash
kubectl apply -f examples/insecure_values.yaml
```

## Undeploy graph:
```bash
kubectl delete -f examples/insecure_values.yaml
```