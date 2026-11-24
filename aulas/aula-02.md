# ☸️ Aula 02 — Pods

> 🌷 Conhecendo os Pods, sua relação com Containers e Nodes, seu ciclo de vida e os principais comandos utilizados para criar, consultar e gerenciar esses recursos no Kubernetes.

---

## 🩷 1. O que é um Pod?

Um **Pod** é a menor unidade implantável e gerenciável pelo Kubernetes.

Ele representa um ambiente no qual um ou mais containers são executados. Cada Pod possui suas próprias configurações e recursos compartilhados entre os containers que o compõem.

Na maioria dos casos, um Pod contém apenas um container principal. Entretanto, também é possível agrupar múltiplos containers que precisam trabalhar juntos.

## 🐳 2. Pods e Containers

Os containers são responsáveis por executar as aplicações, enquanto os Pods fornecem o ambiente utilizado pelo Kubernetes para gerenciar esses containers.

Um Pod pode conter:

- Um único container;
- Dois ou mais containers que trabalham em conjunto.

Quando existem múltiplos containers no mesmo Pod, eles compartilham alguns recursos, como:

- O mesmo endereço IP;
- O mesmo espaço de rede e de portas;
- Volumes compartilhados, quando configurados.

### 🌸 Exemplo de organização

```text
Pod
├── Container 1
└── Container 2
```

Os containers dentro de um mesmo Pod podem se comunicar utilizando a rede compartilhada.

## 🖥️ 3. Pods e Nodes

Um **Node** é uma máquina física ou virtual que pertence ao cluster Kubernetes e fornece recursos para executar aplicações.

Os Pods são executados nos Nodes, conforme as decisões de agendamento realizadas pelo Kubernetes.

A relação entre esses conceitos pode ser representada assim:

```text
CLUSTER KUBERNETES
│
├── Node 1
│   ├── Pod A
│   │   └── Container
│   │
│   └── Pod B
│       └── Container
│
└── Node 2
    └── Pod C
        └── Container
```

### 🔍 Como acontece a execução?

1. O usuário solicita a criação de um Pod.
2. O `kube-scheduler` escolhe um Node adequado para executá-lo.
3. O `kubelet` acompanha a especificação do Pod no Node selecionado.
4. O Container Runtime executa os containers.

Dessa forma:

- **Cluster:** conjunto de Nodes que trabalham em conjunto.
- **Node:** máquina que fornece recursos para executar Pods.
- **Pod:** unidade utilizada pelo Kubernetes para executar containers.
- **Container:** ambiente responsável por executar a aplicação.

## 🔄 4. Ciclo de vida de um Pod

Durante sua existência, um Pod pode passar por diferentes fases, que representam seu estado geral dentro do Kubernetes.

| Fase | Significado |
| :--- | :--- |
| `Pending` | O Pod foi aceito pelo cluster, mas ainda aguarda etapas necessárias para iniciar sua execução. |
| `Running` | O Pod foi associado a um Node e seus containers estão sendo criados ou executados. |
| `Succeeded` | Todos os containers terminaram sua execução com sucesso. |
| `Failed` | Todos os containers terminaram e pelo menos um deles apresentou falha. |
| `Unknown` | O estado do Pod não pode ser determinado pelo Kubernetes. |

**Observação:** a fase `Running` não garante, por si só, que a aplicação esteja saudável. Para verificar a prontidão e a saúde de aplicações, existem mecanismos específicos, como as probes.

## ⚙️ 5. Conhecendo o kubectl

O **kubectl** é a ferramenta de linha de comando utilizada para se comunicar com o Kubernetes e administrar os recursos do cluster.

Por meio dele, podemos criar, consultar, atualizar e remover recursos, além de investigar o funcionamento das aplicações.

Alguns comandos importantes:

| Comando | Função |
| :--- | :--- |
| `kubectl get nodes` | Lista os Nodes do cluster. |
| `kubectl get pods` | Lista os Pods do namespace atual. |
| `kubectl describe pod` | Apresenta informações detalhadas de um Pod. |
| `kubectl logs` | Exibe os logs de um container. |
| `kubectl exec` | Executa comandos dentro de um container. |
| `kubectl apply -f` | Aplica a configuração declarada em um arquivo. |
| `kubectl delete pod` | Solicita a exclusão de um Pod. |

## 📝 6. Estrutura YAML no Kubernetes

O **YAML** é um formato utilizado para organizar informações de configuração.

No Kubernetes, ele é frequentemente utilizado para declarar os recursos que desejamos criar e suas respectivas configurações.

Um manifesto YAML possui campos importantes:

