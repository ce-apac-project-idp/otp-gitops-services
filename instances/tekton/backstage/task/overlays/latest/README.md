# Reference field in Kustomization

K8 enforces an upper bound to the number of characters constituting a value field in the spec. The reference field in the metadata section should be prefixed by the following url:

```
https://github.com/tektoncd/catalog/blob/
```

Concatenate this prefix to the reference field provided, bearing in mind to replace instances of "-" with "/". You should now arrive at the source YAML.

