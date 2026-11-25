# ☸️ Aula 04 — Deployments

> 🌷 Aprendendo a criar, gerenciar, escalar e atualizar aplicações no Kubernetes utilizando Deployments.

---

## 🩷 1. O que é um Deployment?

O **Deployment** é um recurso do Kubernetes responsável por declarar e gerenciar o estado desejado de uma aplicação.

Por meio dele, podemos:

- Definir a quantidade desejada de Pods;
- Criar e gerenciar ReplicaSets;
- Atualizar a imagem dos containers;
- Acompanhar o progresso das atualizações;
- Realizar rollback para versões anteriores.

O Deployment facilita o gerenciamento das aplicações, pois permite controlar seus Pods sem precisar criá-los e administrá-los individualmente.

## 🧩 2. Relação entre Deployment, ReplicaSet e Pods

O Deployment trabalha em conjunto com outros recursos do Kubernetes.

A estrutura funciona da seguinte maneira:

**Deployment → ReplicaSet → Pods**

Cada recurso possui uma responsabilidade:

| Recurso | Responsabilidade |
| :--- | :--- |
| Deployment | Gerencia o estado desejado da aplicação e suas atualizações. |
| ReplicaSet | Garante que a quantidade desejada de Pods esteja em execução. |
| Pod | É a menor unidade implantável do Kubernetes, responsável por executar os containers. |

Quando criamos um Deployment, o Kubernetes cria um ReplicaSet, que por sua vez cria e gerencia os Pods.

Assim, não precisamos criar um ReplicaSet manualmente, como fizemos na aula anterior.

## 🖥️ 3. Criando nosso primeiro Deployment

Para colocar os conceitos em prática, criamos uma pasta chamada `aula-04`.

Dentro dela, criamos o arquivo `deployment.yaml`.

A estrutura ficou assim:

```text
estudos-kubernetes/
└── aula-04/
    └── deployment.yaml
```

### 📄 Arquivo deployment.yaml

Utilizamos o seguinte manifesto:

```yaml
# Define a versão da API do Kubernetes utilizada
apiVersion: apps/v1

# Informa que o recurso criado será um Deployment
kind: Deployment

# Metadados do Deployment
metadata:
  # Nome utilizado para identificar o Deployment
  name: nginx-deployment

# Especificações de funcionamento do Deployment
spec:

  # Quantidade desejada de Pods
  replicas: 3

  # Define como o Deployment encontrará os Pods que deverá gerenciar
  selector:
    matchLabels:
      app: nginx

  # Modelo utilizado para criar os Pods
  template:

    # Metadados dos Pods que serão criados
    metadata:
      labels:
        app: nginx

    # Especificações dos containers dos Pods
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

### 🔍 Entendendo as principais propriedades

| Propriedade | Função |
| :--- | :--- |
| `apiVersion` | Define a versão da API utilizada pelo recurso. |
| `kind` | Identifica o tipo de recurso criado. |
| `metadata` | Armazena informações de identificação, como o nome. |
| `spec` | Define as configurações desejadas do recurso. |
| `replicas` | Determina a quantidade desejada de Pods. |
| `selector` | Define quais Pods serão selecionados pelo Deployment. |
| `template` | Contém o modelo utilizado para criar os Pods. |
| `containers` | Define os containers que serão executados nos Pods. |
| `image` | Informa a imagem utilizada pelo container. |

## ⚙️ 4. Aplicando o Deployment

Depois de criar o arquivo YAML, executamos:

```powershell
kubectl apply -f .\aula-04\deployment.yaml
```

O comando `kubectl apply` aplica a configuração declarada no arquivo ao cluster Kubernetes.

**Resultado:**

```text
deployment.apps/nginx-deployment created
```

Isso confirmou que o Deployment foi criado com sucesso.

### 🔎 Verificando os Deployments

Para listar os Deployments existentes no namespace atual, utilizamos:

```powershell
kubectl get deployments
```

Resultado observado:

```text
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           18s
```

Entendendo as colunas:

| Coluna | Significado |
| :--- | :--- |
| `NAME` | Nome do Deployment. |
| `READY` | Quantidade de Pods prontos em relação à quantidade desejada. |
| `UP-TO-DATE` | Quantidade de réplicas atualizadas conforme a configuração mais recente. |
| `AVAILABLE` | Quantidade de réplicas disponíveis. |
| `AGE` | Tempo desde a criação do recurso. |

O resultado `3/3` confirmou que os três Pods desejados estavam prontos.

### Conferindo os ReplicaSets

Executamos:

```powershell
kubectl get replicasets
```

Resultado:

```text
NAME                         DESIRED   CURRENT   READY   AGE
nginx-deployment-6797d5487   3         3         3       104s
```

O Kubernetes criou automaticamente um ReplicaSet para gerenciar os três Pods.

### Conferindo os Pods

Para listar os Pods criados, utilizamos:

```powershell
kubectl get pods
```

Resultado:

```text
NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-6797d5487-5q2zp   1/1     Running   0          2m25s
nginx-deployment-6797d5487-b4pw8   1/1     Running   0          2m25s
nginx-deployment-6797d5487-vnkcw   1/1     Running   0          2m25s
```

Os três Pods estavam com status `Running`, indicando que estavam em execução.

## 🔍 5. Inspecionando o Deployment

Para obter informações mais detalhadas sobre o recurso, utilizamos:

```powershell
kubectl describe deployment nginx-deployment
```

Esse comando apresenta informações como:

- Nome e namespace;
- Quantidade de réplicas desejadas e disponíveis;
- Estratégia de atualização;
- Imagem utilizada;
- Labels e Selectors;
- Condições do Deployment;
- ReplicaSets associados;
- Eventos relacionados ao recurso.

Entre as informações observadas, encontramos:

```text
Replicas: 3 desired | 3 updated | 3 total | 3 available | 0 unavailable

