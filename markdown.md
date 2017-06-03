class: center, bottom


# From Zero to Hero: Criando uma pipeline de testes com Docker, Jenkins e Gitlab

.right[![Jenkins-Docker](http://localhost:8000/assets/images/jenkins.png)]

???

DevOps Engineer @ Avenue Code

Trabalho com as tecnologias há alguns anos

1. Criar uma pipeline completa utilizando Jenkins, Gitlab, Docker como
plataforma do projeto.

1. Demonstrar como Docker pode facilitar sua vida.

1. Conceitos e tecnologias serão introduzidos enquanto escrevemos a pipeline.

1. Algumas etapas da configuração já foram feitas por não encaixarem no tempo da talk.


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

???
Container = pedaço isolado do kernel com mem/cpu controlados e com seu proprio filesystem executando
alguma coisa (app). CONTAINERS TEM SOMENTE UMA RESPONSABILIDADE

--

#### Build

Constroi sua aplicação utilizando uma série de passos descritos em um arquivo chamado `Dockerfile`

???
Receita de bolo para instalar sua aplicação

--

#### Push/Ship

Envia a sua aplicação para um local centralizado onde todos poderam baixa-la.

???
Colaboração

--

#### Run

Execute a sua aplicação da mesma forma em qualquer lugar


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
class: middle, bottom

# Thank You!