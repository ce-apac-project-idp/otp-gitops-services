# README


## Assumptions

IBM Secret Manager is the secret instance deployed here.


## Procedure

Navigate to secrets manager and create a secret of type "Other secret type". The name field has to be **provideraws** as per the secretKey reference in the secret.yaml file found in this directory. Underscores and hyphens are special characters and will result in errors. Alphanumeric characters only.

The secret value is off the following form:

```
[default]
aws_access_key_id = <aws_access_key>
aws_secret_access_key = <aws_secret_key>
```

Replace accordingly.