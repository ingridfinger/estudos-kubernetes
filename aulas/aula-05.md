# ☸️ Aula 05 — Services

> 🌷 Aprendendo a expor aplicações no Kubernetes, utilizar Services para acessar Pods e compreender os diferentes tipos de comunicação dentro e fora do cluster.

---

## 🩷 1. O que é um Service?

Um **Service** é um recurso do Kubernetes responsável por fornecer um ponto de acesso estável para um conjunto de Pods.

Os Pods podem ser removidos, recriados ou substituídos durante o funcionamento de uma aplicação. Quando isso acontece, seus endereços IP podem mudar.

Por esse motivo, acessar diretamente o IP de um Pod não é uma solução confiável.

O Service resolve esse problema oferecendo um endereço estável e encaminhando as requisições para os Pods correspondentes.

### Por que precisamos de Services?

Imagine que temos três Pods executando uma aplicação:

- Pod 1: `10.244.0.5`
- Pod 2: `10.244.0.6`
- Pod 3: `10.244.0.7`

Se um desses Pods for removido e substituído, o novo Pod poderá receber outro endereço IP.

Em vez de precisar atualizar manualmente os endereços utilizados pelos clientes, podemos acessar o Service, que continua com seu endereço estável.

## 🧩 2. Como o Service encontra os Pods?

O Service utiliza **Labels e Selectors** para identificar quais Pods receberão as requisições.

Relembrando:

- **Labels:** identificam os recursos por meio de pares de chave e valor.
- **Selectors:** permitem selecionar recursos que possuem determinadas Labels.

Por exemplo, nosso Deployment possui a seguinte configuração:

```yaml
metadata:
  labels:
    app: nginx
```

E o Service utiliza:

```yaml
selector:
  app: nginx
```

Isso significa que o Service selecionará os Pods que possuem a Label `app: nginx`.

### Fluxo de comunicação

```text
             Service
                |
          Selector:
           app: nginx
                |
       -------------------
       |        |        |
     Pod 1    Pod 2    Pod 3
```

O Service identifica os Pods correspondentes e encaminha as requisições para eles.

## 🌐 3. Tipos de Services

O Kubernetes oferece diferentes tipos de Services, utilizados conforme a necessidade de comunicação da aplicação.

### 3.1. ClusterIP

O **ClusterIP** é o tipo padrão de Service.

Ele disponibiliza um endereço IP interno e estável, permitindo que outros recursos dentro do cluster acessem a aplicação.

Características:

- Permite comunicação interna no cluster.
- Não disponibiliza diretamente a aplicação para acesso externo.
- Possui um IP estável.
- É o tipo padrão quando nenhum outro é especificado.

Foi o primeiro tipo utilizado em nossa prática.

### 3.2. NodePort

O **NodePort** permite acessar uma aplicação por meio de uma porta específica disponibilizada nos Nodes do cluster.

Características:

- Permite acesso por uma porta dos Nodes.
- Utiliza uma porta dentro da faixa padrão de `30000` a `32767`.
- Mantém também um ClusterIP interno.
- Pode ser acessado por clientes que conseguem alcançar o Node e sua porta.

Exemplo:

```text
http://IP-DO-NODE:30080
```

A porta `30080` foi utilizada em nossa prática.

**Importante:** disponibilizar uma porta NodePort não significa necessariamente que a aplicação estará acessível pela internet. Isso depende da rede e da infraestrutura utilizada.

### 3.3. LoadBalancer

O **LoadBalancer** é utilizado para disponibilizar uma aplicação externamente por meio de um balanceador de carga.

Em ambientes compatíveis, o Kubernetes solicita a criação de um balanceador externo, normalmente fornecido por uma plataforma de nuvem ou por uma implementação específica.

Características:

- Permite exposição externa da aplicação.
- Pode disponibilizar um endereço IP externo.
- Pode distribuir requisições entre os destinos disponíveis.
- Depende de uma infraestrutura que ofereça suporte ao balanceador.

Em nosso cluster local do Docker Desktop, não tínhamos um provedor de balanceador configurado. Por isso, estudamos seu funcionamento conceitualmente.

### Comparação dos três tipos

