class: center, bottom


# From Zero to Hero: Criando uma pipeline de testes com Docker, Jenkins e Gitlab

.right[![Jenkins-Docker](http://localhost:8000/assets/images/jenkins.png)]

---
# Objetivo

Criar uma pipeline completa utilizando _Pipeline as Code_ com Jenkins e Docker como
plataforma do projeto.

--

Demonstrar como Docker pode facilitar sua vida.

--

???
Conceitos e tecnologias serão introduzidos enquanto escrevemos a pipeline.

Algumas etapas da configuração já foram feitas por não encaixarem no tempo da talk.

---


class: left, middle
background-position: bottom;

# Como faremos

Criaremos duas pipelines.

--

1. Acionada após `push` na branch `develop`

???
Para dar tempo, vamos ignorar algumas boas práticas do git-flow

Pipeline develop:

unit, functional, deploy staging, publish artifacts
  
--
1. Acionada após `push` na branch `master`

???

Pipeline master:

Acceptance tests, publish, deploy prod


---
class: top, left, faustao
layout: false

# WARNING

Essa demo pode pegar fogo.

.right[![Faustao](http://localhost:8000/assets/images/pegandofogo.jpg)]


???
Explicar que as coisas podem quebrar, vamos concertar.

Podemos recorrer a replays

---

# Quiz Time!

???
1. Quantos conhecem docker
1. Usam docker para testes
1. Usam docker em produção
1. Jenkins?
1. Pipeline as code

---

# Docker

Docker é uma plataforma que permite isolar aplicações em **containers**.

Isso significa que: 

Todos os componentes necessários para sua aplicação funcionar estarão dentro do **container**.

--

Desenvolvedores utiliza Docker para eliminar o famoso "Funciona na minha máquina"!

--

DevOps/Sysadmins utilizam Docker para executar e gerenciar aplicações