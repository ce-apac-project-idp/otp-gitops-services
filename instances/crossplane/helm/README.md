# Crossplane Installation & Deployment

## Installation

Currently there exists two methods to run Crossplane on your OpenShift/Kubernetes Cluster:

1) [Helm deployment](https://artifacthub.io/packages/helm/crossplane/crossplane)
2) Operator based deployment through the OperatorHub in the OpenShift console (Upbound)

You may be tempted to use the operator based way of standing up crossplane related resources, and rightfully so. That said, this operator is provided by the community and is not fully mature yet. Although the installPlan runs to completion and custom resources including Providers and ProviderConfigs can now be consumed, many runtime errors were encountered, including: 

1) Provisioning an instance of a Provider causes the corresponding operator pod to consume an astounding amount of memory with lower and upper bounds of 1Gi and 2Gi respectively. OOM eventually causes the pod to terminate due to insufficient resources. Sadly, this memory consumption is not transient. Furthermore, no logs are provided by said operator pod to determine the root cause of this burst in memory. Attempts were made at maually modifying the corresponding ClusterServiceVersion spec to alter the memory and compute limits. This "works" in the sense further pod OOM terminations are prevented. That said, this is a band aid solution. Besides, downstream errors were encountered as discussed in the second point. On a similar note, the operator pod is allocated a fairly modest size for the cache provided by an emptyDir volume. This cache size had to be manually increased (via the CSV spec) otherwise the pod would get evicted to another node, where this "vicious cycle" of eviction/rescheduling repeats itself indefinitely. This resulted in the generation of a few hundred dangling evicted pods overnight.
2) The AWS Provider object would never reach a healthy status, no matter what version of the provider was used. Creating a Provider object results in the generation of a ProviderResouce object. Describing the "ProviderResource" yields the following error statement: **Deployment.apps "provider-id-here" is invalid: spec.template.spec.containers[0].image: Required value.**. Sadly, none of the CRD's provide an "image" field for an end user to populate (nor it shouldn't - this ideally should be abstracted away anyhow). Note I have not tried configuring an Azure provider but I suspect it most likely will not work either.

As such, I have included a "helm" subdirectory within this Crossplane directory in this repository. In the event the operator based method of deployment is more supported, mature and stable we may add, populate and use the corresponding directory. Until then, the Helm based method given above will be used.


### Installation - Helm

As given above, the relevant link can be found [here](https://artifacthub.io/packages/helm/crossplane/crossplane). Running the commands works smoothly in a Kubernetes Cluster. SCC restrictions for pods running on OpenShift clusters renders the default Helm installation to fail. This is due to the fact the defaultValue for the UID/GID is hardcoded to 65536 as given in the link above. The allowed UID/GID range in OpenShift is provided by the namespace the pod resides in. This is not known in advance and thus cannot be dynamically modified. Fortunately, this can be circumvented by updating the relevant entries the values.yaml to allow any integers for the UID/GID pair. Specifically, the **securityContextCrossplane** and **securityContextRBACManager** fields were set to null. This successfully provisions  the relevant operator pods, and exposes the CR's such as Provider and ProviderConfigs we wish to consume.


### Installation - Operator

TODO: Populate when operator based method of deployment works.


### Deployment - Helm

Deploying an instance of a Provider spins up the associated pod (deployment). It will, however, error out for the following reasons:

1) Anyuid is required by the pod's main process
2) R/W privileges required for certain Kubernetes objects

Behind the scenes, a serviceAccount whose name coincides with the deployment is created and used by the pod in question. This serviceAccount needs to be elevated to resolve the errors given above. That said, the name of the deployment, and as a result the serviceAccount is not known in advance and so cannot be made declarative in an upstream ArgoApplication with a lower sync wave. A custom job would have to be written and triggered at the appropriate time. That is, **immediately after** deploying an instance of a Provider. Successfully deploying an instance of the provider exposes the relevant cloud resources. This job will:

1) Fetch the name of the ServiceAccount.
2) Have a CRB spec baked in the image with the associated role already Git'Opsed upstream. A sed command will be performed to replace the relevant field in the CRB to reference the correct ServiceAccount identity, given a known string to replace.
3) Create the CRB.
4) Relax the SCC bound to this serviceAccount by performing the following command: **oc adm policy add-scc-to-user anyuid -z system:serviceAccount:project/namespace:name-of-sa-here**
5) Scale down the deployment.
6) Scale up the deployment.
7) Wait for both the provider deployment AND provider revision to be healthy.

This job can be found in the jobs repository [here](https://github.com/ce-apac-project-idp/configuration-jobs). It goes without saying another role/rolebinding pair is required for the serviceAccount running this job.

### Deployment - Operator

TODO: Populate when operator based method of deployment works.

## RBAC

Irrespective of the installation method, the relevant serviceaccounts would need to be elevated to allow R/W permissons on crossplane resources. The "custom-argocd-cluster-argocd-application-controller" as created when Argo is provisioned needs to be extended as seen in the first few stanza entries under rules. Finally, the 2-services and 3-app yaml files would also need to have the relevant resources whitelisted.  

In relation to XRD's and XR's, assume you've defined a new object called x found in api group a.b.c. The serviceaccount named "crossplane" in the crossplane-system namespace need to have CREATE,LIST,WATCH,GET,PATCH priveleges on said object. Better yet, creating a role allowing said access to the entire api group is probably a more efficient way of doing it.
## Next steps - Helm

For rapid iteration and testing purposes we did not pin down the restrictions (given by the ClusterRole objects) according to the principle of least privilege. The roles corresponding to the Provider deployment and the custom job are given in te following locations respectively:

1) [Provider Role](https://github.com/ce-apac-project-idp/otp-gitops-infra/blob/main/rbac/crossplane/providers/aws/clusterrole.yaml)
2) [Job Role](https://github.com/ce-apac-project-idp/otp-gitops-infra/blob/main/rbac/crossplane/providers/aws/job/role.yaml)
3) We need to make a role and rolebinding for the "crossplane" serviceaccount to allow R/W access to all the resources within the api group a.b.c assuming a.b.c is the api group housing the custom resources consumed as claims by the end user. Probably add this as an ArgoApp with the same sync wave as the job executed above, since the serviceaccount will not created until then.

The provider role, no more and no less, should contain the following:

1) Anyuid
2) R/W privileges for the aws specific resources. Specifically, those found in the *.aws.crossplane.io apiVersion.

In relation to point 2 above, perform the following command: 

```
oc api-resources | grep crossplane > resources.log
```

All the required should be present in the third column. The role named "aws-provider-role" created within the Infra currently has the admin role assigned to it. This role should contain the aforementioned resources. Sadly wildcards are not supported.

The Job role, no more and no less, should contain the following:

1) GET/LIST/WATCH Deployments and ServiceAccount
2) CREATE ClusterRoleBindings
3) CREATE SCC
4) PATCH Deployments
5) GET/LIST/WATCH Providers and ProviderRevisions
6) GET/LIST Namespaces