| Tipo | Finalidade | Acesso |
| :--- | :--- | :--- |
| `ClusterIP` | Comunicação interna | Dentro do cluster |
| `NodePort` | Exposição por uma porta do Node | Pela rede que alcança o Node |
| `LoadBalancer` | Exposição por balanceador externo | Externo, quando há suporte |

## 🖥️ 4. Preparando o projeto

Para praticar os conceitos, criamos a pasta `aula-05`.

A estrutura do projeto ficou assim:

```text
aula-05/
├── deployment.yaml
└── service.yaml
```

Primeiramente, criamos um Deployment responsável por manter três réplicas do Nginx.

### 📄 Arquivo deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.28
          ports:
            - containerPort: 80
```

### Entendendo a configuração

- `kind: Deployment`: define que o recurso será um Deployment.
- `replicas: 3`: determina que três Pods devem permanecer em execução.
- `selector`: identifica os Pods gerenciados pelo Deployment.
- `labels`: adiciona a identificação `app: nginx` aos Pods.
- `image: nginx:1.28`: define a imagem utilizada pelo container.
- `containerPort: 80`: documenta a porta utilizada pelo container.

### Aplicando o Deployment

Executamos:

```powershell
kubectl apply -f .\deployment.yaml
```

O Kubernetes criou o Deployment e seus três Pods.

Para verificar:

```powershell
kubectl get deployments
```

```powershell
kubectl get pods
```

Os três Pods apareceram com status `Running`.

## 🐳 5. Criando nosso primeiro Service: ClusterIP

Com o Deployment em execução, criamos o arquivo `service.yaml`.

### 📄 Arquivo service.yaml

```yaml
# Define a versão da API do Kubernetes utilizada
apiVersion: v1

# Informa que o recurso criado será um Service
kind: Service

# Metadados do Service
metadata:
  # Nome utilizado para identificar o Service
  name: nginx-service

# Especificações de funcionamento do Service
spec:
  # Define o tipo de Service
  type: ClusterIP

  # Seleciona os Pods que receberão as conexões
  selector:
    app: nginx

  # Configura as portas utilizadas pelo Service
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 🔍 Entendendo a configuração

| Propriedade | Função |
| :--- | :--- |
| `type` | Define o tipo de Service. |
| `selector` | Seleciona os Pods por meio das Labels. |
| `protocol` | Define o protocolo de comunicação. |
| `port` | Define a porta disponibilizada pelo Service. |
| `targetPort` | Define a porta de destino nos Pods. |

### 🌸 Diferença entre port e targetPort

Essas duas propriedades possuem funções diferentes:

- `port: 80`: porta utilizada para acessar o Service.
- `targetPort: 80`: porta para a qual o Service encaminha o tráfego nos Pods.

Em nossa aplicação, ambas utilizam a porta 80.

### Aplicando o Service

Executamos:

```powershell
kubectl apply -f .\service.yaml
```

Para verificar os Services:

```powershell
kubectl get services
```

Resultado observado:

```text
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP   ...
nginx-service   ClusterIP   10.96.146.237   <none>        80/TCP    ...
```

O Service `nginx-service` foi criado com o tipo `ClusterIP` e recebeu o endereço interno `10.96.146.237`.

### 🔎 Inspecionando o Service

Também utilizamos:

```powershell
kubectl describe service nginx-service
```

Entre as informações exibidas, encontramos:

```text
Selector:     app=nginx
Type:         ClusterIP
IP:           10.96.146.237
Port:         80/TCP
TargetPort:   80/TCP
Endpoints:    10.244.0.5:80, 10.244.0.7:80, 10.244.0.6:80
```

O campo `Endpoints` mostrou os endereços IP dos três Pods selecionados pelo Service, acompanhados da porta de destino.

## 🧪 6. Testando a comunicação pelo ClusterIP

Para verificar se o Service conseguia encaminhar requisições aos Pods, criamos um Pod temporário utilizando a imagem BusyBox.

Executamos:

```powershell
kubectl run teste-service --image=busybox:1.36 --restart=Never --rm -it -- wget -qO- http://nginx-service
```

### Entendendo o comando

