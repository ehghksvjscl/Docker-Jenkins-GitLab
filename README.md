
1. Iniciar o ambiente

Adicionar o repositório ao gitlab
====

1. create a project on gitlab

git remote add origin http://root@code.agiletesters.com/root/todo-app.git


Criar Job no Jenkins
=====http://gitlab/root/todo-app.git=
Configurar Job 
adicinar credenciais
Testar o scan

Iniciar Jenkinsfile
=====
Primeiro estagio

```groovy
#!groovy
node(){
    stage('Checkout'){
        checkout scm
    }
}
```

Integração Gitlab
=====
Criar integração com o Gitlab



Adicionar Build Section
======
```groovy
#!groovy
node(){
    stage('Checkout'){
        checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      sh "docker build -f ci/staging/Dockerfile -t todo-staging:${gitCommit} ."
    }
}
```

Executando Testes Unitarios
=====
```groovy
#!groovy
node(){
    stage('Checkout'){
        checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      sh "docker build -f ci/staging/Dockerfile -t todo-staging:${gitCommit} ."
    }
    stage('Unit Tests'){
      mongo = sh(returnStdout: true, script: 'docker run -d mongo').trim()
      sh "docker run --rm -e MONGODB_HOST=mongo --link ${mongo}:mongo -i todo-staging:${gitCommit} pytest"
      sh "docker rm -f ${mongo}"
    }
}
```

Publicando O Resultado
=====
```groovy
#!groovy
node(){
    stage('Checkout'){
        checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      sh "docker build -f ci/staging/Dockerfile -t todo-staging:${gitCommit} ."
    }
    stage('Unit Tests'){
      mongo = sh(returnStdout: true, script: 'docker run -d mongo').trim()
      def workspace = pwd()
      sh 'mkdir -p results'
      // Volumes mounted from Jenkins container
      sh "docker run --rm -e MONGODB_HOST=mongo --link ${mongo}:mongo --volumes-from jenkins -i todo-staging:${gitCommit} pytest --junit-xml /results/result.xml"
      // Copying results
      sh 'cp /results/result.xml results/'
      sh "docker rm -f ${mongo}"
      // Archiving Junit result
      junit '**/results/*.xml'
    }
}
```


Plugins
====
- Gitlab Hook Plugin

