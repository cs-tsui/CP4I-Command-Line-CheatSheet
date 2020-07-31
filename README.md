# CP4I-Commandline-CheatSheet


## Cloudctl

```
cloudctl login -a $(oc get routes -n kube-system icp-console -ojsonpath='{.spec.host}') -n eventstreams --skip-kubectl-config

# Choose instance if there are multiple deployed
cloudctl es init -n eventstreams
```

## Cloudctl for Managing EventStreams
```
# Permissions need to be added to IAM team in common services for new ES instances
# to access the ES UI properly. Otherwise you will see 403 error even if user has
# cluster admin role via IAM teams

cloudctl iam teams

# Install if it isn't already 
cloudctl plugin install <es-plugin-path>

cloudctl es init -n eventstreams

cloudctl es iam-add-release-to-team --team "Team Name"
```

## Helm CLI via Web Terminal

```
helm init --client-only

# Find home user directory
ls -lah ~/

# Use the user id
helm repo add local-charts https://icp-console.your.cluster.url:443/helm-repo/charts --ca-file /home/65551/.helm/ca.pem --cert-file /home/65551/.helm/cert.pem --key-file /home/65551/.helm/key.pem

helm repo list

helm search local-charts/ --versions
```
