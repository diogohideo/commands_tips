# commands_tips
Tip & sintax command used in Unix and Windows to several platforms.

# Table of Contents
1. [Jenkins](#jenkins)
1. [Openshift](#openshift)
   1. [Deploy a existing image](#deploy-existing-image)
   1. [Setup Roles](#roles)
   1. [Metrics](#metrics)
   1. [S2I - Source to Image](#s2i)
   1. [Odo](#odo)
1. [Docker](#docker)
1. [Git](#git)
1. [Unix Commands](#unix)

<a name="jenkins" />

## Jenkins

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

## Openshift

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
* to a deployed app guestbook, how to attach the config map "default" using comandline:
<br/>oc patch dc/guestbook -p '{"spec":{"template":{"spec":{"volumes":[{"name":"config","configMap":{"name":"default"}}]}}}}'

<a name="deploy-existing-image" />

### Deploy a existing image
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

### Setup Roles
Commands to setup roles.
* list all roles:
<br/>oc get groups
* list all commands:
<br/>oc policy -h
* assign edit / view to a role
<br/>oc policy add-role-to-group edit <role> -n <project/namespace>

<a name="metrics" />

### Metrics
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

### S2I - Source to Image
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

### Odo
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

<a name="docker" />

## Docker

Show docker commands and config details.
* systemctl daemon-reload - daemon aplica as alterações realizados no /etc/sysconfig/docker
* systemctl restart docker - restart docker reinializa o serviço
* docker restart $(docker ps -q)- como o restart do systemctl altera o pid do processo pai bem como os descritores de rede, necessita do restart


<a name="git" />

## Git

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

### Unix Commands

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
