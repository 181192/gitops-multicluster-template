# Flux GitOps multicluster template
>This is a template project for inspiration on how to do GitOps. For more information about GitOps see notes from [my talk about IaC and GitOps](https://github.com/stacc-as/infrastructure-as-code/tree/master/kalli).

## #TL;DR
Ansible is used to setup and configure a new cluster. The ansible script will install some CRD's and the Flux Operator itself. After Flux's public SSH key is added to a Git account it will automatically start pulling from the Git repository and setup everything. The Flux operator will tag the Git log and commit back for changes it does etc. updating a Docker image version.

## Folder structure

```
.
├── ansible
├── charts
├── common
├── cluster_a
└── cluster_b
```

The folder structure of this repo:

- `ansible` folder contains Kubernetes manifests that cannot be automated with Flux. Typically CRD's and Flux itself will included here. The setup will use Ansible with spesific environments variables for each cluster.
- `charts` folder contains the Helm charts of our applications.
- `common` folder contains common Kubernetes manifests that will be used in all clusters, will typically contains monitoring HelmReleases and RBAC rules.
- `cluster_name` folders will contains HelmReleases, Namespaces and Workloads for a spesific cluster.

### A cluster folder structure
```
.
├── namespaces
├── releases
└── workloads
```

The folder structure of a cluster
- `namespaces` folder contains different namespaces that are to be created in the cluster.
- `releases` folder contains subfolders for each namespace that contains `HelmReleases`.
- `workloads` folder contains subfolders for each namespace that contains different Kubernetes manifests.

## Whats included?
Two minikbue clusters have been setup.

One with Nginx ingress and the other one with Istio.

The clusters have some monitoring tools setup:
- [Grafana](https://github.com/grafana/grafana)
- [Grafana Loki](https://github.com/grafana/loki)
- [Weave Scope](https://github.com/weaveworks/scope)
- [Prometheus Operator](https://github.com/coreos/prometheus-operator)
- [Kiali](https://github.com/kiali/kiali) (Istio only)
- [Jaeger](https://github.com/jaegertracing/jaeger) (Istio only)

## Setup for new clusters/minikube

Installation guides for new clusters and minikube are in the `ansible/README.md`.

## Guides for developers

A cluster should include both `commmon` and `cluster-name` folder. This is specified in the `values.cluster-name.yaml` file for Flux in `ansible/common/flux`.

To include the `common` folder the cluster requires RBAC to be enabled. If the cluster does not have RBAC enabled, this folder should not be included.

**Recommended tools:**

- [ansible-playbook](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [fluxctl](https://www.weave.works/blog/install-fluxctl-and-manage-your-deployments-easily)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [istioctl](https://istio.io/docs/setup/kubernetes/download/)
- [kubebox](https://github.com/astefanutti/kubebox)
- [kubectx](https://github.com/ahmetb/kubectx#kubectx1)
- [kubens](https://github.com/ahmetb/kubectx#kubens1)
- [jq](https://stedolan.github.io/jq/)

## Cheat sheet's

```
# Kiali dashboard
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001


# Weave dashboard
kubectl port-forward -n monitoring "$(kubectl get -n monitoring pod --selector=weave-scope-component=app -o jsonpath='{.items..metadata.name}')" 4040
```
