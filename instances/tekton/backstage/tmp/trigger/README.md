# Attention

It appears as though the default patch behaviour is to replace (not append) on lists when it comes to Kustomize.

The file patch.yaml should not have the following params defined:

1) targetEnv
2) repo-url
3) REQUESTOR_ID
4) APPLICATION_NAE


Essentially, anything with tt.params should be included in the base file and the other params to be passed in via patches as they are variable with each deployment.

Need to find an elegant way around this. Quick googling gave me the following:

1) https://blog.argoproj.io/argo-crds-and-kustomize-the-problem-of-patching-lists-5cfc43da288c
2) https://stackoverflow.com/questions/71622419/adding-items-to-a-list-with-kubectl-kustomize
3) https://blog.scottlowe.org/2021/07/07/adding-multiple-items-using-kustomize-json-6902-patches/
4) https://fabianlee.org/2022/04/18/kubernetes-kustomize-transformations-with-patchesstrategicmerge/


