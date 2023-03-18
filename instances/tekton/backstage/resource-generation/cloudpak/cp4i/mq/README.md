# README


## TODO

Right now, the MQ creation is relatively simple. The corresponding docker image in the artefacts repo uses a hardcoded YAML for the QM and MQSC configmap. I should be taking the user input and parsing said input using sed/yq and changing the QueueManager and ConfigMap YAML's, but this is not being done at the moment. This is a future action.

As a consequence of this, the Queue Manager, Queue, Channel and Sec Objects are hardcoded. Which means we need to hardcode these the values associated with said resources within the ACE app. Again, these ought to be decoupled as Configuration Objects in the future re ACE flow.

All the resources created here do NOT use the Kustomize framework. This ought to be done in the future.