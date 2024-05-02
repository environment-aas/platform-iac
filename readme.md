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
export gitops_repo=https://github.com/environment-aas/platform-iac.git #<your newly created repo>
export cluster_name=hub #<your hub cluster name, typically "hub">
export cluster_base_domain=$(oc get ingress.config.openshift.io cluster --template={{.spec.domain}} | sed -e "s/^apps.//")
export platform_base_domain=${cluster_base_domain#*.}
oc apply -f .bootstrap/subscription.yaml
oc apply -f .bootstrap/cluster-rolebinding.yaml
envsubst < .bootstrap/argocd.yaml | oc apply -f -
envsubst < .bootstrap/root-application.yaml | oc apply -f -
```

To get the prod and non-prod cluster created you'll have to prepare a secret in the way ACM expects it. Here is an example of this secret required by ACM.

```yaml
apiVersion: v1
stringData:
  additionalTrustBundle: ""
  aws_access_key_id: "-----"
  aws_secret_access_key: "-----"
  baseDomain: "the root domain of the SOA Record in your DNS"
  httpProxy: ""
  httpsProxy: ""
  noProxy: ""
  pullSecret: "copy yours from https://console.redhat.com/openshift/downloads#tool-pull-secret"
  ssh-privatekey: "-----"
  ssh-publickey: "-----"
kind: Secret
metadata:
  labels:
    cluster.open-cluster-management.io/credentials: ""
    cluster.open-cluster-management.io/type: aws
  name: aws-credentials
  namespace: open-cluster-management
type: Opaque
```

copy&paste inside `./.rosa/aws-secret.yaml`

then run:

```sh
oc delete secret aws-credentials -n open-cluster-management
oc apply -f ./.rosa/aws-secret.yaml
```

> NOTE: you can also use the ACM Console to create it following the UI guided form. Go to https://your-console-url/multicloud/credentials, and select your cloud provider (AWS in our case).

> NOTE: after creating this `aws-secret` you may also "force" Argo to reconsile the `ClusterDeployment` resource for bothe `prod` and `non-prod`. To do that go to ArgoCD Console open the `non-prod` Application and manually delete the `clusterdepoyment` resource. This will force Argo reconsile it and start the cluster provisioning job.
> 

To deploy RHDH run the following (you need to have prepared the secret)
```sh
oc delete secret rhdh-pull-secret -n redhat-developer-hub
oc delete secret github-pat -n redhat-developer-hub
oc create -f ./clusters/hub/overlays/redhat-developer-hub/rhdh-pull-secret.yml -n redhat-developer-hub
```

