# ☸️ Aula 01 — Introdução ao Kubernetes

> 🌷 Conhecendo os fundamentos do Kubernetes, a orquestração de containers e a arquitetura de um cluster.

---

## 🩷 1. O que é Kubernetes?

O **Kubernetes** é uma plataforma de código aberto utilizada para automatizar a implantação, o gerenciamento e o escalonamento de aplicações executadas em containers.

Ele permite administrar aplicações compostas por vários containers, facilitando tarefas como:

- Aumentar ou diminuir a quantidade de réplicas de uma aplicação;
- Recuperar aplicações quando algum container apresenta falhas;
- Distribuir aplicações entre diferentes máquinas;
- Gerenciar a comunicação entre os componentes;
- Manter o estado desejado da aplicação.

O Kubernetes também é conhecido como **K8s**. Esse nome é uma abreviação de *Kubernetes*, substituindo as oito letras entre o K e o s pelo número 8.

## 🐳 2. O que é orquestração de containers?

A **orquestração de containers** consiste em automatizar o gerenciamento de containers e das aplicações que são executadas neles.

Quando uma aplicação possui vários containers, administrar cada um manualmente pode se tornar complexo.

O Kubernetes facilita esse gerenciamento, permitindo:

| Recurso | Função |
| :--- | :--- |
| Escalonamento | Aumenta ou diminui a quantidade de réplicas da aplicação. |
| Recuperação automática | Detecta falhas e trabalha para restabelecer o estado desejado. |
| Balanceamento de carga | Distribui o tráfego entre as aplicações disponíveis. |
| Atualizações | Gerencia atualizações das aplicações. |
| Gerenciamento de recursos | Considera os recursos disponíveis nos Nodes do cluster. |

Dessa forma, o Kubernetes automatiza tarefas que seriam trabalhosas de executar manualmente.

## 💻 3. Docker x Kubernetes

Embora trabalhem juntos, Docker e Kubernetes possuem finalidades diferentes.

| Docker 🐳 | Kubernetes ☸️ |
| :--- | :--- |
| Trabalha com imagens e containers. | Orquestra aplicações executadas em containers. |
| Permite construir imagens. | Gerencia aplicações em um cluster. |
| Executa containers. | Gerencia réplicas e recuperação de aplicações. |
| Pode ser utilizado em ambientes locais. | É utilizado para administrar aplicações em clusters. |

### 🌸 Como eles trabalham juntos?

Podemos utilizar o Docker para construir uma imagem de uma aplicação e, posteriormente, utilizar o Kubernetes para gerenciar sua execução em um cluster.

Por exemplo, em uma aplicação que utiliza Python e MySQL:

- O Docker pode ser utilizado para construir as imagens dos serviços.
- O Kubernetes pode administrar a execução desses componentes, controlar suas réplicas e ajudar a manter as aplicações disponíveis.

**Importante:** Kubernetes não substitui completamente o Docker. São ferramentas complementares.

Além disso, o Kubernetes não exige que o Docker Engine seja utilizado para executar containers. Ele pode trabalhar com runtimes compatíveis, como `containerd` e `CRI-O`.

## 🧩 4. O que é um Cluster Kubernetes?

Um **Cluster Kubernetes** é um conjunto de máquinas que trabalham em conjunto para executar e gerenciar aplicações em containers.

Essas máquinas são chamadas de **Nodes**.

Um cluster é composto principalmente por duas partes:

- **Control Plane:** responsável por controlar e gerenciar o cluster.
- **Worker Nodes:** responsáveis por executar as aplicações.

A organização pode ser representada da seguinte maneira:

```text
              CLUSTER KUBERNETES
                      |
          +-----------+-----------+
          |                       |
     CONTROL PLANE           WORKER NODES
          |                       |
    Gerencia o cluster       Executam aplicações
                                  |
                                 Pods
```

Um cluster de estudos pode possuir apenas um Node, enquanto ambientes de produção podem utilizar vários Nodes para distribuir as aplicações e os recursos.

## 🖥️ 5. O que são Nodes?

Um **Node** é uma máquina física ou virtual que faz parte de um cluster Kubernetes.

Os Nodes fornecem os recursos necessários para executar as aplicações, como processamento, memória e armazenamento.

Existem dois papéis principais:

### 🧠 5.1. Control Plane Node

É o Node que hospeda os componentes responsáveis pelo gerenciamento do cluster.

Ele controla o funcionamento do Kubernetes e acompanha o estado das aplicações.

### ⚙️ 5.2. Worker Node

É o Node responsável por executar as aplicações.

Nele ficam os Pods, que são as unidades utilizadas pelo Kubernetes para executar containers.

## 🎀 6. O que é o Control Plane?

O **Control Plane** é o conjunto de componentes responsáveis por controlar e gerenciar o cluster Kubernetes.

Ele funciona como o centro de administração do cluster, recebendo solicitações, armazenando informações e coordenando as ações necessárias para manter o estado desejado.

Seus principais componentes são:

| Componente | Responsabilidade |
| :--- | :--- |
| `kube-apiserver` | Recebe e processa as solicitações feitas ao Kubernetes por meio da API. |
| `etcd` | Armazena o estado e as informações de configuração do cluster. |
| `kube-scheduler` | Escolhe em qual Node um novo Pod deverá ser executado. |
| `kube-controller-manager` | Executa os controllers que acompanham e trabalham para manter o estado desejado do cluster. |
| `cloud-controller-manager` | Integra o Kubernetes aos recursos de provedores de nuvem, quando utilizado. |

