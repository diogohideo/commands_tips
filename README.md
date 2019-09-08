# commands_tips
Tip for commands used in Unix and Windows

# Table of Contents
1. [Openshift] (#Openshift)

## Openshift

* login
  * oc login
  * oc login --username developer --password developer
  * oc login --token=xxxxxxxxxxxxxxxxxxxxxx --server=https://xxxx.openshift.com
  * API - curl -H "Authorization: Bearer xxxxxxxxxxxxxxxxxxxxxx" "https://xxxx.openshift.com/oapi/v1/users/~"
  
* get oc user id
  * oc whoami

* get oc server url
  * oc whoami --show-server
  
* show project that can be accessed
  * oc get projects


