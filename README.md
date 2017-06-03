Adicionar o repositório ao gitlab
====

```
SLIDES 5 MIN
DEV PIPELINE 25 - 30
PROD PIPELINE 15
```

1. Criar projeto todo-app no gitlab

git remote add origin http:root@gitlab/root/todo-app.git

1. Alterar branch default para develop
1. Publicar codigo git para a branch develop


Criar Job no Jenkins Branch Develop
====

=====http:root@gitlab/root/todo-app.git
Configurar Job 
Configurar credenciais


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
node {
    stage('Checkout'){
          checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      app = docker.build("bernardovale/todo-app:${gitCommit}", "-f ci/staging/Dockerfile ${WORKSPACE}")
    }
}
```

Executando Testes Unitarios
=====
```groovy
#!groovy
node {
    stage('Checkout'){
          checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      app = docker.build("bernardovale/todo-app:${gitCommit}", "-f ci/staging/Dockerfile ${WORKSPACE}")
    }
    stage('Unit Tests'){
      app.inside{
        sh 'pytest tests/unit --junit-xml results/unit.xml'
        junit '**/results/*.xml'
      }
    }
}
```

Deployando Staging
=====

```groovy
#!groovy
node {
    stage('Checkout'){
          checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      app = docker.build("bernardovale/todo-app:${gitCommit}", "-f ci/staging/Dockerfile ${WORKSPACE}")
    }
    stage('Unit Tests'){
      app.inside{
        sh 'pytest tests/unit --junit-xml results/unit.xml'
        junit '**/results/*.xml'
      }
    }
    stage('Deploy Staging'){
      sh "GIT_COMMIT=${gitCommit} docker-compose -p todo -f staging.yml up -d"
    }
}
```

Testes Funcionais
=====
```groovy
#!groovy
node {
    stage('Checkout'){
          checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      app = docker.build("bernardovale/todo-app:${gitCommit}", "-f ci/staging/Dockerfile ${WORKSPACE}")
    }
    stage('Unit Tests'){
      app.inside{
        sh 'pytest tests/unit --junit-xml results/unit.xml'
        junit '**/results/*.xml'
      }
    }
    stage('Deploy Staging'){
      sh "GIT_COMMIT=${gitCommit} docker-compose -p todo -f staging.yml up -d"
    }
    stage('Functional Tests'){
        docker.image('mongo').withRun(){ m ->
          app.inside("-e MONGODB_HOST=mongo -e APP_ENV=staging --link ${m.id}:mongo"){
            sh 'pytest tests/functional --junit-xml results/functional.xml'
            junit '**/results/*.xml'
          }
        }
    }
}
```

Publish Image
====

```groovy
#!groovy
node(){
    stage('Checkout'){
          checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      app = docker.build("bernardovale/todo-app:${gitCommit}", "-f ci/staging/Dockerfile ${WORKSPACE}")
    }
    stage('Unit Tests'){
      app.inside{
        sh 'pytest tests/unit --junit-xml results/unit.xml'
        junit '**/results/*.xml'
      }
    }
    stage('Deploy Staging'){
      sh "GIT_COMMIT=${gitCommit} docker-compose -p todo -f staging.yml up -d"
    }
    stage('Functional Tests'){
        docker.image('mongo').withRun(){ m ->
          app.inside("-e MONGODB_HOST=mongo -e APP_ENV=staging --link ${m.id}:mongo"){
            sh 'pytest tests/functional --junit-xml results/functional.xml'
            junit '**/results/*.xml'
          }
        }
    }
    // Define Docker hub credentials jenkins/configure
    stage('Publish'){
      docker.withRegistry("https://index.docker.io/v1/", 'registry') {
        app.push()
      }
    }
}
```

Pipeline Production
======
Criar novo Job para production build from master


Checkout & Build
=====
```groovy
#!groovy
node('master'){
    stage('Checkout'){
          checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      app = docker.build("bernardovale/todo-app:${gitCommit}", "-f ci/production/Dockerfile ${WORKSPACE}")
    }
}
```

Acceptance Tests
======
```groovy
#!groovy
node('master'){
    stage('Checkout'){
          checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      app = docker.build("bernardovale/todo-app:${gitCommit}", "-f ci/production/Dockerfile ${WORKSPACE}")
    }
    stage('Acceptance Tests'){
      dir('tests/acceptance'){
        docker.image('mongo').withRun('--network agile --name mongo-accept'){ m ->
          app.withRun("--network agile --name todo-accept -e MONGODB_HOST=mongo-accept -e APP_ENV=production"){
            try{
              sh 'docker-compose build'  
              sh 'URL=http://todo-accept:8001 docker-compose run --rm cuchromer features'
            }finally{
              sh 'docker-compose down'
            }
          }
        }
      }
    }
}
```

Publish & Deploy
========
```groovy
#!groovy
node('master'){
    stage('Checkout'){
          checkout scm
    }
    stage('Build'){
      gitCommit = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
      app = docker.build("bernardovale/todo-app:${gitCommit}", "-f ci/production/Dockerfile ${WORKSPACE}")
    }
    stage('Acceptance Tests'){
      dir('tests/acceptance'){
        docker.image('mongo').withRun('--network agile --name mongo-accept'){ m ->
          app.withRun("--network agile --name todo-accept -e MONGODB_HOST=mongo-accept -e APP_ENV=production"){
            try{
              sh 'docker-compose build'  
              sh 'URL=http://todo-accept:8001 docker-compose run --rm cuchromer features'
            }finally{
              sh 'docker-compose down'
            }
          }
        }
      }
    }
    stage('Publish'){
      appVersion = input(
        id: 'userInput', message: 'Publish new version', parameters: [
        [$class: 'TextParameterDefinition', description: 'App Version', name: 'version']
      ])
      docker.withRegistry("https://index.docker.io/v1/", 'registry') {
        app.push(appVersion)
      }
    }
}
```


Plugins
====
- Gitlab Hook Plugin
- BlueOcean