StrategyType: RollingUpdate

RollingUpdateStrategy: 25% max unavailable, 25% max surge
```

Isso confirmou que o Deployment possuía três réplicas disponíveis e utilizava a estratégia de atualização `RollingUpdate`.

### 🌸 Acompanhando o progresso

Também utilizamos:

```powershell
kubectl rollout status deployment/nginx-deployment
```

Resultado:

```text
deployment "nginx-deployment" successfully rolled out
```

Esse comando acompanha o progresso de uma atualização ou implantação até que ela seja concluída ou apresente algum problema.

## 📈 6. Scaling — Alterando a quantidade de réplicas

Uma das vantagens dos Deployments é permitir alterar a quantidade de Pods de maneira simples.

Esse processo é chamado de **Scaling**.

### Aumentando a quantidade de Pods

Para aumentar de três para cinco réplicas, executamos:

```powershell
kubectl scale deployment nginx-deployment --replicas=5
```

Depois, verificamos:

```powershell
kubectl get deployments
```

Resultado observado:

```text
nginx-deployment   5/5   5   5
```

Ao consultar os Pods com `kubectl get pods`, encontramos cinco instâncias em execução.

O Kubernetes criou automaticamente dois novos Pods para atender à quantidade desejada.

### Diminuindo a quantidade de Pods

Em seguida, voltamos para três réplicas:

```powershell
kubectl scale deployment nginx-deployment --replicas=3
```

Após verificar novamente:

```powershell
kubectl get deployments
kubectl get pods
```

Observamos que o Deployment voltou a apresentar três réplicas disponíveis.

Os dois Pods adicionais foram removidos.

### ⚠️ Atenção ao arquivo YAML

O comando `kubectl scale` altera a quantidade de réplicas diretamente no cluster, mas não modifica automaticamente o arquivo YAML salvo no computador.

Por isso, é importante manter o manifesto local alinhado com a configuração desejada.

Caso o arquivo YAML ainda declare três réplicas e tenha sido alterado apenas o estado do cluster para cinco, uma nova aplicação do manifesto poderá retornar a quantidade para três.

## 🔄 7. Rolling Update — Atualizando a aplicação

O Kubernetes permite atualizar aplicações sem precisar remover todos os Pods antigos de uma só vez.

Esse processo é chamado de **Rolling Update**.

A estratégia padrão de atualização dos Deployments é `RollingUpdate`.

Ela substitui os Pods gradualmente, criando novas instâncias enquanto reduz as antigas.

### 🌷 Entendendo a estratégia

No nosso Deployment, que possuía três réplicas, o comando `describe` apresentou:

```text
RollingUpdateStrategy: 25% max unavailable, 25% max surge
```

Essas configurações controlam o comportamento da atualização:

- `maxUnavailable`: quantidade máxima de Pods que podem ficar indisponíveis durante a atualização.
- `maxSurge`: quantidade máxima de Pods adicionais que podem ser criados acima do número desejado durante a atualização.

O Kubernetes utiliza esses limites para controlar a substituição dos Pods.

### Atualizando a imagem do container

Inicialmente, nosso Deployment utilizava:

```yaml
image: nginx:latest
```

Para atualizar a imagem para uma versão específica, executamos:

```powershell
kubectl set image deployment/nginx-deployment nginx=nginx:1.28
```

Entendendo o comando:

| Parte | Função |
| :--- | :--- |
| `kubectl set image` | Altera a imagem de um container. |
| `deployment/nginx-deployment` | Identifica o Deployment. |
| `nginx` | Nome do container. |
| `nginx:1.28` | Nova imagem que será utilizada. |

Depois, acompanhamos a atualização:

```powershell
kubectl rollout status deployment/nginx-deployment
```

Resultado:

```text
deployment "nginx-deployment" successfully rolled out
```

### Conferindo os ReplicaSets após a atualização

Executamos:

```powershell
kubectl get replicasets
```

Resultado observado:

```text
NAME                         DESIRED   CURRENT   READY   AGE
nginx-deployment-6797d5487   0         0         0       57m
nginx-deployment-bdd5f448f   3         3         3       49s
```

O Kubernetes criou um novo ReplicaSet para os Pods que utilizavam a imagem `nginx:1.28`.

O ReplicaSet anterior permaneceu registrado, mas com zero réplicas.

Isso é importante porque o Kubernetes mantém o histórico das revisões para permitir o rollback.

## 🕰️ 8. Rollback — Voltando para uma versão anterior

O **Rollback** permite retornar a uma configuração anterior do Deployment.

Durante nossa prática, primeiro consultamos o histórico:

```powershell
kubectl rollout history deployment/nginx-deployment
```

Resultado:

```text
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

