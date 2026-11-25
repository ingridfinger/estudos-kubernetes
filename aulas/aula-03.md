# ☸️ Aula 03 — ReplicaSets, Labels e Selectors

> 🌷 Aprendendo a manter a quantidade desejada de Pods em execução, identificar recursos por meio de Labels e selecioná-los utilizando Selectors.

---

## 🩷 1. O que é um ReplicaSet?

O **ReplicaSet** é um recurso do Kubernetes responsável por garantir que uma quantidade específica de Pods esteja em execução.

Ele monitora continuamente os Pods e compara a quantidade existente com a quantidade desejada.

Por exemplo, se configurarmos um ReplicaSet para manter 3 réplicas, ele trabalhará para garantir que existam 3 Pods correspondentes.

Caso algum Pod seja excluído ou apresente uma falha, o ReplicaSet identifica a diferença e cria outro Pod para recuperar a quantidade desejada.

### 🔍 Como funciona?

Imagine que configuramos um ReplicaSet com 3 réplicas:

| Situação | Quantidade de Pods |
| :--- | :---: |
| Quantidade desejada | 3 |
| Quantidade atual | 3 |
| Um Pod é excluído | 2 |
| ReplicaSet cria outro Pod | 3 |

O ReplicaSet não recupera exatamente o Pod excluído. Ele cria um novo Pod para substituir aquele que deixou de existir.

## 🧩 2. O que são réplicas?

As **réplicas** representam instâncias independentes de Pods que executam uma aplicação.

Por exemplo, quando configuramos:

```yaml
replicas: 3
```

Estamos solicitando que o ReplicaSet mantenha três Pods.

Cada Pod possui seu próprio ciclo de vida, mas todos podem utilizar a mesma imagem e configuração.

É importante lembrar que três réplicas significam três Pods diferentes, e não três containers dentro de um único Pod.

## 🏷️ 3. O que são Labels?

As **Labels** são identificações atribuídas aos recursos do Kubernetes, como Pods.

Elas permitem organizar, classificar e identificar recursos por meio de pares de chave e valor (*key-value*).

Exemplo:

```yaml
labels:
  app: nginx
  ambiente: desenvolvimento
```

Nesse exemplo, temos duas Labels:

| Chave | Valor |
| :--- | :--- |
| `app` | `nginx` |
| `ambiente` | `desenvolvimento` |

As Labels funcionam como etiquetas que ajudam a identificar os recursos dentro do cluster.

## 🔎 4. O que são Selectors?

Os **Selectors** são mecanismos utilizados para localizar recursos que possuem determinadas Labels.

Eles funcionam como filtros: procuram os recursos que correspondem às identificações especificadas.

Por exemplo, imagine que temos três Pods:

| Pod | Label |
| :--- | :--- |
| `nginx-1` | `app: nginx` |
| `nginx-2` | `app: nginx` |
| `mysql-1` | `app: mysql` |

Se utilizarmos o Selector `app: nginx`, serão selecionados os Pods `nginx-1` e `nginx-2`, pois ambos possuem a Label correspondente.

### 🌸 Diferença entre Labels e Selectors

- **Labels:** são as etiquetas atribuídas aos recursos.
- **Selectors:** são os filtros utilizados para encontrar recursos com determinadas etiquetas.

## 💗 5. Relação entre ReplicaSets, Labels e Selectors

O ReplicaSet utiliza Selectors para identificar quais Pods deve gerenciar.

Para isso, as Labels definidas no template dos Pods precisam corresponder ao Selector configurado no ReplicaSet.

Exemplo:

**Selector do ReplicaSet:**

```yaml
selector:
  matchLabels:
    app: nginx
```

**Labels do template dos Pods:**

```yaml
template:
  metadata:
    labels:
      app: nginx
```

Observe que `app: nginx` aparece nos dois lugares.

Essa correspondência permite que o ReplicaSet identifique os Pods que deverá gerenciar.

## 🖥️ 6. Criando nosso primeiro ReplicaSet

Para colocar os conceitos em prática, criamos uma pasta chamada `aula-03` dentro do repositório `estudos-kubernetes`.

Dentro dela, criamos o arquivo `replicaset.yaml`.

A estrutura ficou assim:

```text
aula-03/
└── replicaset.yaml
```

