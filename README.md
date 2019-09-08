# commands_tips
Tip & sintax command used in Unix and Windows

# Table of Contents
1. [Openshift](#openshift)


<a name="openshift" />

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

* create a new project
  * oc new-project <name>
 
* grant access project to a user
  * oc adm policy add-role-to-user <admin|basic-user|view|edit|system:deployer|system:image-builder|system:image-puller|system:image-pusher> <user> -n <project-name>