As revisões representavam:

- Revisão 1: configuração inicial com `nginx:latest`.
- Revisão 2: atualização para `nginx:1.28`.

A coluna `CHANGE-CAUSE` apresenta a descrição da alteração, quando informada. O valor `<none>` indica que não havia uma descrição registrada.

### Executando o rollback

Para retornar à configuração anterior, utilizamos:

```powershell
kubectl rollout undo deployment/nginx-deployment
```

O Kubernetes exibiu um aviso relacionado à anotação `last-applied-configuration`, mas também confirmou:

```text
deployment.apps/nginx-deployment rolled back
```

Isso indicou que o rollback foi realizado com sucesso.

### 🔍 Entendendo o aviso

O aviso explicou que o Deployment havia sido gerenciado anteriormente com `kubectl apply` e que o rollback não atualizaria automaticamente a anotação `kubectl.kubernetes.io/last-applied-configuration`.

Essa anotação armazena informações relacionadas à última configuração aplicada pelo `kubectl apply`.

Como o rollback não atualiza essa anotação, ela pode ficar diferente da configuração atual do recurso e causar comportamentos inesperados em futuras aplicações de manifestos.

Portanto, o aviso não significava que o rollback havia falhado. Era apenas um alerta para termos cuidado com futuras operações de `kubectl apply`.

### Conferindo o rollback

Primeiro, verificamos novamente o progresso:

```powershell
kubectl rollout status deployment/nginx-deployment
```

Resultado:

```text
deployment "nginx-deployment" successfully rolled out
```

Depois, consultamos:

```powershell
kubectl describe deployment nginx-deployment
```

Entre as informações apresentadas, encontramos:

```text
Annotations:
  deployment.kubernetes.io/revision: 3

Replicas: 3 desired | 3 updated | 3 total | 3 available | 0 unavailable

NewReplicaSet: nginx-deployment-6797d5487 (3/3 replicas created)
```

Também verificamos os ReplicaSets:

```powershell
kubectl get replicasets
```

Resultado:

```text
NAME                         DESIRED   CURRENT   READY
nginx-deployment-6797d5487   3         3         3
nginx-deployment-bdd5f448f   0         0         0
```

O Deployment voltou a utilizar a imagem `nginx:latest`.

### 💡 Por que a revisão passou para 3?

O rollback não faz o número da revisão voltar para o número anterior.

Ele registra uma nova revisão, pois representa uma nova alteração no histórico do Deployment.

Na nossa prática:

| Revisão | Configuração |
| :--- | :--- |
| 1 | `nginx:latest` — configuração inicial |
| 2 | `nginx:1.28` — atualização |
| 3 | `nginx:latest` — retorno após rollback |

Assim, mesmo voltando para uma configuração anterior, o histórico continua avançando.

## 🧹 9. Removendo o Deployment

Depois de finalizar os testes, removemos o Deployment utilizando:

```powershell
kubectl delete -f .\aula-04\deployment.yaml
```

Resultado:

```text
deployment.apps "nginx-deployment" deleted from default namespace
```

O Kubernetes removeu o Deployment e também os ReplicaSets e Pods gerenciados por ele.

Para confirmar a limpeza, executamos:

```powershell
kubectl get deployments
kubectl get replicasets
kubectl get pods
```

Os três comandos apresentaram:

```text
No resources found in default namespace.
```

Isso confirmou que todos os recursos criados durante a prática haviam sido removidos do namespace `default`.

O arquivo `deployment.yaml`, entretanto, continuou salvo no nosso computador e no repositório.

## 📝 10. Resumo da aula

- ☸️ Deployment é um recurso que gerencia o estado desejado de uma aplicação.
- 🧩 A hierarquia funciona como Deployment → ReplicaSet → Pods.
- 📄 Arquivos YAML permitem declarar as configurações desejadas.
- 🔎 `kubectl get deployments` lista os Deployments.
- 🔍 `kubectl describe deployment` apresenta informações detalhadas.
- 🚀 `kubectl rollout status` acompanha o progresso de uma implantação ou atualização.
- 📈 `kubectl scale` altera a quantidade de réplicas.
- 🔄 Rolling Update substitui os Pods gradualmente.
- 🖼️ `kubectl set image` altera a imagem utilizada pelo container.
- 🕰️ `kubectl rollout history` consulta o histórico de revisões.
- ↩️ `kubectl rollout undo` permite retornar a uma configuração anterior.
- 🧹 `kubectl delete` remove os recursos do cluster, sem apagar o manifesto local.

---
