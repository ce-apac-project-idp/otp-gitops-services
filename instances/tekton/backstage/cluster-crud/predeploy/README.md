# README

Prior to deploying an OpenShift cluster via RHACM, a public-private SSH key pair ought to be created. This applies to all hyperscalers.

The public component can be, as the name implies, public. As such, an Argo Application will be made out of this.

Run the following command:

```
ssh-keygen -t rsa
```

Follow the prompts to save this pair to a known location within your workstation. Base 64 decode the public component as such:

```
cat /path/to/public/component/here | base64
```

Finally, overwrite the "data.ssh-publickey" field of the public-key.yaml with the encoded output generated from the above command.
 
Refer to the README given in the directory one layer above for instructions on how, amongst other steps.