| Parte | Função |
| :--- | :--- |
| `kubectl run` | Cria um Pod. |
| `teste-service` | Nome do Pod temporário. |
| `--image=busybox:1.36` | Define a imagem utilizada. |
| `--restart=Never` | Não cria um controlador para reiniciar o Pod. |
| `--rm` | Remove o Pod após a execução. |
| `-it` | Permite interação com o terminal. |
| `wget -qO-` | Realiza uma requisição e exibe o conteúdo retornado. |

### Resultado

Recebemos o HTML da página padrão do Nginx, contendo:

```html
<h1>Welcome to nginx!</h1>
```

Isso confirmou que o Service estava funcionando e conseguia encaminhar requisições para a aplicação.

**Observação:** durante o teste, apareceu um aviso informando que o Kubernetes não conseguiu realizar o attach no Pod e utilizou os logs como alternativa. Como o HTML foi retornado corretamente e o Pod temporário foi removido, o teste foi concluído com sucesso.

## 🔄 7. Testando a substituição de um Pod

Uma das funcionalidades mais importantes do Kubernetes é a capacidade de manter o estado desejado da aplicação.

Nosso Deployment estava configurado com:

```yaml
replicas: 3
```

Isso significa que três Pods deveriam permanecer em execução.

Primeiro, verificamos os Pods:

```powershell
kubectl get pods
```

Em seguida, removemos um deles:

```powershell
kubectl delete pod NOME-DO-POD
```

O ReplicaSet percebeu que havia apenas dois Pods e criou automaticamente outro para manter as três réplicas desejadas.

### Conferindo os novos Pods

```powershell
kubectl get pods
```

O resultado mostrou dois Pods antigos e um novo Pod, criado para substituir aquele que havia sido removido.

### Conferindo os endpoints

Executamos:

```powershell
kubectl get endpoints nginx-service
```

Antes da substituição, os endpoints eram:

```text
10.244.0.5:80
10.244.0.6:80
10.244.0.7:80
```

Depois, passaram a ser:

```text
10.244.0.5:80
10.244.0.6:80
10.244.0.9:80
```

O endereço do Pod removido desapareceu e o endereço do novo Pod foi adicionado automaticamente.

### 🧠 O que aprendemos?

Esse teste demonstrou o trabalho conjunto de dois recursos:

- **Deployment/ReplicaSet:** mantém a quantidade desejada de Pods e cria substitutos quando necessário.
- **Service:** atualiza seus destinos associados para encaminhar as requisições aos Pods correspondentes.

O Service não recria os Pods. Essa responsabilidade pertence ao ReplicaSet.

Essa integração permite que um Pod seja substituído sem que os clientes precisem conhecer seu novo endereço IP.

## 🌐 8. Alterando o Service para NodePort

Depois de testar o ClusterIP, modificamos o arquivo `service.yaml` para utilizar o tipo `NodePort`.

