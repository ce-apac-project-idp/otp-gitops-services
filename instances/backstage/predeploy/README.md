# External Secrets

## TLDR;
1. Create an external secret holding JSON object like
```
{ "key1": "value1", "key2":"value2" }
```
2. Use `spec.dataFrom` to load all keys in the JSON object.
```
spec: 
  dataFrom :
  - extract: 
      key: external-key-id
```

## Using `data`
Previously, we used to define `ExternalSecret` to refer to different secrets per key.
For example,
```
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
...
spec: 
  data:
  - secretKey: clientId
    remoteRef: 
      key: abcd  <-- id for an arbitrary secret
  - secretKey: clientSecret
    remoteRef:
      key: efgh <-- id for another arbitrary secret
  target:
    name: my-secrets
    template:
      type: Opaque
      data:
        clientId: |-
          {{ .clientId | toString }}
        clientSecret: |-
          {{ .clientSecret | toString }}
```
On IBM Secrets Manager, you have

| name         | type      | id   | containing |
|--------------|-----------|------|------------|
| clientId     | arbitrary | abcd | "123"      |
| clientSecret | arbitrary | efgh | "456"      |
 
In this case, you have to make sure to **use camelCase** in `spec.data[].secretKey`.
In practice, we found using dash(`-`) in `secretKey` wouldn't be picked up properly.

## Using `dataFrom`

The approach above works just well, but you have to register as many secrets as your apps refer to.
For example, if you want to use 10 env vars from secrets, you have to have 10 secrets in the vault and their corresponding IDs.

[This document](https://external-secrets.io/v0.6.1/guides/all-keys-one-secret/) shows how to extract
multiple secret keys from one external secret. 

```
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
...
spec: 
  dataFrom :
  - extract: 
      key: abcd  <-- key to secret holding several key-value pairs
  target:
    name: my-secrets
```
On IBM Secrets Manager, you have

| name     | type      | id   | containing                                   |
|----------|-----------|------|----------------------------------------------|
| any-name | arbitrary | abcd | { "clientId": "123", "clientSecret": "456" } |

Using `dataFrom` will automatically load all keys as it parses external secret as JSON object.

> Note!  
> Using `key-value` type in IBM Secrets Manager would not work with ArgoCD

### Clarifying keys
If you want to manipulate secret keys or just want to be verbose,
you can write `spec.target.template` as below.

```
template:
  type: Opaque
  data:
    clientId: "{{ .clientId }}" <-- specify which key from which entry
    clientSecret: "{{ .clientSecret }}"
    echoString: "echo {{ .clientId }}" <-- make a new key
```

