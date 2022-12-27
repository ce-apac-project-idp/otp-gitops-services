# Uninstall RHACS

To uninstall ACS instances (central, secured cluster, etc) and ACS operator, 
follow the documentation [here](https://docs.openshift.com/acs/3.73/installing/uninstall-acs.html)

But you might find that namespaces are stuck in deletion 
because of Centrals, SecuredClusters CR and CRD remain still. 

To delete those, try deleting finalizers in
 - centrals.platform.stackrox.io CRD
 - securedclusters.platform.stackrox.io

After deleting CRDs and CRs, verify namespace's deletion.
