```sh
oc new-project redhat-developer-hub
oc delete secret rhdh-pull-secret -n redhat-developer-hub
oc delete secret github-pat -n redhat-developer-hub
oc create -f ./clusters/hub/overlays/redhat-developer-hub/rhdh-pull-secret.yaml -n redhat-developer-hub
```