# commands_tips
Tip & sintax command used in Unix and Windows

# Table of Contents
1. [Openshift](#openshift)
1. [Git](#git)

<a name="openshift" />

## Openshift

* login
  * oc login
  * oc login --username developer --password developer
  * switch back to signed in user - oc login --username developer
  * oc login --token=xxxxxxxxxxxxxxxxxxxxxx --server=https://xxxx.openshift.com
  * API - curl -H "Authorization: Bearer xxxxxxxxxxxxxxxxxxxxxx" "https://xxxx.openshift.com/oapi/v1/users/~"
  
* get oc user id
  * oc whoami

* get oc server url
  * oc whoami --show-server
  
* show project that can be accessed
  * oc get projects

* create a new project
  * oc new-project \<name>
 
* grant access project to a user
  * oc adm policy add-role-to-user \<admin|basic-user|view|edit|system:deployer|system:image-builder|system:image-puller|system:image-pusher> \<user> -n \<project-name>

<href name="git" />

## Git

Problema: fatal: unable to access 'https://corp-git.ccorp.local/gcsc/pipeline/': SSL certificate problem: self signed certificate 
Workaround: git config --global http.sslVerify false 

 
Editar configurações globais: 
git config --global -e 

Armazenar as credentials globalmente: 
git config --global credential.helper store 

Processo de commit: 
git add . 
git commit -m "Commit inicial com aplicação com hello world" 
git push -u origin develop 
git remote –v – equivalente ao svn info (determina a url de origem do repositorio) 

- rename de pastas / arquivos envolve add e rm 
git add . - inclui o diretório renomeado 
git rm . - remove os arquivos que não existem mais no diretorio do projeto 

- comando para commit 
git fetch; git pull; git add .; git commit -m 'Updated Pipeline'; git push -u origin featureopenshift; git fetch; git pull;java -jar jenkins-cli.jar -s http://10.82.100.13:8080/ -auth @../jenkins_secret.pass build CAN_can-orchestratorV2/develop/ -s –v 

- comando para executar bash dentro do docker 
docker exec -it 326e bash (não precisa digitar o hash inteiro, apenas os 4 primeiros digitos) 
