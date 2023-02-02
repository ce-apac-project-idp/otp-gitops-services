# Attention

Intro to Tekton:

1) https://medium.com/hiredscore-engineering/getting-started-with-tekton-pipelines-part-1-container-image-build-and-push-5de1f96b0515
2) https://ibm.github.io/tekton-tutorial-openshift/lab1/6_create-pipeline-run/


Git Clone links:

1) https://github.com/tektoncd/catalog/tree/main/task/git-clone/0.9
2) https://github.com/tektoncd/catalog/tree/main/task/git-clone/0.9#cloning-private-repositories
3) https://github.com/tektoncd/catalog/tree/main/task/git-clone/0.9/samples (See the using results for obtaining the SHA)
4) https://tekton.dev/docs/how-to-guides/clone-repository/

Git Cli links:

1) https://github.com/tektoncd/catalog/tree/main/task/git-cli/0.4/tests
2) https://github.com/tektoncd/catalog/blob/main/task/git-cli/0.4/git-cli.yaml
3) https://github.com/tektoncd/catalog/tree/main/task/git-cli/0.4

## Git Cli Task - Notes

The main bulk* of the task happens between (not including) the following lines respectively:

```
git config --global user.name "$(params.GIT_USER_NAME)"
```

```
RESULT_SHA="$(git rev-parse HEAD | tr -d '\n')"
```

The logic is as such:

1) Check for the presence of claims in the request. Claims in this case map to resources that ought to be created in the corresponding cloud provider. These resources are infrastructure based resources, and hence ought to be created prior to the "native K8" resources.
2) In order to differentiate claims from non claims, we check for the presence of the word "compositionRef" in the spec itself. This can be changed in the future, for instance enforcing developers to put a specific label for claims (or having git rules do this for us automatically). At the time of writing however, this is how claims are identified.
3) The claims are pushed first to the apps repository, which triggers ArgoCD to create the necessary claim (which Crossplane uses to create the composite-resource). 
4) A wait is performed to check for the health of the composite resource prior to pushing the remaining "consumer" resources (as these resources will consume the composite resource). 
5) In the event claims are not present in the request, all yaml's (under the supposition ordering is not relevant here**) are pushed simultaneously in a "one shot" manner.

**I have written this script very very quickly. It is definitely not the best and can stand a refactor.**

**Perhaps this can be extended to support "native" resource dependencies. For instance, assuming a request contained a.yaml, b.yaml and c.yaml with respective key value pairs "order:1", "order:2" and "order:3". The script can make use of this information to create the resources in the order desired, namely by mapping to argocd sync waves, for instance. (Perhaps there are ArgoAPI's allowing one to identify the current sync wave, for instance.)**


## Git Cli Task - Notes

The latest iteration of this task is used, that is, at the time of writing, [v0.9](https://github.com/tektoncd/catalog/tree/main/task/git-clone/0.9#git-clone).

A key feature of this particular version is the fact this task is run as a non-root user. Because of this, it is imperative the pipelineRun spec contains the "FsGroup" security context as given [here](https://github.com/tektoncd/catalog/tree/main/task/git-clone/0.9#workspaces). Please do not forget this.