### 🐳 Conteúdo do arquivo replicaset.yaml

```yaml
# Define a versão da API do Kubernetes utilizada
apiVersion: apps/v1

# Informa que o recurso criado será um ReplicaSet
kind: ReplicaSet

# Metadados do ReplicaSet
metadata:
  # Nome utilizado para identificar o ReplicaSet
  name: nginx-replicaset

# Especificações de funcionamento do ReplicaSet
spec:

  # Quantidade desejada de Pods que devem permanecer em execução
  replicas: 3

  # Define como o ReplicaSet encontrará os Pods que deverá gerenciar
  selector:
    matchLabels:
      # Seleciona Pods que possuam a Label app com o valor nginx
      app: nginx

  # Modelo utilizado para criar os Pods
  template:

    # Metadados dos Pods que serão criados
    metadata:
      labels:
        # Adiciona a Label que corresponde ao Selector acima
        app: nginx

    # Especificações dos containers que estarão dentro dos Pods
    spec:
      containers:

        # Configuração do container
        - name: nginx

          # Imagem utilizada para criar o container
          image: nginx:latest
```

### 🔍 Entendendo a configuração

| Campo | Função |
| :--- | :--- |
| `apiVersion` | Define a versão da API utilizada. |
| `kind` | Identifica o tipo de recurso criado. |
| `metadata.name` | Define o nome do ReplicaSet. |
| `spec.replicas` | Determina a quantidade desejada de Pods. |
| `selector.matchLabels` | Define as Labels utilizadas para selecionar os Pods. |
| `template` | Define o modelo dos Pods que serão criados. |
| `template.metadata.labels` | Define as Labels atribuídas aos Pods. |
| `containers` | Define os containers presentes nos Pods. |
| `image` | Especifica a imagem utilizada pelo container. |

**Observação:** existem dois campos chamados `spec` no arquivo. O primeiro pertence ao ReplicaSet e o segundo está dentro de `template`, definindo as especificações dos Pods.

## ⚙️ 7. Criando o ReplicaSet no Kubernetes

Com o arquivo preparado, executamos:

```powershell
kubectl apply -f replicaset.yaml
```

Esse comando envia o manifesto ao Kubernetes para criar o ReplicaSet.

Em seguida, verificamos sua criação:

```powershell
kubectl get replicasets
```

Resultado:

```text
NAME               DESIRED   CURRENT   READY   AGE
nginx-replicaset   3         3         3       11s
```

### 🔍 Entendendo a saída

| Coluna | Significado |
| :--- | :--- |
| `NAME` | Nome do ReplicaSet. |
| `DESIRED` | Quantidade desejada de Pods. |
| `CURRENT` | Quantidade atual de Pods controlados. |
| `READY` | Quantidade de Pods prontos. |
| `AGE` | Tempo desde a criação do ReplicaSet. |

O resultado mostrou que o ReplicaSet desejava 3 Pods, possuía 3 Pods e todos estavam prontos.

### Conferindo os Pods

Também executamos:

```powershell
kubectl get pods
```

Resultado:

```text
NAME                     READY   STATUS    RESTARTS   AGE
nginx-replicaset-5xphc   1/1     Running   0          60s
nginx-replicaset-wmstx   1/1     Running   0          60s
nginx-replicaset-xggvv   1/1     Running   0          60s
```

O comando apresentou os três Pods criados pelo ReplicaSet.

### Entendendo a saída

| Coluna | Significado |
| :--- | :--- |
| `NAME` | Nome do Pod. |
| `READY` | Quantidade de containers prontos em relação ao total. |
| `STATUS` | Estado atual do Pod. |
| `RESTARTS` | Quantidade de reinicializações dos containers. |
| `AGE` | Tempo desde a criação do Pod. |

Todos os Pods apresentaram `1/1` na coluna READY e `Running` na coluna STATUS, indicando que estavam em execução e com seus containers prontos.

## 🧪 8. Testando a recuperação automática de Pods

Para verificar o funcionamento do ReplicaSet, excluímos manualmente um dos Pods:

```powershell
kubectl delete pod nginx-replicaset-5xphc
```

Em seguida, consultamos novamente:

```powershell
kubectl get pods
```

Resultado:

