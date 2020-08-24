
# CP4I Command-Line CheatSheet

This is a cheatsheet for working with IBM Cloud Pak for Integration. The following commands are mostly geared towards cluster administrators, but some commands may also be useful for CP4I users.

For version `2020.1` and below, common services reside in the `kube-system` namesapce. For version `2020.2`, common services reside in namespace `ibm-common-services`. Commands targeting the `kube-system` namespace may need to target `ibm-common-services` instead for the latest CP4I release.

### Table of Content

- [CP4I Info](#cp4i-info)
- [cloudctl](#cloudctl)
- [cloudctl for Managing Event Streams](#cloudctl-for-managing-event-streams)
- [Admin Credentials](#admin-credentials)
- [Secrets for Deploying Capabilities](#secrets-for-deploying-capabilities)
- [Handy Helm Commands](#handy-helm-commands)
- [Openshift Internal Registry](#openshift-internal-registry)
- [Accessing IBM Entitled Registry](#accessing-ibm-entitled-registry)
- [Checking Images in Docker Registries](#checking-images-in-docker-registries)
- [EventStreams REST API](#eventstreams-rest-api)
- [CLI Autocomplete Setup](#cli-autocomplete-setup)
- [Add SCC to User or Group](#add-scc-to-user-or-group)

## CP4I Info

```
# Shows CP4I endpoints, ports, and version

# 2020.2
oc get configmap ibmcloud-cluster-info -o yaml -n ibm-common-services

# 2020.1
oc get configmap ibmcloud-cluster-info -o yaml -n kube-public
```

## cloudctl

```
# Login with fetched route
cloudctl login -a $(oc get routes -n kube-system icp-console -ojsonpath='{.spec.host}') -n eventstreams --skip-kubectl-config

# Or single command to login as default CP4I admin 
cloudctl login -n integration -a $(oc get routes -n kube-system icp-console -ojsonpath='{.spec.host}') -u admin -p $(oc get secrets -n kube-system platform-auth-idp-credentials -ojsonpath='{.data.admin_password}' | base64 --decode ) --skip-kubectl-config

# List local charts pushed in if offline install
cloudctl catalog charts --repo local-charts

# Manually push capability archive if inside the installer icp4icontent directory
cloudctl catalog load-archive \
    --archive IBM-DataPower-Virtual-Edition-for-IBM-Cloud-Pak-for-Integration-1.0.5.tgz \
    --image-registry $IMG_REGISTRY_ROUTE/datapower \
    --chart-registry local-charts
    
# Change default admin password
cloudctl pm update-secret kube-system platform-auth-idp-credentials -d admin_password=notadmin
```

## cloudctl for Managing Event Streams

Public download link for ES plugin. (Without ability to log in as default CP4I cluster admin, this comes in handy). It is backwards compatible.

http://public.dhe.ibm.com/ibmdl/export/pub/software/websphere/messaging/eventstreams/cli-plugin/

```
# Choose instance if there are multiple deployed
cloudctl es init -n eventstreams

# Permissions need to be added to IAM team in common services for new ES instances
# to access the ES UI properly. Otherwise you will see 403 error even if user has
# cluster admin role via IAM teams

cloudctl iam teams

# Install if it isn't already 
cloudctl plugin install <es-plugin-path>

cloudctl es init -n eventstreams

# Add the team to the ES release
cloudctl es iam-add-release-to-team --team "Team Name"

# Download the ES cert using the CLI instead of the UI

# JKS
cloudctl es certificates

# PEM
cloudctl es certificates --format pem
```


## Admin Credentials
```
# Getting default admin password

# 2020.2
oc get secrets -n ibm-common-services platform-auth-idp-credentials -ojsonpath='{.data.admin_password}' | base64 --decode && echo "" 

# 2020.1
oc get secrets -n kube-system platform-auth-idp-credentials -ojsonpath='{.data.admin_password}' | base64 --decode && echo ""
```


## Secrets for Deploying Capabilities
```
# Copying ibm-entitlement-key from one namespace to another
oc get secret ibm-entitlement-key --namespace=src-ns --export -o yaml | kubectl apply --namespace=target-ns -f -

# Get Secret for deploying local-chart into the target namespace
oc get secret -n mq | grep 'deployer-dockercfg'
```


## Handy Helm Commands

```
# Get local-charts repo URL from cloudctl
cloudctl catalog repos | grep local-charts | awk '{print $2}'

helm init --client-only

# Add the "local-charts" repo inside the cluster to Helm in the web terminal session
helm repo add local-charts https://icp-console.your.cluster.url:443/helm-repo/charts --ca-file /home/65551/.helm/ca.pem --cert-file /home/65551/.helm/cert.pem --key-file /home/65551/.helm/key.pem

helm repo list

helm search local-charts/ --versions

# Add entitled charts repo
helm repo add entitled-charts https://raw.githubusercontent.com/IBM/charts/master/repo/entitled/

# List all charts in local-chart repo (or any other repo you've added)
helm search local-charts

# Search for a specific chart
helm search local-charts/ibm-eventstreams-icp4i-prod

# Upgrade the number of brokers in the Kafka cluster
helm upgrade --reuse-values --set kafka.brokers=3 es-release-name local-charts/ibm-eventstreams-icp4i-prod --tls

# Get all the generated YAML from the helm release
helm get release-name --tls

# Get the helm values used to deploy the capability for a release
helm get values release-name --tls
```



## Openshift Internal Registry
```
# Get exposed registry route and store in a variable
IMG_ROUTE=$(oc get route default-route -n openshift-image-registry -ojsonpath='{.spec.host}')

# Login to internal registry with route. Username can be any value with no colons
docker login $IMG_ROUTE -u user -p $(oc whoami -t) 

# Example tag and push
docker tag cstsui/kafkaconnect-mq $IMG_ROUTE/kconnect-e2e/kafkaconnect-mq

docker push $IMG_ROUTE/kconnect-e2e/kafkaconnect-mq
```

## Accessing IBM Entitled Registry
```
# Entitlement key for deployment use
oc create secret docker-registry ibm-entitlement-key \
    --docker-username=cp \
    --docker-password=<api-key> \
    --docker-server=cp.icr.io \
    --namespace=my-namespace

# Login locally
docker login cp.icr.io --username cp --password <api-key>
```

## Checking Images in Docker Registries

```
# Docker registry
curl -X GET 'your-registry:5000/v2/_catalog?n=5000' | jq . - | tee registry.txt

# OCP Internal Registry
curl -k -u kubeadmin:$(oc whoami -t) -X GET https://default-route-openshift-image-registry.ocp.cluster/v2/_catalog?n=500
```


## EventStreams REST API
```
# Store the variables for a shorter curl
# All these info can be fetched from the ES UI Home page
REST_ROUTE=<rest_api_route>
ES_API_KEY=<api_key>
CA_CERT=<es_cert_pem_path>
TOPIC=<topic_name>

# REST API Route
oc get route -n eventstreams | grep rest-route

# REST API test
curl -k -v -X POST -H "Authorization: Bearer $ES_API_KEY" -H "Content-Type: text/plain" -H "Accept: application/json" -d 'test message' --cacert $CA_CERT "$REST_ROUTE/topics/$TOPIC/records"

# Simple loop to keep sending for a while for testing
for ((i=1;i<=500;i++)); do curl -k -X POST -H "Authorization: Bearer $ES_API_KEY" -H "Content-Type: text/plain" -H "Accept: application/json" -d "test message $i" --cacert es-cert.pem "$REST_ROUTE/topics/multipartition/records"; sleep 1; done
```

## Add SCC to User or Group

```
# Add SCC to default service account in "cp4i" Namesapce
oc adm policy add-scc-to-user restricted system:serviceaccounts:cp4i:default


# Add SCC to group of service accounts in "cp4i" namespace 
oc adm policy add-scc-to-group ibm-anyuid-scc system:serviceaccounts:cp4i
```


## CLI Autocomplete Setup

Add autocompletion for your shell. Skip the one's you already have or don't want/need. Note this may increase the time it takes to load your shell.

```
# zsh
echo 'source <(kubectl completion zsh)' >>~/.zshrc
echo 'source <(oc completion zsh)' >>~/.zshrc
echo 'source <(cloudctl completion zsh)' >>~/.zshrc
echo 'source <(helm completion zsh)' >>~/.zshrc

# Reload your current terminal with completion functions
source ~/.zshrc
```

```
# bash
echo 'source <(kubectl completion bash)' >>~/.bashrc
echo 'source <(oc completion bash)' >>~/.bashrc
echo 'source <(cloudctl completion bash)' >>~/.bashrc
echo 'source <(helm completion bash)' >>~/.bashrc

# Reload your current terminal with completion functions
source ~/.bashrc
```