### 🔍 6.1. kube-apiserver

É o componente responsável por disponibilizar a API do Kubernetes.

As solicitações feitas ao cluster passam pelo API Server, que recebe e processa essas requisições.

Ele funciona como a principal porta de comunicação com o Control Plane.

### 💾 6.2. etcd

É um banco de dados distribuído do tipo chave-valor (*key-value store*).

Ele armazena informações importantes sobre o cluster, como seu estado e suas configurações.

É por meio dessas informações que o Kubernetes consegue acompanhar o que está configurado e o que precisa ser mantido.

### 📅 6.3. kube-scheduler

É responsável por decidir em qual Node um novo Pod será executado.

Para tomar essa decisão, considera fatores como:

- Recursos disponíveis nos Nodes;
- Restrições de execução;
- Requisitos definidos para os Pods.

O scheduler não executa diretamente os containers. Sua função é selecionar o Node adequado para a execução do Pod.

### 🔄 6.4. kube-controller-manager

Executa os controllers do Kubernetes.

Esses controllers acompanham continuamente o estado do cluster e trabalham para aproximar o estado atual do estado desejado.

Por exemplo, se uma aplicação deve possuir três réplicas, os controllers acompanham essa configuração e trabalham para manter a quantidade solicitada.

### ☁️ 6.5. cloud-controller-manager

É um componente utilizado quando o Kubernetes precisa se integrar a recursos de um provedor de nuvem.

Pode ajudar no gerenciamento de recursos externos, como máquinas virtuais, balanceadores de carga e outras funcionalidades oferecidas pela infraestrutura em nuvem.

Esse componente é opcional, dependendo do ambiente utilizado.

## ⚙️ 7. Componentes dos Worker Nodes

Os **Worker Nodes** são responsáveis por executar as aplicações do cluster.

Eles possuem componentes que permitem receber instruções do Control Plane, executar containers e gerenciar aspectos da comunicação.

| Componente | Responsabilidade |
| :--- | :--- |
| `kubelet` | Agente que acompanha os Pods e garante que os containers descritos estejam em execução. |
| `kube-proxy` | Mantém regras de rede relacionadas à comunicação dos Services. |
| Container Runtime | Responsável por executar os containers. |

### 🧩 7.1. kubelet

É um agente executado em cada Node.

Sua função é receber as especificações dos Pods e garantir que os containers descritos estejam em execução naquele Node.

Ele acompanha o estado dos Pods e comunica informações ao Control Plane.

### 🌐 7.2. kube-proxy

É um componente relacionado à comunicação de rede dos Services.

Ele mantém regras de rede que permitem direcionar o tráfego para os Pods correspondentes.

**Observação:** o `kube-proxy` pode ser substituído por outras soluções de rede que ofereçam funcionalidades equivalentes.

### 📦 7.3. Container Runtime

É o componente responsável por executar os containers dentro dos Worker Nodes.

Alguns exemplos são:

- `containerd`;
- `CRI-O`.

O Kubernetes utiliza runtimes compatíveis com a Container Runtime Interface (CRI) para realizar essa comunicação.

## 🧠 8. Entendendo a arquitetura na prática

Agora que conhecemos os componentes, podemos imaginar uma situação em que solicitamos ao Kubernetes que execute três réplicas de uma aplicação.

De maneira simplificada, acontece o seguinte:

1. O `kube-apiserver` recebe a solicitação feita ao Kubernetes.
2. As informações sobre o estado desejado são armazenadas no `etcd`.
3. O `kube-scheduler` escolhe os Nodes nos quais os novos Pods serão executados.
4. O `kubelet` dos Nodes selecionados acompanha as especificações dos Pods e garante que os containers estejam em execução.
5. Os controllers acompanham o estado do cluster e trabalham para manter a quantidade de réplicas solicitada.

Assim, o Kubernetes consegue coordenar diferentes componentes para manter as aplicações funcionando conforme a configuração desejada.

## 📚 9. Resumo dos principais conceitos

| Conceito | Significado |
| :--- | :--- |
| Kubernetes | Plataforma de orquestração de containers. |
| Cluster | Conjunto de máquinas que executam aplicações. |
| Node | Máquina física ou virtual que pertence ao cluster. |
| Control Plane | Parte responsável por controlar e gerenciar o cluster. |
| Worker Node | Node responsável por executar as aplicações. |
| Pod | Unidade utilizada pelo Kubernetes para executar containers. |
| Container Runtime | Componente responsável por executar containers. |

### Componentes do Control Plane

- `kube-apiserver`: recebe e processa solicitações pela API.
- `etcd`: armazena o estado e as configurações do cluster.
- `kube-scheduler`: escolhe o Node onde um Pod será executado.
- `kube-controller-manager`: executa controllers que acompanham o estado do cluster.
- `cloud-controller-manager`: integra o cluster a recursos de provedores de nuvem, quando necessário.

### Componentes dos Worker Nodes

- `kubelet`: acompanha os Pods e garante a execução dos containers.
- `kube-proxy`: mantém regras de rede relacionadas aos Services, quando utilizado.
- Container Runtime: executa os containers.

## 🔗 10. Referência

- [Documentação oficial do Kubernetes — Arquitetura do Kubernetes](https://kubernetes.io/pt-br/docs/concepts/architecture/)

---
