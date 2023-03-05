# README

## Context

All, or most, RHACM related resources are deployed in the "open-cluster-management" 

## Findings

An OpenShift Cluster (type - VPC2 with ODF) was deployed via TechZone to act as the Hub Cluster.

Deploying this operator out of the box will NOT work as the scheduler won't assign the pods bought about by the respective deployments to a node. This is due to the following reasons. This is despite the OpenShift Add on being installed successfully as per IBM Cloud. Note this did not deploy additonal nodes - presumably ones labelled and tained correctly as per below. Perhaps this is an issue with IBM Cloud?

All RHACM deployment resources defined the following toleration: "**key=node-role.kubernetes.io/infra, operator=Exists, effect=NoSchedule**". The nodes using the aformentioned cluster were not labelled as such. I was conscious not to apply the corresponding taint to all the nodes as this will result in all the workloads being ejected once re-scheduled. As an aside - recall the difference between NoSchedule and NoExecute. NoSchedule means the node in question will not be chosen on the subsequent reassignment whereas NoExecute evicts the pod immediately. The taint command is found below:

```
kubectl taint nodes  node_name_here  node-role.kubernetes.io/infra=:NoSchedule
```

Replace node_name_here accordingly. I tained two out of the five available nodes for this.

Next, the deployment resources in question contained the following NodeSelector: "**node-role.kubernetes.io/infra: ''**". The chosen nodes tainted will have to be labelled accordingly.

## Conclusion

TLDR:

1) Identify the nodes.
2) Taint said nodes.
3) Label said nodes.
4) Voila

This will allow the following ArgoApps to deploy successfully:

1) openshift-acm-instance
2) openshift-acm-cim
3) openshift-acm-observability
4) opebshift-acm-gitops-cluster
5) openshift-acm-discovery-instance
