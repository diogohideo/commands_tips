# commands_tips
Tip & sintax command used in Unix and Windows to several platforms.

# Table of Contents
1. [Jenkins](#jenkins)
1. [Openshift](#openshift)
   1. [Deploy a existing image](#deploy-existing-image)
   1. [Setup Roles](#roles)
   1. [Kibana - Troubleshooting](#kibana)
   1. [Metrics](#metrics)
   1. [S2I - Source to Image](#s2i)
   1. [Odo](#odo)
   1. [Troubleshooting](#troubleshooting)
1. [Docker](#docker)
1. [Git](#git)
1. [Unix Commands](#unix)
1. [Utilities](#utilities)
   1. [Exporting CA Certificates](#certificates)

<a name="jenkins" />

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
* oc adm policy add-role-to-user edit \<username> -n \<project>: Add another user to a project so that they can work within the project, including creating new deployments or deleting applications.<br>
* oc adm policy add-role-to-user view \<username> -n \<project>: Add another user to a project but such that they can only view what is in the project.<br>
* oc adm policy add-role-to-user admin \<username> -n \<project>: Add another user to a project such that they are effectively a joint owner of the project and have administration rights over it, including the ability to delete the project.
* grant access project to a user
<br/>oc adm policy add-role-to-user \<admin|basic-user|view|edit|system:deployer|system:image-builder|system:image-puller|system:image-pusher> \<user> -n \<project-name>
* grant access project to a group (example):
<br/>oc policy add-role-to-group edit Openshift_HML_CAN_Edit -n can-hml
* to a deployed app guestbook, how to attach the config map "default" using comandline:
<br/>oc patch dc/guestbook -p '{"spec":{"template":{"spec":{"volumes":[{"name":"config","configMap":{"name":"default"}}]}}}}'
* oc get nodes - check node availability

<a name="deploy-existing-image" />

## Deploy a existing image
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

<a name="roles" />

## Setup Roles
Commands to setup roles.
* list all roles:
<br/>oc get groups
* list all commands:
<br/>oc policy -h
* assign edit / view to a role
<br/>oc policy add-role-to-group edit <role> -n <project/namespace>
   
<a name="kibana" />

## Kibana - Troubleshooting
### Safe guard is red status - Gateway timeout on Kibana
First of all, access on console online the project openshift-logging on Openshift. On the overview, access the resource logging-kibana and access the external route to check if the service is available.
If the message "Gateway timeout is prompted out", follow the steps:
* Get the pod name of logging data master with hash on the left

* Check index status:
<br/>oc exec \<logging-es-data-master-hash-of-pod> -- curl -s --key /etc/elasticsearch/secret/admin-key --cert /etc/elasticsearch/secret/admin-cert --cacert /etc/elasticsearch/secret/admin-ca -HContent-Type:application/json  https://localhost:9200/_cat/indices?v 
* Check if the searchguard is RED as the example:
<br/>health status index
<br/>green  open   .kibana.bb615795cff20a34fd133d6a13e4a9c5a9ce5e57                    NjVWuumISCO_a8sXprkPaA   1   0          4            0     46.9kb         46.9kb
<br/>green  open   project.ngc-dev.10e561e1-d354-11e9-a92b-0050569374d9.2019.10.29     vZ6AvXWWTTC83ZMXSCDicQ   1   0        102            0    153.7kb        153.7kb
<br/>red  open   .searchguard                                                        Y1piRFOMR3iMyl1p1LF-_Q   1   0          5            0     33.1kb         33.1kb
<br/>green  open   project.fac-dev.c0615391-d353-11e9-a92b-0050569374d9.2019.10.30     0ZiMxg1lT7GDXv2d-PVXgQ   1   0       1805            0      2.1mb          2.1mb
<br/>green  open   .kibana.2df074c6d9620ca5a1e2d11ffa09997a6c78d4c9                    LDl-Bsx4SBeYID895tjPhA   1   0          1            1     26.4kb         26.4kb
<br/>green  open   .operations.2019.10.26                                              0uGsaJrlTLmWDhkj_fZF1Q   1   0   24592923            0     15.7gb         15.7gb
* If searchguard is red, re-deploy the logging data master. Try to access kibana. If the problem persists, re-deploy loggin-kibana too. This should solve the problem.
* Otherwise, delete the index with red state:
<br/>health status index
<br/>green  open   .kibana.bb615795cff20a34fd133d6a13e4a9c5a9ce5e57                    NjVWuumISCO_a8sXprkPaA   1   0          4            0     46.9kb         46.9kb

<br/>oc exec \<logging-es-data-master-hash-of-pod> --  curl -s --key /etc/elasticsearch/secret/admin-key --cert /etc/elasticsearch/secret/admin-cert --cacert /etc/elasticsearch/secret/admin-ca -XDELETE 'https://localhost:9200/.kibana.bb615795cff20a34fd133d6a13e4a9c5a9ce5e57'
<br/>

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
Solutions to problems faced on Openshift:
* authentication error occurred on LDAP authentication - if the problem is not related to wrong typing credential issue, you may be facing a problem of identity.
<br/>
oc delete identity LDAP:cn=xxxxxx,ou=yyyyyyyy,ou=wwwwwwww,o=ORGANIZATIONZ
<br/>
<br/>Get the identity parameter using oc get identity. Locate te user line info and get the data from the first column and paste on the command above.

<a name="docker" />

# Docker

Show docker commands and config details.
* systemctl daemon-reload - daemon aplica as alterações realizados no /etc/sysconfig/docker
* systemctl restart docker - restart docker reinializa o serviço
* docker restart $(docker ps -q) - como o restart do systemctl altera o pid do processo pai bem como os descritores de rede, necessita do restart


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

<a name="unix" />

# Unix Commands

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

<a name="utilities" />

# Utilities

<a name="certificates" />

## Exporting CA Certificates

Some platforms needs certification authority to connect to other services. In my case, to connect Openshift CLI plugin from Jenkins to Openshift, it was necessary to export certificates from the site. It was a little trick, since it is necessary to be clear which format of certificate is necessary to be exported.
