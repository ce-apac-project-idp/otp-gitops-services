# README


## Notice

Be mindful of the following:

1) This has to happen after successful instantiation of both the ExternalSecret Operator (200 at the time of writing) and an instance of the External Secret (203 at the time of writing)
2) Secrets need to be pre-created in your External Secret Store. We use IBM Secrets Manager for this. Refer to the next section.

## PreReqs - External Secrets Creation - AWS

This section concerns AWS, but similar lines of reasoning can be applied for other hyperscalers. The RedHat Pull Secret and SSH Private Key can be reused. That said, entirely new ones can be made too.

This assumes an instance of IBM Secrets Manager has been created. Please refer to the README given in the "external-secrets-instance" for more information. This also assumes an access and secret key pair created within AWS is created within your secret store, in addition to the RedHat Pull Secret and the private component of the key pair you created in the pre-requisite (predeploy) section.

Create the following keys in your secret store. In IBM Secrets Manager, the type is "other secret type":

1) awsAccessKey
2) awsSecretKey
3) openshiftPullSecret
4) openshiftSSHPrivateKey

Note down their respective ID's. Substitute the ID's as found the env section of the "secret-generator" step of the "generate-resources" Tekton Task. In the event you decided to use alternative names to those presented above, make the corresponding changes to the fields with names ending in **NAME** too.


## Decisions

Right now, the secrets generated in the respective secret creation steps within the resource generator task are classic K8 secrets. They NEED to be made ExternalSecrets prior to commiting. There are two possible ways this can be achieved. Note they both require the creation of the secrets in the External Secret Instance from the first place:

1) Individially make an ES resources from each required AWS credential and save that to the relevant namespace as defined in the cluster-crud namespace in the Infra AppProject. OC get the relevant resource in the relevant step and change the namespace field to reflect the desired namespace. The name of the relevant secret must be made known to the step via a parameter. The ID, URL and all other configs do not need to be templated in.
2) Do not create an ES resource for each AWS credential and create (dry run) them directly via a template YAML. This avoids having to duplicate the secret creation in the relevant cluster-crud namespace at the expense of introducing more parameters to the Step.

Option 2 was chosen. Please note the although the install config is a secret it does NOT contain sensitive information. And as such can be made a normal K8 secret. Since it is generated on the fly it cannot be made an External Secret anyhow. **Note this has since been done. The logic is baked in the "secrets-generator" step of the Tekton task.**


## TODO

1) Use the kustomize framework here: https://stackoverflow.com/questions/71948193/replace-contents-of-an-item-in-a-list-using-kustomize
2) Include known_hosts in the External secrets for git-cli-ssh as opposed to disabling strict host key checking via the "GIT_SSH_COMMAND" commmand. That was a cheap copout. More here: https://artifacthub.io/packages/tekton-task/tekton-catalog-tasks/git-cli#using-ssh-credentials



