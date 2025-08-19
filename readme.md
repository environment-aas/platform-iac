# Environment as a Service

This repo contains a set of gitops configuration to setup a developer platform based on OpenShift.

This repo mainly focuses on the Environment as a Service use case.

This repo tries to showcase a realistic scenario featuring:

1. a multi cluster setup: we have three cluster at play, hub, non-prod prod. Other clusters can be easily added.
2. a multi tenant setup: we have two teams configured: team-a, team-b. More teams can be easily added.
3. a multi environment setup: each application can have multiple environments. In this example we have: dev, qa (running in the non-prod cluster) and prod (running in the prod cluster)

This is a gitops repo build from the red hat CoP [starter template](https://github.com/redhat-cop/gitops-standards-repo-template) and populated ion large part with elements from the red hat CoP [gitops catalog](https://github.com/redhat-cop/gitops-catalog)

This example was built starting from the [AWS ROSA Open Environment](https://demo.redhat.com/catalog?search=rosa&item=babylon-catalog-prod%2Fsandboxes-gpte.rosa.prod) [Red Hat demo catalog](https://demo.redhat.com/catalog) item, you might have to adjust pieces of the configuration if you start from a different place. Adjustments should be minimal. 

To get started run the following:

```sh
export gitops_repo=https://github.com/josephassiga/platform-iac.git #<your newly created repo>
export cluster_name=hub #<your hub cluster name, typically "hub">
export cluster_base_domain=$(oc get ingress.config.openshift.io cluster --template={{.spec.domain}} | sed -e "s/^apps.//")
export platform_base_domain=${cluster_base_domain#*.}
oc apply -f .bootstrap/subscription.yaml
oc apply -f .bootstrap/cluster-rolebinding.yaml
envsubst < .bootstrap/argocd.yaml | oc apply -f -
envsubst < .bootstrap/root-application.yaml | oc apply -f -
```

To get the prod and non-prod cluster created you'll have to prepare a secret in the way ACM expects it, then run:

```sh
oc new-project open-cluster-management
oc delete secret aws-credentials -n open-cluster-management
oc apply -f ./.rosa/aws-secret.yaml
```

To deploy RHDH run the following (you need to have prepared the secret)
```sh
oc new-project redhat-developer-hub
oc delete secret rhdh-pull-secret -n redhat-developer-hub
oc delete secret github-pat -n redhat-developer-hub
oc create -f ./clusters/hub/overlays/redhat-developer-hub/rhdh-pull-secret.yaml -n redhat-developer-hub
```