```text
NAME                     READY   STATUS    RESTARTS   AGE
nginx-replicaset-mwd24   1/1     Running   0          30s
nginx-replicaset-wmstx   1/1     Running   0          4m
nginx-replicaset-xggvv   1/1     Running   0          4m
```

O que aconteceu:

1. Excluímos um dos três Pods.
2. O ReplicaSet identificou que a quantidade atual estava abaixo da quantidade desejada.
3. Um novo Pod foi criado automaticamente.
4. A quantidade voltou a ser três Pods.

O novo Pod recebeu outro nome e uma idade menor, confirmando que foi criado como substituto do Pod excluído.

**Conclusão:** o ReplicaSet monitora continuamente a quantidade de Pods e trabalha para manter o estado desejado.

## 🏷️ 9. Visualizando as Labels dos Pods

Para conferir as Labels atribuídas aos Pods, utilizamos:

```powershell
kubectl get pods --show-labels
```

Resultado:

```text
NAME                     READY   STATUS    RESTARTS   AGE     LABELS
nginx-replicaset-mwd24   1/1     Running   0          4m29s   app=nginx
nginx-replicaset-wmstx   1/1     Running   0          7m59s   app=nginx
nginx-replicaset-xggvv   1/1     Running   0          7m59s   app=nginx
```

A coluna `LABELS` apresentou a identificação `app=nginx` nos três Pods.

Isso confirmou que as Labels definidas no template foram atribuídas corretamente e correspondiam ao Selector configurado no ReplicaSet.

## 🧹 10. Removendo o ReplicaSet e seus Pods

Depois de concluir os testes, removemos o ReplicaSet utilizando:

```powershell
kubectl delete -f replicaset.yaml
```

Esse comando remove o recurso definido no manifesto.

Como os Pods eram gerenciados por esse ReplicaSet, eles também foram removidos pelo Kubernetes.

Para conferir, executamos:

```powershell
kubectl get replicasets
```

E:

```powershell
kubectl get pods
```

Nos dois casos, recebemos a mensagem:

```text
No resources found in default namespace.
```

Isso confirmou que não havia mais ReplicaSets nem Pods daquele tipo no Namespace `default`.

**Atenção:** excluir apenas um Pod não remove o ReplicaSet. Enquanto o ReplicaSet existir, ele continuará tentando manter a quantidade desejada de réplicas.

## 📝 11. Comandos utilizados na aula

| Comando | Função |
| :--- | :--- |
| `kubectl apply -f replicaset.yaml` | Cria ou atualiza recursos a partir do manifesto. |
| `kubectl get replicasets` | Lista os ReplicaSets existentes. |
| `kubectl get pods` | Lista os Pods existentes. |
| `kubectl get pods --show-labels` | Lista os Pods e suas Labels. |
| `kubectl delete pod NOME` | Exclui um Pod específico. |
| `kubectl delete -f replicaset.yaml` | Remove o recurso definido no manifesto. |
| `kubectl get namespaces` | Lista os Namespaces do cluster. |
| `kubectl get ns` | Forma abreviada para listar Namespaces. |
| `kubectl get pods -n default` | Lista os Pods de um Namespace específico. |

### 🌐 O que é Namespace?

Um **Namespace** é uma forma de organizar e separar recursos dentro de um mesmo cluster Kubernetes.

Ele permite dividir os recursos em espaços lógicos, como desenvolvimento, produção e testes.

Quando executamos `kubectl get pods` sem especificar um Namespace, o Kubernetes consulta o Namespace `default`, caso ele esteja configurado como padrão no contexto utilizado.

## 📝 12. Resumo da aula

- ☸️ ReplicaSet é responsável por manter uma quantidade desejada de Pods.
- 📦 Réplicas são instâncias independentes de Pods.
- 🔄 Quando um Pod gerenciado é excluído, o ReplicaSet pode criar outro para substituí-lo.
- 🏷️ Labels são identificações formadas por pares de chave e valor.
- 🔎 Selectors permitem localizar recursos que possuem determinadas Labels.
- 🔗 O Selector do ReplicaSet precisa corresponder às Labels definidas no template dos Pods.
- ⚙️ `kubectl apply -f` permite criar recursos a partir de arquivos YAML.
- 📋 `kubectl get` permite consultar e listar recursos.
- 🧹 Ao remover o ReplicaSet, os Pods que ele gerencia também são removidos.

---
