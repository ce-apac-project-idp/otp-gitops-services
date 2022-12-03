# Reference field in Kustomization

K8 enforces an upper bound to the number of characters constituting a value field in the spec. The reference field in the metadata section should be prefixed by the following url:

```
https://github.com/tektoncd/catalog/blob/
```

Concatenate this prefix to the reference field provided, bearing in mind to replace instances of "-" with "/". You should now arrive at the source YAML.

The better approach for the above is to simply reference a remote YAML as given by the commented out resources field in the kustomization file found in the base directory. But I get YAML parse errors. Perhaps a cleaner approach would be is to store all the tekton tasks in another dedicated task repository internal to this organisation and reference them as such. 

Next steps:

1) See point raised above

