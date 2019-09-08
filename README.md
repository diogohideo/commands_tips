# commands_tips
Tip & sintax command used in Unix and Windows

# Table of Contents
1. [Openshift](#openshift)
1. [Git](#git)

<a name="openshift" />

## Openshift

* oc login: Log in to your OpenShift cluster and save the session token for subsequent use. You will be prompted for the user name and password or directed to a web console page where you can obtain a token you must then use to use to login from the command line. The web page will require you to first login in to the web console if you are not already logged in.<br>
* oc login \<server>: Log in to a specific OpenShift cluster. You will need to specify the name of the server as argument the first time you are using it, or if switching back to it after having used a different cluster.<br>
* oc login --username \<user>: Log in to your OpenShift cluster as a specific user.<br>
* oc login --username \<username> --password \<password>: Log in to your OpenShift cluster as a specific user and supply the password on the command line. Note that this is not recommended for real systems as your password may be logged or retained in command history.<br>
* oc login --token \<token>: Log in to your server using a token for an existing session.<br>
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
* oc adm policy add-role-to-user \<admin|basic-user|view|edit|system:deployer|system:image-builder|system:image-puller|system:image-pusher> \<user> -n \<project-name>

<href name="git" />

## Git

Problema: fatal: unable to access 'https://corp-git.ccorp.local/gcsc/pipeline/': SSL certificate problem: self signed certificate 
Workaround: git config --global http.sslVerify false 

 
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