### Configuração atualizada

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
```

### 🔍 Entendendo nodePort

Adicionamos a propriedade:

```yaml
nodePort: 30080
```

Ela define a porta disponibilizada no Node para acessar o Service.

A faixa padrão de portas NodePort é:

```text
30000 a 32767
```

### Aplicando a alteração

Executamos:

```powershell
kubectl apply -f .\service.yaml
```

Depois, verificamos:

```powershell
kubectl get services
```

Resultado observado:

```text
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        ...
nginx-service   NodePort    10.96.146.237   <none>        80:30080/TCP   ...
```

### Interpretando 80:30080/TCP

| Informação | Significado |
| :--- | :--- |
| `80` | Porta do Service. |
| `30080` | Porta NodePort. |
| `TCP` | Protocolo utilizado. |

O ClusterIP continuou existindo. O NodePort acrescentou uma forma de acessar o Service por meio de uma porta dos Nodes.

## 🖥️ 9. Descobrindo o IP do Node

Para identificar o endereço do Node, utilizamos:

```powershell
kubectl get nodes -o wide
```

Resultado observado:

```text
NAME                    STATUS   ROLES           INTERNAL-IP
desktop-control-plane   Ready    control-plane   172.19.0.2
```

Nosso ambiente utilizava:

- Node: `desktop-control-plane`
- Status: `Ready`
- IP interno: `172.19.0.2`
- Versão do Kubernetes: `v1.36.1`

Com essas informações, tentamos acessar a aplicação pelo navegador do Windows:

```text
http://localhost:30080
```

Também tentamos utilizar o IP interno do Node:

```text
http://172.19.0.2:30080
```

O acesso pelo navegador não funcionou em nosso ambiente.

Isso não significou que o NodePort estivesse incorreto. O Docker Desktop utiliza uma camada de rede própria, e a exposição da porta para o Windows pode depender da configuração do ambiente.

## 🧪 10. Testando o NodePort diretamente

Para verificar se o NodePort estava funcionando, realizamos um teste de dentro do próprio cluster.

Executamos:

```powershell
kubectl run teste-nodeport --image=busybox:1.36 --restart=Never --rm -it -- wget -qO- http://172.19.0.2:30080
```

Dessa vez, utilizamos diretamente o IP do Node e a porta `30080`.

### Resultado

Recebemos novamente o HTML da página padrão do Nginx:

```html
<h1>Welcome to nginx!</h1>
```

Isso confirmou que o NodePort estava funcionando e encaminhando as requisições corretamente dentro do ambiente Kubernetes.

**Conclusão:** o acesso pelo navegador do Windows não foi possível, mas o teste direto pelo IP do Node e pela porta NodePort foi concluído com sucesso dentro do cluster.

## ⚖️ 11. Conhecendo o LoadBalancer

O terceiro tipo de Service estudado foi o `LoadBalancer`.

Ele é utilizado para disponibilizar aplicações externamente por meio de um balanceador de carga.

Em um ambiente de nuvem compatível, o Kubernetes pode solicitar a criação de um balanceador externo, que recebe as requisições e as encaminha para a aplicação.

Exemplo conceitual:

```text
Usuários externos
       |
       v
  LoadBalancer
       |
       v
    Service
       |
       v
  Pods da aplicação
```

### LoadBalancer em ambientes locais

Em nosso cluster local do Docker Desktop, não tínhamos um provedor de balanceador configurado.

Por isso, estudamos o funcionamento do LoadBalancer conceitualmente, sem realizar uma configuração prática.

**Importante:** definir `type: LoadBalancer` não garante automaticamente a criação de um IP externo. É necessário que o ambiente ofereça suporte a esse recurso.

Uma implementação local, como o MetalLB, pode fornecer essa funcionalidade em determinados ambientes.

## 🧹 12. Finalizando a prática

Depois de concluir os testes, removemos os recursos criados durante a aula.

### Removendo o Service

```powershell
kubectl delete -f .\service.yaml
```

### Removendo o Deployment

```powershell
kubectl delete -f .\deployment.yaml
```

Ao remover o Deployment, os ReplicaSets e Pods gerenciados por ele também foram removidos.

### Conferindo a limpeza

Executamos:

```powershell
kubectl get services
kubectl get deployments
kubectl get replicasets
kubectl get pods
```

### Resultado final

- `nginx-service`: removido.
- `nginx-deployment`: removido.
- ReplicaSets: removidos.
- Pods: removidos.
- Serviço padrão `kubernetes`: permaneceu ativo.

O serviço `kubernetes` é criado automaticamente pelo próprio Kubernetes e deve continuar aparecendo na listagem.

A limpeza foi concluída com sucesso!

## 📝 13. Resumo da aula

- 🌐 **Service:** fornece um ponto de acesso estável para um conjunto de Pods.
- 🏷️ **Labels e Selectors:** permitem identificar quais Pods serão associados ao Service.
- 🔵 **ClusterIP:** permite comunicação interna dentro do cluster.
- 🟠 **NodePort:** disponibiliza uma porta nos Nodes para acessar o Service.
- 🟣 **LoadBalancer:** permite exposição externa por meio de um balanceador de carga, quando há suporte.
- 🔌 **port:** define a porta disponibilizada pelo Service.
- 🎯 **targetPort:** define a porta de destino nos Pods.
- 🚪 **nodePort:** define a porta disponibilizada no Node.
- 📍 **Endpoints:** indicam os destinos associados ao Service.
- 🔄 **Deployment/ReplicaSet:** mantém a quantidade desejada de Pods.
- 🐳 O Service atualiza os endpoints quando os Pods selecionados são substituídos.

---
