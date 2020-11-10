#!/bin/bash

# Copyright 2020 Crown Copyright
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

Provision a kind cluster with access to a local docker repository:

```bash
./scripts/kind-with-registry.sh
```

Then follow the [instructions here](../../docs/kind-deployment.md), excluding the step already performed to provision the kind cluster.

To generate and use Operator Lifecycle Manager components access to a docker repository is required, for local testing use:

```bash
export DOCKER_REPO="localhost:5000"
```

Generate and publish the gaffer helm operator artifacts using this script:

```bash
./scripts/generate-helm-operator.sh "${DOCKER_REPO}"
```

Install Operator Lifecycle Manager:

```bash
operator-sdk olm install
```

Create a Catalog Source which points to the operator index image, an operator group and subscription:

```bash
OPERATOR_INDEX_IMAGE="${DOCKER_REPO}/gchq/gaffer/gaffer-operator-index:${GAFFER_VERSION}"

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

Deploy a graph using the examples:

```bash
./generated/gaffer-operator/bin/kustomize build ./examples/overlays | kubectl apply -f -
```

Undeploy the graph:

```bash
./generated/gaffer-operator/bin/kustomize build ./examples/overlays | kubectl delete -f -
```