| Campo | Função |
| :--- | :--- |
| `apiVersion` | Define a versão da API utilizada. |
| `kind` | Identifica o tipo de recurso. |
| `metadata` | Armazena informações de identificação do recurso. |
| `spec` | Descreve as características e configurações desejadas. |

### 🌸 Exemplo de estrutura

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: meu-primeiro-pod

spec:
  containers:
    - name: nginx
      image: nginx:latest
```

Uma informação importante: no YAML, o símbolo `#` indica um comentário. O conteúdo comentado não é interpretado como configuração.

## 🐳 7. Criando nosso primeiro Pod

Para colocar os conceitos em prática, criamos nosso primeiro Pod utilizando a imagem do Nginx.

### 🖥️ 7.1. Preparando o ambiente

Antes de começar, habilitamos o Kubernetes no Docker Desktop e verificamos o funcionamento do cluster.

Utilizamos:

```powershell
kubectl get nodes
```

Resultado:

```text
NAME                    STATUS   ROLES           AGE     VERSION
desktop-control-plane   Ready    control-plane   2m10s   v1.36.1
```

O status `Ready` confirmou que o Node estava pronto para executar Pods.

### 📂 7.2. Organizando o projeto

Criamos uma pasta chamada `aula-02` dentro do repositório de estudos de Kubernetes.

Dentro dela, criamos o arquivo:

```text
aula-02/
└── pod.yaml
```

### 📄 7.3. Criando o manifesto pod.yaml

O arquivo foi preenchido com a seguinte configuração:

```yaml
# Versão da API do Kubernetes utilizada para criar o recurso.
apiVersion: v1

# Tipo de recurso que queremos criar.
kind: Pod

# Metadados: informações utilizadas para identificar o recurso.
metadata:
  # Nome que será atribuído ao nosso Pod.
  name: meu-primeiro-pod

# Spec: descreve o estado desejado do recurso.
spec:

  # Lista os containers que serão executados dentro do Pod.
  containers:

    # Nome que identifica o container dentro do Pod.
    - name: nginx

      # Imagem utilizada para criar o container.
      image: nginx:latest
```

### ⚙️ 7.4. Criando o Pod

Com o manifesto preparado, executamos:

```powershell
kubectl apply -f pod.yaml
```

Entendendo o comando:

| Parte | Significado |
| :--- | :--- |
| `kubectl` | Ferramenta de comunicação com o Kubernetes. |
| `apply` | Aplica a configuração declarada. |
| `-f` | Indica que será utilizado um arquivo. |
| `pod.yaml` | Nome do manifesto YAML. |

Resultado:

```text
pod/meu-primeiro-pod created
```

Isso confirmou que o Kubernetes recebeu o manifesto e criou o recurso Pod.

### 🔎 7.5. Consultando o Pod

Para verificar se o Pod estava em execução, utilizamos:

```powershell
kubectl get pods
```

Resultado:

```text
NAME               READY   STATUS    RESTARTS   AGE
meu-primeiro-pod   1/1     Running   0          46s
```

Interpretando as colunas:

| Campo | Significado |
| :--- | :--- |
| `NAME` | Nome do Pod. |
| `READY` | Quantidade de containers prontos em relação ao total. |
| `STATUS` | Estado atual do Pod. |
| `RESTARTS` | Quantidade de reinicializações. |
| `AGE` | Tempo desde a criação. |

O resultado `1/1 Running` confirmou que o container estava pronto e em execução.

## 🔍 8. Inspecionando Pods

Além de listar os Pods, podemos consultar informações detalhadas sobre eles.

### 🩷 8.1. Utilizando kubectl describe

Executamos:

```powershell
kubectl describe pod meu-primeiro-pod
```

Esse comando apresenta informações como:

- Nome e namespace do Pod;
- Node onde foi executado;
- Endereço IP;
- Imagem utilizada;
- Estado do container;
- Condições do Pod;
- Eventos relacionados à execução.

### 🌸 8.2. Interpretando os Events

Durante nossa prática, encontramos os seguintes eventos:

```text
Scheduled
Pulling
Pulled
Created
Started
```

Eles representam a sequência de acontecimentos durante a inicialização do Pod:

1. `Scheduled`: o kube-scheduler escolheu o Node onde o Pod seria executado.
2. `Pulling`: o kubelet iniciou o download da imagem.
3. `Pulled`: a imagem foi baixada com sucesso.
4. `Created`: o container foi criado.
5. `Started`: o container foi iniciado.

Essa sequência permitiu acompanhar o processo de criação do nosso container.

## 📋 9. Consultando logs e executando comandos

Também aprendemos a investigar o que acontece dentro dos containers.

### 🔎 9.1. Visualizando os logs

Utilizamos:

```powershell
kubectl logs meu-primeiro-pod
```

Esse comando exibe a saída de logs do container.

