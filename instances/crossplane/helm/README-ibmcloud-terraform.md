# Deploying resources using Crossplane 

## General / high level
The best examples for these process flows are housed in `ce-apac-project-idp/otp-gitops-services/instances/crossplane/helm/runtime/terraform` where the files are labeled in relevant deployment order. The Jobspec (or the 3rd file to be applied) is housed in `ce-apac-project-idp/otp-gitops-services/instances/crossplane/helm/postdeploy`

1. Create secrets for the relevant provider credentials 
    * In this case, secrets containing IBM Cloud API keys, and AWS IAM keys. 
    * The scope of the API keys should be relevant to what you are trying to provision. i.e. an IBM Cloud API key should have access to administrator roles for the Kubernetes Service resources, and access to COS, VPCs etc for the underlying requirements. 
2. Create a provider 
3. Update the provider by applying the configuration job, or by applying the relevant CRB with ***, and oc adm scc command
4. Apply the provider config 
5. create your resources (ibm cloud CRDs, or terraform CRDs)

## IBM Cloud 

To deploy resources on IBM cloud there is an IBM cloud specific provider. 

There are some **prerequisites**.

1. Firstly, a secret must exist that contains an IBM Cloud API key with the relevant IAM permissions. API keys and permissions can be managed from the [IAM section of the IBM Cloud Console](https://cloud.ibm.com/iam/overview). 
This secret can be applied using the External Secrets Operator, but should ultimately take the form of the below: 
```kind: Secret
apiVersion: v1
metadata:
  name: provider-ibm-cloud-secret
  namespace: crossplane-system
data:
  credentials: <b64 encoded IBM cloud API key here>
type: Opaque
```
2. Secondly, a provider must be created. Use the example provider spec provided, but be sure to reference a correct Dockerhub tag for the image, and reference the provider secret correctly. After this you may notice some issues with pods pending, or resources hanging. This is because the provider does not have correct permissions to spin things up. We need to apply a CRB, and an anyuid scc to the providers Service Account. An example CRB can be found [here](https://github.com/langley-2/crossplane-kubernetes/tree/main/ibm-cloud-provider-deployment/configuration-job/crossplane-provider-deployment) - be sure to modify the name of the service account before applying. 
    * After this, run `oc adm policy add-scc-to-user anyuid -z $provider` to resolve any uid issues. 

3. Assuming the prior is performed correctly, some pods should start spinning up in the crossplane namespace. It is at this point a providerconfig can be applied as the following: 
```
apiVersion: ibmcloud.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: ibm-cloud
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: provider-ibm-cloud-secret
      key: credentials
  region: us-south
```
    * note - the crossplane docs mention that currently, Cloud Object Storage APIs are only supported in us-south and east regions - hence us setting the region as us-south. If you notice your resources being created elsewhere you can update your target region using the IBM Cloud CLI (`ibmcloud target -r <region name>`)

4. Finally, you can create IBM Cloud resources.
    * Firstly, create a resource group to use (or use the default).
    * Secondly, create a VPC using the provided spec - modify the parameters.
    * Thirdly, create a subnet that uses the VPC as a reference using the manifest provided. 
    * Fourth, create an instance of Cloud Object Storage using the manifest provided. 
        * be sure to give your COS instance and bucket a globally unique name 
    * fifth - change the COS CRN and subnet ID in the IBM Cloud cluster spec provided, and apply this into your openshift cluster


**Debugging:**
* If you have any issues, run an oc describe on the resource that is giving you issues: 
    * i.e. `oc describe clusters.container.containerv2.ibmcloud.crossplane.io idp-crossplane-made-1` - this should give you some relevant errors to follow. 


## Terraform 
