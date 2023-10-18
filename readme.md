# GitOps standard folder layout for OpenShift multi-cluster day-two configurations

As the title suggest this standard layout is laser focused on addressing the infrastructure configurations (a.k.a. day-two) for a multi-cluster deployment of OpenShift.

TL;DR: jump to the [getting started](#getting-started-with-this-repo) section.

## Repo Structure

These are the main folders:

![Folders](.docs/media/folders.png "Folders")

### Components

This folder contains all of the root pieces of configurations. Each piece of configuration resides in its own subfolder. These components should never derive from anything (i.e. their resources and components lists in the `kustomization.yaml` file are empty).

![Components](.docs/media/components.png "Components")

### Groups

This folder contains common pieces of configurations that can be applied to a group of clusters. Groups can capture concepts like the purpose of the cluster (lab,non-prod, prod), the geography of the cluster (west, east, dc1, dc2), the security profile of the cluster (pci, non-pci, connected, disconnected) etc...

Groups are designed to be composable. So, based on the example groups from above, one should be able to express that a cluster belongs to the non-prod, east, pci groups.

Composition of groups is possible because groups are defined as kustomize [components](https://kubectl.docs.kubernetes.io/references/kustomize/kustomization/components/):

```yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component
```

The `all` group almost always exists and it captures the configuration that goes in every cluster.

Each group has it own folder. Within this folder, we can find two things: 

- A set of folders containing the group-specific overlays over some set of components.
- A root level kustomization that generates the ArgoCD applications for this group, using the [argocd-app-of-app](.helm/charts/argocd-app-of-app/) helm chart.

![Groups](.docs/media/groups.png "Groups")

In this example, in green you can see the overlays over one component, while in red you can see the resources needed to generate the Argocd Applications for this group.

### Clusters

This folder contains the cluster-specific configurations. As for groups it is made of two parts:

- A set of folders containing the cluster-specific overlays over some set of components.
- A root level kustomization that generates the ArgoCD applications for this group, using the [argocd-app-of-app](.helm/charts/argocd-app-of-app/) helm chart. This kustomization must import the correct groups for this cluster, here is an example:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

components:
  - ../../groups/all
  - ../../groups/non-prod
  - ../../groups/geo-east
```

## Design Decisions

In no particular order, here are the design decisions that guides us to this current folder structure.

- ArgoCD Applications for the [App of App pattern](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/#app-of-apps-pattern) are generated via a Helm chart. This was chosen over the ApplicationSet approach. The reason of this choice is that The ApplicationSet controller seems a bit unstable and that helm charts offer mode flexibly.
- Kustomize is the primary templating mechanism, if one needs to use helm charts, that is still possible via the Kustomize [HelmChartInflaterGenerator](https://kubectl.docs.kubernetes.io/references/kustomize/builtins/#_helmchartinflationgenerator_). The reason of this choice are that Kustomize is easy to pick up in general so starting with kustomize is easier for new users. Also having kustomize at the top level provides homogeneity. This without losing flexibly as one can always use helm charts. 
- Groups of clusters a modeled via Kustomize components, this way they can be composed as opposed to being inherited giving more flexibility.


## Getting started with this repo

Press the `Use This Template` button at the right top corner of this page and follow the github instructions to create a detached copy of this repo.

Once you have a copy of this repo in your organization, you have to seed your Hub cluster to point to this repo.

To do so you can simply run this commands, however you might want to implement these steps in different ways in your environment:

```sh
export gitops_repo=<your newly created repo>
export cluster_name=<your hub cluster name, typically "hub">
oc apply -f .bootstrap/subscription.yaml
oc apply -f .bootstrap/cluster-rolebinding.yaml
oc apply -f .bootstrap/argocd.yaml
envsubst < .bootstrap/root-application.yaml | oc apply -f -
```

Note: for pedagogical reason this repo contains some example of components, groups and clusters, you will have to likely remove these examples and start adding the configurations you actually need.

## Use cases

### component configuration pinning and promotion

### group configuration pinning and promotion

### cluster configuration pinning and promotion

### how to bootstrap a newly create/registered cluster with gitops

### how to pass cluster-level variables to all of the configurations