Como estávamos utilizando o Nginx, encontramos mensagens relacionadas à inicialização do servidor.

Entre elas:

```text
Configuration complete; ready for start up
```

E também:

```text
nginx/1.31.8
```

Essas mensagens confirmaram que o servidor Nginx havia iniciado corretamente.

### 🖥️ 9.2. Entrando no container com kubectl exec

Para executar comandos dentro do container, utilizamos:

```powershell
kubectl exec -it meu-primeiro-pod -- /bin/bash
```

Entendendo o comando:

| Parte | Função |
| :--- | :--- |
| `exec` | Executa um comando dentro do container. |
| `-i` | Mantém a entrada interativa aberta. |
| `-t` | Cria um terminal. |
| `meu-primeiro-pod` | Identifica o Pod. |
| `--` | Separa as opções do kubectl do comando executado no container. |
| `/bin/bash` | Abre o terminal Bash. |

Após executar o comando, o terminal apresentou:

```text
root@meu-primeiro-pod:/#
```

Isso indicou que estávamos dentro do container, utilizando o usuário `root`.

### 📂 9.3. Explorando os arquivos do Nginx

Para listar os arquivos do diretório padrão de páginas do Nginx, executamos:

```bash
ls -la /usr/share/nginx/html
```

Encontramos:

```text
50x.html
index.html
```

- `index.html`: página HTML padrão exibida pelo Nginx.
- `50x.html`: página utilizada para apresentar erros de servidor da família HTTP 50x.

Depois, visualizamos o conteúdo do arquivo:

```bash
cat /usr/share/nginx/html/index.html
```

Encontramos o HTML responsável por apresentar a mensagem:

```html
<h1>Welcome to nginx!</h1>
```

Isso confirmou que conseguimos acessar e explorar os arquivos armazenados dentro do container.

Para sair do ambiente, utilizamos:

```bash
exit
```

O comando encerrou a sessão interativa e nos levou de volta ao PowerShell.

## 🗑️ 10. Removendo Pods

Depois de testar nosso Pod, aprendemos a solicitar sua exclusão.

### 🌸 10.1. Excluindo o Pod

Executamos:

```powershell
kubectl delete pod meu-primeiro-pod
```

Resultado:

```text
pod "meu-primeiro-pod" deleted from default namespace
```

O Kubernetes confirmou que o Pod foi excluído do namespace `default`.

### 🔎 10.2. Conferindo a remoção

Para verificar se o recurso havia sido removido, utilizamos:

```powershell
kubectl get pods
```

Resultado:

```text
No resources found in default namespace.
```

Isso confirmou que não havia mais Pods no namespace atual.

**Observação:** excluir um Pod criado diretamente não significa excluir a imagem utilizada pelo container. Além disso, como esse Pod não estava sendo gerenciado por um controlador, como um Deployment, ele não foi recriado automaticamente.

---

## 📝 11. Resumo dos principais conceitos

| Conceito ou comando | Significado |
| :--- | :--- |
| Pod | Menor unidade implantável e gerenciável do Kubernetes. |
| Container | Ambiente responsável por executar a aplicação. |
| Node | Máquina física ou virtual que executa Pods. |
| `Pending` | Pod aguardando etapas necessárias para iniciar. |
| `Running` | Pod associado a um Node e com containers em criação ou execução. |
| `Succeeded` | Todos os containers terminaram com sucesso. |
| `Failed` | Todos os containers terminaram e pelo menos um falhou. |
| `Unknown` | Estado do Pod não pôde ser determinado. |
| `kubectl` | Ferramenta de linha de comando do Kubernetes. |
| `apiVersion` | Define a versão da API. |
| `kind` | Identifica o tipo de recurso. |
| `metadata` | Contém informações de identificação. |
| `spec` | Descreve o estado desejado do recurso. |

### Principais comandos utilizados

```powershell
# Listar os Nodes do cluster
kubectl get nodes

# Criar ou aplicar um recurso a partir de YAML
kubectl apply -f pod.yaml

# Listar os Pods
kubectl get pods

# Consultar informações detalhadas de um Pod
kubectl describe pod meu-primeiro-pod

# Visualizar os logs
kubectl logs meu-primeiro-pod

# Entrar no container
kubectl exec -it meu-primeiro-pod -- /bin/bash

# Excluir o Pod
kubectl delete pod meu-primeiro-pod
```

## 🔗 12. Referências

- [Documentação oficial do Kubernetes — Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- [Documentação oficial do Kubernetes — kubectl](https://kubernetes.io/docs/reference/kubectl/)
- [Documentação oficial do Kubernetes — Objetos e manifestos](https://kubernetes.io/docs/concepts/overview/working-with-objects/)

---
