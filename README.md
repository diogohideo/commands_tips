# commands_tips
Tip & sintax command used in Unix and Windows to several platforms useful to DevOps.

# Table of Contents
1. [Jenkins](#jenkins)
1. [Openshift](#openshift)
   1. [Policies, Access and Roles](#policies)
   1. [Setup LDAP and other changes on OCP Server](#ocp_setup)
   1. [Connect to Pod Terminal](#pod-terminal)
   1. [Kibana - Troubleshooting](#kibana)
   1. [Metrics](#metrics)
   1. [S2I - Source to Image](#s2i)
   1. [Odo](#odo)
   1. [Troubleshooting](#troubleshooting)
         1. [Login error using LDAP after applying new changes](#login_error_ldap)
         1. [Pods on Pending Status - 0/X nodes are available: X Insufficient memory, X node(s) had taints that the pod didn't tolerate](#ocp_insuficient_memory)
         1. [Frontend - new image is not being pushed and the page is showing a default ngnix Openshift content](#frontend_ocp_empty_container)
1. [Docker](#docker)
   1. [Troubleshooting](#troubleshooting_docker)
      1. [X509: certificate signed by unknown authority](#certificate_error)
      1. [dial tcp: lookup <docker_registry_url> on XXX.XXX.XXX.XXX:XX: no such host](#dial_tcp)
1. [Ansible](#ansible)
1. [Git](#git)
1. [AWS](#aws)
   1. [Bucket (S3)](#s3)
1. [Unix Commands](#unix)
   1. [Most common Unix Commands](#unix-common-cmd)
   1. [apt-get to MacOS - Why use HomeBrew Instead](#apt-get-mac)
1. [Utilities](#utilities)
   1. [Exporting CA Certificates](#certificates)
   1. [Test connection telnet protocol using CURL](#curl_telnet)

<a name="jenkins"/>

# Jenkins

Useful commands to troubleshoot on Jenkins and other tips.

* Show the output comand on http://<jenkins_url>/script:
```groovy
def sout = new StringBuilder(), serr = new StringBuilder()
def proc = 'ls /badDir'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $serr"
```

<a name="openshift" />

# Openshift

* oc login: Log in to your OpenShift cluster and save the session token for subsequent use. You will be prompted for the user name and password or directed to a web console page where you can obtain a token you must then use to use to login from the command line. The web page will require you to first login in to the web console if you are not already logged in.<br>
* oc login \<server>: Log in to a specific OpenShift cluster. You will need to specify the name of the server as argument the first time you are using it, or if switching back to it after having used a different cluster.<br>
* oc login --username \<user>: Log in to your OpenShift cluster as a specific user.<br>
* oc login --username \<username> --password \<password>: Log in to your OpenShift cluster as a specific user and supply the password on the command line. Note that this is not recommended for real systems as your password may be logged or retained in command history.<br>
* oc login --token \<token>: Log in to your server using a token for an existing session.<br>
* oc login -u \<user> -p \<pass>
* oc logout: Log out of the active session by clearing the session token.<br>
* oc whoami: Show the name of the user for the current login session.<br>
* oc whoami --token: Show the token for the current login session.<br>
* oc whoami --show-server: Show which OpenShift cluster you are logged into.<br>
* oc whoami --show-context: Shows the context for the current session. This will include details about the project, server and name of user, in that order.<br>
* oc config get-clusters: Show a list of all OpenShift clusters ever logged in to.<br>
* oc config get-contexts: Show a list of contexts for all sessions ever created. For each context listed, this will include details about the project, server and name of user, in that order.<br>
* to a deployed app guestbook, how to attach the config map "default" using comandline:
<br/>oc set env --from=configmap/config-default dc/guest-book: update the environment variable from a config map
* oc get nodes - check node availability
* oc policy add-role-to-user view system:serviceaccount:myproject:default: add view access to default service account on a project
* oc get all -o name: list all resources has been created to the project
* oc get all --selector app=\<label-value> -o name: shows all resources assigned to a label created during route creation
* oc delete all --sector app=\<label-value>: delete all resource related to specified label
* oc new-app --search openshiftkatacoda/blog-django-py - checks if a image is valid
* oc new-app openshiftkatacoda/blog-django-py: deploy a image
* oc expose service/blog-django-py - expose the deployed image to public
* oc get route/blog-django-py - get the external route
* oc get imagestream -o name - list all image pulled into openshift
* oc describe imagestream/blog-django-py - show complete information of pulled image
* oc new-project \<proj-name>
* oc delete project \<proj-name>
* oc import-image openshiftkatacoda/blog-django-py --confirm: import the image into the openshift without deploying it
* oc new-app blog-django-py --name blog-1 - deploy an existing image from openshift
* oc delete all -l app=\<application name> - delete a existing project application

<a name="policies" />

## Policies, Access and Roles
Commands to setup roles.
* list all roles:
<br/>oc get groups
* list all commands:
<br/>oc policy -h
* assign edit / view to a group
<br/>oc policy add-role-to-group edit \<group>  -n <project/namespace>
* Add another user to a project so that they can work within the project, including creating new deployments or deleting applications.
<br/>oc adm policy add-role-to-user edit \<username> -n \<project>
* Add another user to a project but such that they can only view what is in the project.
<br/>oc adm policy add-role-to-user view \<username> -n \<project>
* Add another user to a project such that they are effectively a joint owner of the project and have administration rights over it, including the ability to delete the project.
<br/>oc adm policy add-role-to-user admin \<username> -n \<project>
* grant access project to a user
<br/>oc adm policy add-role-to-user \<admin|basic-user|view|edit|system:deployer|system:image-builder|system:image-puller|system:image-pusher> \<user> -n \<project-name>
* grant access project to a group (example):
<br/>oc policy add-role-to-group edit Openshift_HML_CAN_Edit -n can-hml
* grant cluster admin
<br/>oc adm policy add-cluster-role-to-user cluster-admin \<user>

<a name="ocp_setup" />

## Setup LDAP and other changes on OCP Server
Usually there are more than one master on Openshift cluster composition. Each node should be configured to change has effect. Default configuration file can be found on path /etc/origin/master-config.yaml.
```yaml

#configuration OCP sample
oauthConfig:
  assetPublicURL: https://ocm-manager.org/console/
  grantConfig:
    method: auto
  identityProviders:
  - challenge: true
    login: true
    mappingMethod: claim
    name: LDAP
    provider:
      apiVersion: v1
      attributes:
        email:
        - mail
        id:
        - dn
        name:
        - cn
        preferredUsername:
        - cn
      bindDN: CN=svc_openshift_ldap,OU=service accounts,OU=Users,O=CORP
      bindPassword:
        file: bindPassword.encrypted
        keyfile: bindPassword.key
      ca: /etc/origin/master/LDAP-CORP_ldap_ca.crt
      insecure: false
      kind: LDAPPasswordIdentityProvider
      url: ldaps://10.80.30.61:636/ou=Users,O=CORP?cn?sub?(&(objectClass=inetOrgPerson)(employeeStatus=Active))
```
After some some change is done on configuration file, use the following commands on each node:
```
# Restart API Kubernetes services
master-restart api api

# Restart Controller services
master-restart controllers controllers
```

<a name="pod-terminal" />

## Connect to Pod Terminal
Access the terminal pod from console, following the steps:
```
# select the project
oc project <project-name>

# list the desired pod
oc get pods

# go into terminal
oc rsh <pod-name>
```

<a name="kibana" />

## Kibana - Troubleshooting
### Safe guard is red status - Gateway timeout on Kibana
First of all, access on console online the project openshift-logging on Openshift. On the overview, access the resource logging-kibana and access the external route to check if the service is available.
If the message "Gateway timeout is prompted out", follow the steps:
* Get the pod name of logging data master with hash on the left:
<br/>oc get pods | grep logging-es-data-master
* Check index status:
<br/>oc exec \<logging-es-data-master-hash-of-pod> -- curl -s --key /etc/elasticsearch/secret/admin-key --cert /etc/elasticsearch/secret/admin-cert --cacert /etc/elasticsearch/secret/admin-ca -HContent-Type:application/json  https://localhost:9200/_cat/indices?v 
* Check if the searchguard is RED as the example:
```
health status index
green  open   .kibana.bb615795cff20a34fd133d6a13e4a9c5a9ce5e57                    NjVWuumISCO_a8sXprkPaA   1   0          4            0     46.9kb         46.9kb
green  open   project.ngc-dev.10e561e1-d354-11e9-a92b-0050569374d9.2019.10.29     vZ6AvXWWTTC83ZMXSCDicQ   1   0        102            0    153.7kb        153.7kb
red  open   .searchguard                                                        Y1piRFOMR3iMyl1p1LF-_Q   1   0          5            0     33.1kb         33.1kb
green  open   project.fac-dev.c0615391-d353-11e9-a92b-0050569374d9.2019.10.30     0ZiMxg1lT7GDXv2d-PVXgQ   1   0       1805            0      2.1mb          2.1mb
green  open   .kibana.2df074c6d9620ca5a1e2d11ffa09997a6c78d4c9                    LDl-Bsx4SBeYID895tjPhA   1   0          1            1     26.4kb         26.4kb
green  open   .operations.2019.10.26                                              0uGsaJrlTLmWDhkj_fZF1Q   1   0   24592923            0     15.7gb         15.7gb
```
* If searchguard is red, re-deploy the logging data master. Try to access kibana. If the problem persists, re-deploy loggin-kibana too. This should solve the problem.
* Otherwise, delete the index with red state:
```
health status index
green  open   .kibana.bb615795cff20a34fd133d6a13e4a9c5a9ce5e57                    NjVWuumISCO_a8sXprkPaA   1   0          4            0     46.9kb         46.9kb

oc exec <logging-es-data-master-hash-of-pod> --  curl -s --key /etc/elasticsearch/secret/admin-key --cert /etc/elasticsearch/secret/admin-cert --cacert /etc/elasticsearch/secret/admin-ca -XDELETE 'https://localhost:9200/.kibana.bb615795cff20a34fd133d6a13e4a9c5a9ce5e57'
```
<br/> Note the parameter "https://localhost:9200/.kibana.bb615795cff20a34fd133d6a13e4a9c5a9ce5e57". The index name include also the initial "."

<a name="metrics" />

## Metrics
The metrics settings are all deployed on openshift-infra (pods, secrets, config maps, etc). To login, it is required to use system:admin. The password is not required, but is necessary to have certificate instead.
* oc login -u system:admin
* list the pods running the metrics components:
<br/>oc get pods -n openshift-infra
* list all the routes of Hawkular metrics:
<br/>oc get all -n openshift-infra

#### Setting Horizontal Pod Autoscaler (HPA)
For this section, let's use guestbook application as example:
* set HPA to trigger autoscaling when cpu usage hits 20% of usage:
<br/> oc autoscale dc/guestbook --min 1 --max 3 --cpu-percent=20
* limit cpu to 80 mili cores
<br/>oc patch dc/guestbook -p '{"spec":{"template":{"spec":{"containers":[{"name":"guestbook","resources":{"limits":{"cpu":"80m"}}}]}}}}'


<a name="s2i" />

## S2I - Source to Image
Creates a images from a source.
* oc new-app <image:tag>~<source-code> \--name <name>: Deploy an application from source code hosted on a Git repository using the specified S2I builder image.
<br/> ex: oc new-app python:latest~https://github.com/openshift-katacoda/blog-django-py --name blog
<br/> python:latest - specify the language and tag
<br/> url - define the source code to be built
<br/> name - define the label of the application
* oc logs bc/blog -f: show the building progress log
* oc expose service/blog: expose the specified resource
* oc get route/blog: use this command to get the route url
* oc start-build blog: triggers a build to the related application (blog)
* oc start-build blog --from-dir=. --wait: triggers a build using local code
* oc describe bc/blog: show build configuration info
* oc cancel-build <build>: Cancel a running build.
* oc get builds: Display a list of all builds, completed, cancelled and running.
* oc get builds --watch: Monitor the progress of any active builds.
* oc get pods --watch: Monitor any activity related to pods in the project. This will include pods run to handle building and deployment of an application.

<a name="odo" />

## Odo
Abstracts the kubernetes and Openshift concepts so that developer focus on code.
* login
<br/> odo login -u developer -p developer
* create project 
<br/> odo project create myproject
* grant access to application service
<br/> oc policy add-role-to-user view system:serviceaccount:myproject:default
* list catalog components
<br/> odo catalog list components
* First build the components: backend and frontend (for this example)
  * backend
    * build
      <br/> mvn package
    * create yaml config
      <br/> odo create java backend --binary target/wildwest-1.0.jar
    * view config
      <br/> odo config view
    * push the binary into Openshift
      <br/> odo push
    * check if application is running
      <br/> odo log -f
  * frontend    
    * create yaml config
      <br/> odo create nodejs frontend
    * view config
      <br/> odo config view
    * push the binary into Openshift
      <br/> odo push
    * check if application is running
      <br/> odo log -f
* link the components - backend x frontend
<br/> odo link backend --component frontend --port 8080
* expose component to public
  * create route to frontend
    <br/> odo url create frontend --port 8080
  * push the url exposure
    <br> odo push
* automatic push to any changes on a project
<br/> odo watch &

<a name="troubleshooting" />

## Troubleshooting

<a name="login_error_ldap" />

### Login error using LDAP after applying new changes
Authentication error occurred on LDAP authentication: if the problem is not related to wrong typing credential issue or LDAP setting, you may be facing a problem of identity. Follow the steps:
* Get the users list
```
oc get users

NAME        UID                                    FULL NAME   IDENTITIES
John_37    ff3b3e73-eeb2-11e9-8940-005056937315   John_37     LDAP:cn=John_37,ou=Empresas,ou=Parceiros,ou=Usuarios,o=ENTERPRISE
MTH01      d6cb2447-e534-11e9-877b-0050569374d9   MTH01       LDAP-CORP:cn=MTH01,ou=Empresas,ou=Parceiros,ou=Usuarios,o=ENTERPRISE
Jessy      b7e80b61-f4d6-11e9-af43-005056937315   Jessy       LDAP:cn=Jessy,ou=Empresas,ou=Parceiros,ou=Usuarios,o=ENTERPRISE
```
* Delete the identity - get the string from the last column (IDENTITIES)
```
oc delete identity LDAP:cn=John_37,ou=Empresas,ou=Parceiros,ou=Usuarios,o=ENTERPRISE
```
* Delete the user
```
oc delete user John_37
```

<a name="ocp_insuficient_memory" />

### Pods on Pending Status - 0/X nodes are available: X Insufficient memory, X node(s) had taints that the pod didn't tolerate
When facing this problem, two main issues may be affecting the pod to start up:
* Resource limits: the min cpu of container was defined to high value, p.e. 650MiB. Since the container needed less resource it couldn't start because of this restriction. On container resource limit, it's not recommended to define a minimum set to memory ou cpu. Only if you suposed to and know what are doing.
* Low quota resource to openshift project: if your resource max memory, max cpu or max pods is on the edge, consider raising it. There was a curious situation that it was remaing 1 cpu unit free. My pod was suposed to consume 1 unit, but still the status was locked on pending. When I raised the cpu limit, the status changed to running.

<a name="frontend_ocp_empty_container" />

# Frontend - new image is not being pushed and the page is showing a default ngnix Openshift content
After running the pipeline, the image wasn't being updated on Openshift registry. Checking the deploying log on Jenkins, the sha (container hash code) generated to container is the same of the preexisting image on registry. On that case, the image is not replaced. Since it is not changed, the Openshift doesn't update the pod image.
To force the Openshift to update the image, the current image can be erased using the Openshift Console:
* Delete all resources related to the pod:
```
#  list the services of a project
oc get services -n <project>

# select the service by name and delete it all resources related to it:
oc delete all -l app=<name of selected service>
```
* Delete the image: Build -> Images -> Select the image to delete.
<br/> Run the deploy process again and it will work.

<a name="docker" />

# Docker

Show docker commands and config details.
* systemctl daemon-reload - daemon aplica as alterações realizados no /etc/sysconfig/docker
* systemctl restart docker - restart docker reinializa o serviço
* docker restart $(docker ps -q) - como o restart do systemctl altera o pid do processo pai bem como os descritores de rede, necessita do restart

<a name="troubleshooting_docker" />

## Troubleshooting

<a name="certificate_error" />

### X509: certificate signed by unknown authority
There some steps that should be acomplished to solve the problem of certificate:
* Create the registry directory with domain name as presented in the following:
```
mkdir /etc/docker/certs.d/domain.name:port
```
<br/>example:
```
mkdir /etc/docker/certs.d/registry-docker.apps:443
```
* Create the files with the following structure:
```
/etc/docker/certs.d/registry-docker.apps:443
|- registry-docker.apps.cer
|- ca.cer
|- registry-docker.apps.key
```
* Restart the docker service
```
service docker restart
docker restart $(docker ps -q) 
```

<a name="dial_tcp" />

### dial tcp: lookup <docker_registry_url> on XXX.XXX.XXX.XXX:XX: no such host
The key file configuration to solve this problem is /etc/sysconfig/docker. The file should look like this:
```shell
# /etc/sysconfig/docker

# Modify these options if you want to change the way the docker daemon runs
OPTIONS='--selinux-enabled  --log-driver=journald -g /app/docker --signature-verification=false --log-level=info --disable-legacy-registry --userland-proxy=false'

if [ -z "${DOCKER_CERT_PATH}" ]; then
    DOCKER_CERT_PATH=/etc/docker
fi

# Do not add registries in this file anymore. Use /etc/containers/registries.conf
# instead. For more information reference the registries.conf(5) man page.

# Location used for temporary files, such as those created by
# docker load and build operations. Default is /var/lib/docker/tmp
# Can be overriden by setting the following environment variable.
# DOCKER_TMPDIR=/var/tmp

# Controls the /etc/cron.daily/docker-logrotate cron job status.
# To disable, uncomment the line below.
# LOGROTATE=false
ADD_REGISTRY='--add-registry docker.io'
ADD_REGISTRY='--add-registry docker-registry.apps'

INSECURE_REGISTRY='--insecure-registry docker.io --insecure-registry docker-registry.dev.apps'

HTTP_PROXY="http://user:password@proxycielo.visanet.corp:8080"
HTTPS_PROXY="http://user:password@proxycielo.visanet.corp:8080"
NO_PROXY="docker.io,.dev.apps"
```
Set the following parameters:
* ADD_REGISTRY - add a line for each registry domain;
* If the domain has a insecure URL, then add the domain to INSECURE_REGISTRY like other itens;
* If your organization has proxy, add the complete domain or partial as the example. Wildcard is not needed.
* Restart the docker service
```
systemctl daemon-reload
service docker restart
docker restart $(docker ps -q) 
```
1. [Ansible](#)

<a name="ansible" />

# Ansible
It always about to do some tasks on a targe server. Let's start on this jorney doing the following steps:
* The first thing to do is guarantee ssh services on target server
```
# install a hand package manager
apt-get install aptitude

# get the name on the catalog of what is needed. p.e. openssl server
aptitude search openssh

# openssl-server can be located on the list. Use it as parameter
aptitude install openssh-server

# start the ssh service
service ssh start
```

* Generate the pair key:
```
# generate a key using default values. Hit <Enter> three times
ssh-keygen -t rsa
```

* Copy ~/.ssh/id_rsa.pub to a shared volume or to an appropriate director.
```
cp ~/.ssh/id_rsa.pub <dest>
```

<a name="git" />

# Git

Show most used commands on Git.

- Problema: fatal: unable to access 'https://corp-git.ccorp.local/gcsc/pipeline/': SSL certificate problem: self signed certificate 
<br/>Workaround: git config --global http.sslVerify false 

 
- Editar configurações globais: 
<br/> git config --global -e 

- Armazenar as credentials globalmente: 
<br/> git config --global credential.helper store 

- Processo de commit: 
<br/> git add . 
<br/> git commit -m "Commit inicial com aplicação com hello world" 
<br/> git push -u origin develop 
<br/> git remote –v – equivalente ao svn info (determina a url de origem do repositorio) 

- rename de pastas / arquivos envolve add e rm 
<br/> git add . - inclui o diretório renomeado 
<br/> git rm . - remove os arquivos que não existem mais no diretorio do projeto 

- comando completo para commit 
<br/> git fetch; git pull; git add .; git commit -m 'Updated Pipeline'; git push -u origin featureopenshift; git fetch; git pull;java -jar jenkins-cli.jar -s http://10.82.100.13:8080/ -auth @../jenkins_secret.pass build CAN_can-orchestratorV2/develop/ -s –v 
<br/> Nota: para usar o jenkins-cli, baixar [aqui](https://jenkins.io/doc/book/managing/cli/). O Arquivo jenkins_secret.pass pode ter qualquer nome, bastando que seja criado e o path passado em @. O conteudo do arquivo deverá ser: username:password

- comando para executar bash dentro do docker 
docker exec -it 326e bash (não precisa digitar o hash inteiro, apenas os 4 primeiros digitos) 

<a name="aws" />

# AWS

<a name="s3" />

## Bucket (S3)

Simple Storage Service used to storage any data with several advantages like resilience, disaster proctection, on demand service and others.
* Copy data into S3:
```
# local to remote
aws s3 cp data.txt s3://amazing-bucket-dho/data.txt
# remote to local
aws s3 cp s3://amazing-bucket-dho/data.txt data.txt 
```
   * data.txt - filename
   * amazing-bucket-dho - the bucket name
<br/>AWS CLI copy command is bi-directional. A file can be copied from local to remote or remote to local
* Sync local folder into remote bucket folder:
```
aws s3 sync folder-dho s3://amazing-bucket-dho/files
```
   * folder-dho - local foleder
   * s3://amazing-bucket-dho/files - complete remote bucket folder path
   
<a name="unix" />

# Unix Commands

<a name="unix-common-cmd" />

## Most common Unix Commands

Show main used commands on unix and shell.
- net user /domain <user_login>
- Test network conectivity:
<br/> telnet \<ip\> \<port\>
<br/> nmap \<ip\> -p \<port\>
<br/> see also: netcat
- create fs (file system): sudo mke2fs /dev/sdf
- to check the device to format the fs or mount: ls /dev -ltr
- mount and map the fs on unix: sudo mount /dev/sdf /mnt
- unmap the fs: sudo umount /mnt

<a name="apt-get-mac" />

## apt-get to MacOS - Why use HomeBrew Instead
Unfortunately this no apt-get package manager as used in Linux. Instead, HomeBrew can be used.
```
# Install Homebrew using Perl
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
<a name="utilities" />

# Utilities

<a name="certificates" />

## Exporting CA Certificates

Some platforms needs certification authority to connect to other services. In my case, to connect Openshift CLI plugin from Jenkins to Openshift, it was necessary to export certificates from the site. It was a little trick, since it is necessary to be clear which format of certificate is necessary to be exported.

<a name="curl_telnet" />

## Test connection telnet protocol using CURL

Test connection using curl using telnet protocol:
```
curl -v telnet://portal.com:9000
```
