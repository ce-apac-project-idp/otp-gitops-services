# README


## Context

All tekton pipelines would ultimately commit and push YAML resouces to the relevant GitOps repository, be it clusters or apps.

The configmap and secrets provided in this directory provide the context necessary to perform the relevant git commands.

## TODO

1) Use the Kustomise framework here.
2) The git-cli task does NOT enforce host checking prior to performing the git related commands. Ie, git push or git add. It is best to include the known hosts as a configuration present in the ExternalSecret as specified [here](https://superuser.com/questions/523099/linux-copy-all-files-by-extension-to-single-dirrectory) (known_hosts field). The temporary workaround involves setting the GIT_SSH_COMMAND to a value such that strictHostKeyChecking is disabled. The command is provided below.


```
export GIT_SSH_COMMAND="ssh -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no"
```

Alternatively include this prior to performing the git commmand (in the same line) if you wish not to export variables.

## Caveats

For the love of god, do NOT:

1) Change the "id_rsa" field to a different value in the ExternalSecret. Do not change it to anything else. Should you decide to change it, update the SSH global config file as given in the link above and add it as a field within the [ExternalSecret](https://superuser.com/questions/523099/linux-copy-all-files-by-extension-to-single-dirrectory)
2) Forget to include a newline at the end of the private key string prior to saving it to your secret store. Otherwise you will get LibCryto errors.
