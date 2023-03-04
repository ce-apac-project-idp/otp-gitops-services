# README


## Requirements

1) The Kubeseal binary must be downloaded and installed in your local machine
2) You must be logged on to your cluster
3) The SealedSecret controller must already be running on your cluster (This is instantiated in the Infra repo)
4) An instance of IBM secrets Manager created in IBM Cloud
5) An API Key


## Procedure

Run the following command first, replacing the api key where instructed:

```
kubectl create secret --dry-run=client generic secrets-manager -n external-secrets --from-literal=apiKey=api_key_here -o yaml > sm.yaml
```

This will create the secret YAML locally but not submit this to the Kubernetes API thanks to the dry run parameter.

Now, run the command given below:

```
kubeseal --controller-name=sealed-secrets --controller-namespace=sealed-secrets -o yaml < sm.yaml > sealed-secret.yaml && mv sealed-secret.yaml overlays/default && rm sm.yaml
```


This will create the necessary resource in the required directory and perform the appropriate cleanup. The sealed-secret.yaml generated can safely be pushed to Git.


Change the serviceUrl field in the overlays/default/cluster-secret-store.yaml file accordingly.