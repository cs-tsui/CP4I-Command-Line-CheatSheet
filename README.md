# CP4I-Commandline-CheatSheet


## Helm CLI via Web Terminal

```
helm init --client-only

# Find home user directory
ls -lah ~/

# Use the user id
helm repo add local-charts https://icp-console.your.cluster.url:443/helm-repo/charts --ca-file /home/65551/.helm/ca.pem --cert-file /home/65551/.helm/cert.pem --key-file /home/65551/.helm/key.pem

helm repo list
``
