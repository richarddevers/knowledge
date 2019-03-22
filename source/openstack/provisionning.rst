Provisionning overview
***********************

Starting with the identity service, the following steps summarize briefly the provisioning

workflow based on API calls in OpenStack:

Calling the identity service for authentication

Generating a token to be used for subsequent calls

Contacting the image service to list and retrieve a base image

Processing the request to the compute service API

Processing compute service calls to determine security groups and keys

Calling the network service API to determine available networks


Choosing the hypervisor node by the compute scheduler service

Calling the block storage service API to allocate volume to the instance

Spinning up the instance in the hypervisor via the compute service API call

Calling the network service API to allocate network resources to the instance