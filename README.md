<a href="http://tarantool.org">
   <img src="https://avatars2.githubusercontent.com/u/2344919?v=2&s=250"
align="right">
</a>

# Tarantool Kubernetes operator

The Tarantool Operator provides automation that simplifies the administration of [Tarantool Cartridge](https://github.com/tarantool/cartridge)-based clusters on Kubernetes.

The Operator introduces new API version `tarantool.io/v1alpha1` and installs custom resources for objects of three custom types: Cluster, Role, and ReplicasetTemplate.

##Table of contents

* [Resources](#resources)
* [Resource ownership](#resource-ownership)
* [Deploying the Tarantool operator on minikube](#deploying-the-tarantool-operator-on-minikube)
* [Example: key value storage](#example-key-value-storage)
  * [Application topology overview](#application-topology-overview)
  * [Running the example](#running-the-example)
  * [Scaling the example application](#scaling-the-example-application)
  * [Running tests](#running-tests)

## Resources

**Cluster** represents a single Tarantool Cartridge cluster.

**Role** represents a Tarantool Cartridge user role.

**ReplicasetTemplate** is a template for StatefulSets created as members of Role.

## Resource ownership

Resources managed by the Operator being deployed have the following resource ownership hierarchy:

![Resource ownership](./assets/resource_map.png)

Resource ownership directly affects how Kubernetes garbage collector works. If you execute a delete command on a parent resource, then all its dependants will be removed.

## Deploying the Tarantool operator on minikube

1. Install the required software:

    - [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl)

    - [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

1. Create a `minikube` cluster:

    ```shell
    minikube start --memory=4096
    ```

    You will need 4Gb of RAM allocated to the `minikube` cluster to run examples.

    Ensure `minikube` is up and running:

    ```shell
    minikube status
    ```

    In case of success you will see this output:

    ```shell
    host: Running
    kubelet: Running
    apiserver: Running
    ```

1. Create operator resources:

    ```shell
    kubectl create -f deploy/service_account.yaml
    kubectl create -f deploy/role.yaml
    kubectl create -f deploy/role_binding.yaml
    ```

1. Create Tarantool Operator CRD's;

    ```shell
    kubectl create -f deploy/crds/tarantool_v1alpha1_cluster_crd.yaml
    kubectl create -f deploy/crds/tarantool_v1alpha1_role_crd.yaml
    kubectl create -f deploy/crds/tarantool_v1alpha1_replicasettemplate_crd.yaml
    ```

1. Start the operator:

    ```shell
    kubectl create -f deploy/operator.yaml
    ```

    Ensure the operator is up:

    ```shell
    kubectl get pods --watch
    ```

    Wait for `tarantool-operator-xxxxxx-xx` Pod's STATUS to become `Running`.

## Example: key value storage

`examples/kv` contains a Tarantool-based distributed key value storage. Data are accessed via HTTP REST API.

### Application topology overview

![App topology](./examples/kv/assets/topology.png)

### Running the example

We assume that commands are executed from the repository root and Tarantool Operator is up and running.

1. Create a cluster:

    ```shell
    kubectl create -f examples/kv/deployment.yaml
    ```

1. Wait until the cluster Pods are up:

     ```shell
     kubectl get pods --watch
     ```

1. Access the cluster web UI:

   1. Get `minikube` vm ip-address:

       ```shell
       minikube ip
       ```

   1. Navigate to **http://MINIKUBE_IP** with your browser. Replace MINIKUBE_IP with the IP-address reported by the previous command.

1. Access KV API:

   1. Store some value:

       ```shell
       curl -XPOST http://MINIKUBE_IP/kv -d '{"key":"key_1", "value": "value_1"}'
       ```

   1. Access stored values:

       ```shell
       curl http://MINIKUBE_IP/kv_dump
       ```

### Scaling the example application

1. Increase the number of replica sets in Storages Role:

    ```shell
    kubectl scale roles.tarantool.io storage --replicas=3
    ```

    This will add one or more replica sets to the existing cluster.

    View the new cluster topology via the cluster web UI.

1. Increase the number of replicas across all Storages Role replica sets:

    ```shell
    kubectl edit replicasettemplates.tarantool.io storage-template
    ```

    This will open a text editor. Change `spec.replicas` field value to 3, then save and exit the editor.

    This will add one more replica to each Storages Role replica set.

    View the new cluster topology via the cluster web UI.

### Running tests

```shell
make build
make start
./bootstrap.sh
make test
```
