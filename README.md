
1. Iniciar o ambiente

Adicionar o repositório ao gitlab
====

1. create a project on gitlab

git remote add origin http:root@gitlab/root/todo-app.git

1. Alterar branch default para develop




Criar Job no Jenkins
=====http:gitlab/root/todo-app.git=
Configurar Job 
adicinar credenciais


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
      sh "docker run --rm -i todo-staging:${gitCommit} pytest tests/unit"
    }
}
```

Deployando Staging
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
      sh "docker run --rm -i todo-staging:${gitCommit} pytest tests/unit"
    }
    stage('Deploy Staging'){
      sh "GIT_COMMIT=${gitCommit} docker-compose -f staging.yml up -d"
    }
}
```

Testes Funcionais
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
      sh "docker run --rm -i todo-staging:${gitCommit} pytest tests/unit"
    }
    stage('Deploy Staging'){
      sh "GIT_COMMIT=${gitCommit} docker-compose -f staging.yml up -d"
    }
    stage('Funcional Tests'){
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

Pipeline Production
======
Criar credenciais para download do Oracle JDK
Criar um no na AWS
http://jenkins:8080/computer/createItem



Introduzindo um Bug
======
Intruduzir um bug, e fazer rollback da aplicação


Plugins
====
- Gitlab Hook Plugin
- PlueOcean

Unico Branch
====

- Pipeline de push
- Pipeline de tag

Push
===
build staging

Tag
====
- Build
- Publish
- Deploy aws
- smoke tests
- rollback if need

Develop & Master
===

Push develop
Push master

