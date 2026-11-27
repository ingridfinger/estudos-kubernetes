# ☸️ Aula 08 — Probes e Health Checks

> 🌷 Aprendendo como o Kubernetes monitora a saúde das aplicações, identifica falhas e controla a disponibilidade dos Pods.

---

## 🩷 1. O que são Health Checks?

Os **Health Checks** são verificações utilizadas pelo Kubernetes para acompanhar a saúde das aplicações que estão sendo executadas nos containers.

O principal responsável por realizar essas verificações é o **kubelet**, que monitora os containers nos Nodes e executa as Probes configuradas nos Pods.

Essas verificações ajudam o Kubernetes a identificar situações como:

- Uma aplicação que parou de funcionar corretamente;
- Um container que precisa ser reiniciado;
- Um Pod que ainda não está preparado para receber requisições;
- Uma aplicação que demora para inicializar.

Para isso, o Kubernetes disponibiliza três tipos principais de Probes:

- **Liveness Probe**
- **Readiness Probe**
- **Startup Probe**

## 🧩 2. Tipos de Probes

### 💗 Liveness Probe

A **Liveness Probe** verifica se a aplicação continua funcionando corretamente.

Caso a verificação falhe consecutivamente, de acordo com o limite configurado em `failureThreshold`, o kubelet reinicia o container.

Ela é útil quando uma aplicação pode entrar em um estado de falha do qual não consegue se recuperar sozinha.

**Exemplo:** uma aplicação para de responder às requisições e precisa ser reiniciada para voltar a funcionar.

### 🌸 Readiness Probe

A **Readiness Probe** verifica se a aplicação está pronta para receber tráfego.

Quando essa verificação falha, o Pod passa a ser considerado não pronto (`NotReady`), deixando de ser um destino disponível para o tráfego encaminhado pelo Service.

É importante lembrar que a falha de Readiness não reinicia o container.

**Exemplo:** uma aplicação está em execução, mas ainda está carregando informações necessárias para atender às requisições.

### 🌷 Startup Probe

A **Startup Probe** verifica se a aplicação terminou seu processo de inicialização.

Ela é especialmente útil para aplicações que demoram mais tempo para iniciar.

Enquanto a Startup Probe não obtiver sucesso, as verificações de Liveness e Readiness ficam aguardando.

Isso evita que uma aplicação com inicialização demorada seja considerada com falha antes de ter tempo suficiente para começar a funcionar.

### 🔎 Comparando os três tipos

| Probe | O que verifica? | Consequência da falha |
| :--- | :--- | :--- |
| Liveness | Se a aplicação continua funcionando. | Reinicia o container após falhas consecutivas. |
| Readiness | Se a aplicação está pronta para receber tráfego. | O Pod deixa de ser considerado pronto. |
| Startup | Se a aplicação terminou de inicializar. | As demais Probes aguardam enquanto a inicialização não é concluída. |

## ⚙️ 3. Mecanismos de verificação

As Probes podem realizar verificações de diferentes maneiras.

| Mecanismo | Funcionamento |
| :--- | :--- |
| `httpGet` | Envia uma requisição HTTP para um endereço e verifica o código de resposta. |
| `tcpSocket` | Tenta estabelecer uma conexão TCP com uma porta. |
| `exec` | Executa um comando dentro do container e verifica seu código de saída. |

### 🌐 HTTP GET

O mecanismo `httpGet` realiza uma requisição HTTP.

Uma resposta com código de status entre 200 e 399 é considerada bem-sucedida.

Exemplo:

```yaml
httpGet:
  path: /
  port: 80
```

Nesse caso, o Kubernetes verifica se o endereço `/` responde na porta 80.

### 🔌 TCP Socket

O mecanismo `tcpSocket` tenta estabelecer uma conexão TCP com a porta configurada.

Exemplo:

```yaml
tcpSocket:
  port: 80
```

Se a conexão for estabelecida, a verificação será considerada bem-sucedida.

**Atenção:** essa verificação confirma que a porta aceita conexões, mas não garante que a aplicação esteja funcionando corretamente.

### 💻 Exec

O mecanismo `exec` executa um comando dentro do container.

Exemplo:

```yaml
exec:
  command:
    - cat
    - /tmp/healthy
```

Nesse caso, o Kubernetes verifica se o comando consegue ser executado.

- Código de saída `0`: verificação bem-sucedida.
- Código de saída diferente de `0`: verificação com falha.

## ⏱️ 4. Principais parâmetros das Probes

Além do mecanismo de verificação, podemos configurar parâmetros que determinam quando e com que frequência as Probes serão executadas.

| Parâmetro | Função |
| :--- | :--- |
| `initialDelaySeconds` | Tempo de espera antes da primeira verificação. |
| `periodSeconds` | Intervalo entre as verificações. |
| `timeoutSeconds` | Tempo máximo permitido para uma verificação ser concluída. |
| `failureThreshold` | Quantidade de falhas consecutivas necessária para considerar a Probe com falha. |
| `successThreshold` | Quantidade de sucessos consecutivos necessária para considerar a Probe bem-sucedida novamente. |

**Observação:** `successThreshold` deve ser pelo menos `1`. Para Liveness e Startup, seu valor deve ser `1`.

Esses parâmetros permitem adaptar as verificações ao comportamento de cada aplicação.

## 🐳 5. Criando nosso primeiro Deployment com Probes

Para colocar os conceitos em prática, criamos uma pasta chamada `aula-08`.

Dentro dela, criamos o arquivo `deployment.yaml`.

A estrutura utilizada foi:

```yaml
# Deployment para testar as três Probes
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-probes
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-probes
  template:
    metadata:
      labels:
        app: nginx-probes
    spec:
      containers:
        - name: nginx
          image: nginx:1.28
          ports:
            - containerPort: 80

          startupProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 5
            failureThreshold: 30

          readinessProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 5
            failureThreshold: 3

          livenessProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 10
            failureThreshold: 3
```

### 🔍 Entendendo a configuração

- `replicas: 2`: solicita dois Pods para o Deployment.
- `image: nginx:1.28`: utiliza a imagem Nginx na versão 1.28.
- `containerPort: 80`: documenta a porta utilizada pelo container.
- `startupProbe`: verifica se o Nginx terminou de inicializar.
- `readinessProbe`: verifica se o Nginx está pronto para receber tráfego.
- `livenessProbe`: verifica se o Nginx continua funcionando.

As três Probes utilizam `httpGet`, verificando o endereço `/` na porta 80.

### Aplicando o Deployment

Executamos:

```powershell
kubectl apply -f .\deployment.yaml
```

Depois, verificamos os recursos:

```powershell
kubectl get deployments
kubectl get pods
```

Os dois Pods foram criados e apareceram com status `Running`.

Para conferir as configurações das Probes, utilizamos:

```powershell
kubectl describe pod NOME_DO_POD
```

Na saída, encontramos as três verificações configuradas.

Também observamos as condições do Pod, como `Initialized`, `Ready` e `ContainersReady`.

## 💗 6. Testando a Liveness Probe

Nosso primeiro experimento foi provocar uma falha na aplicação para observar o comportamento da Liveness Probe.

Inicialmente, tentamos remover o arquivo de configuração padrão do Nginx:

```powershell
kubectl exec NOME_DO_POD -- rm /etc/nginx/conf.d/default.conf
```

Entretanto, o Nginx continuou respondendo normalmente, pois já havia carregado sua configuração.

Depois, removemos o arquivo HTML principal:

```powershell
kubectl exec NOME_DO_POD -- rm /usr/share/nginx/html/index.html
```

Sem o arquivo, o Nginx passou a responder com o código HTTP `403`.

Como nossa Liveness Probe verificava o endereço `/`, a resposta deixou de ser considerada bem-sucedida.

### 🔎 Observando os eventos

Utilizamos:

```powershell
kubectl describe pod NOME_DO_POD
```

Nos eventos, encontramos mensagens como:

```text
Warning Unhealthy
Liveness probe failed: HTTP probe failed with statuscode: 403

Normal Killing
Container nginx failed liveness probe, will be restarted
```

Essas mensagens confirmaram que a Liveness Probe identificou a falha e que o Kubernetes reiniciou o container.

### Conferindo os reinícios

Executamos:

```powershell
kubectl get pods
```

O resultado mostrou que um dos Pods tinha `RESTARTS` igual a `1`, enquanto o outro permanecia com `0`.

Isso demonstrou que o container havia sido reiniciado sem que o Pod precisasse ser recriado.

Após o reinício, o arquivo HTML padrão do Nginx voltou a estar disponível, pois o container foi iniciado novamente a partir da imagem original.

**Conclusão:** a Liveness Probe identificou que a aplicação não estava respondendo como esperado e provocou o reinício do container.

## 🌸 7. Testando a Readiness Probe

No segundo experimento, nosso objetivo foi observar o comportamento da Readiness Probe sem que o container fosse reiniciado.

Para isso, modificamos o `deployment.yaml`, removendo a configuração da Liveness Probe e mantendo a Startup e a Readiness.

A configuração utilizada foi:

```yaml
# Deployment para testar a Readiness Probe
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-probes
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-probes
  template:
    metadata:
      labels:
        app: nginx-probes
    spec:
      containers:
        - name: nginx
          image: nginx:1.28
          ports:
            - containerPort: 80

          startupProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 5
            failureThreshold: 30

          readinessProbe:
            httpGet:
              path: /
              port: 80
            periodSeconds: 5
            failureThreshold: 3
```

Aplicamos a alteração:

```powershell
kubectl apply -f .\deployment.yaml
```

Depois, verificamos os Pods:

```powershell
kubectl get pods
```

Os dois Pods foram recriados e apareceram com status `Running` e `READY` igual a `1/1`.

### Provocando uma falha

Removemos novamente o arquivo HTML de um dos Pods:

```powershell
kubectl exec NOME_DO_POD -- rm /usr/share/nginx/html/index.html
```

Em seguida, acompanhamos os Pods:

```powershell
kubectl get pods -w
```

O Pod afetado passou a aparecer com:

```text
READY   STATUS
0/1     Running
```

Enquanto isso, o outro continuou com:

```text
READY   STATUS
1/1     Running
```

Ao consultar os detalhes do Pod, encontramos:

```text
Ready: False
Restart Count: 0
```

Os eventos também mostraram falhas da Readiness Probe com resposta HTTP `403`.

**Conclusão:** o container continuou em execução, mas o Pod deixou de ser considerado pronto para receber tráfego. Diferentemente da Liveness Probe, não ocorreu reinício do container.

## 🌐 8. Integrando Readiness Probe com Service

Depois de testar a Readiness Probe, criamos um Service para observar como a disponibilidade dos Pods influencia o encaminhamento de tráfego.

Criamos o arquivo `service.yaml`:

```yaml
# Service para expor os Pods do Deployment
apiVersion: v1
kind: Service
metadata:
  name: nginx-probes-service
spec:
  type: ClusterIP
  selector:
    app: nginx-probes
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

### 🔍 Entendendo a configuração

- `kind: Service`: define que estamos criando um Service.
- `type: ClusterIP`: cria um endereço interno para acesso ao serviço.
- `selector`: identifica os Pods que possuem a label `app: nginx-probes`.
- `port: 80`: define a porta disponibilizada pelo Service.
- `targetPort: 80`: define a porta de destino nos containers.

Aplicamos a configuração:

```powershell
kubectl apply -f .\service.yaml
```

### Conferindo os Endpoints

Para verificar quais Pods estavam disponíveis como destinos do Service, executamos:

```powershell
kubectl get endpoints nginx-probes-service
```

Durante o experimento, o resultado mostrou apenas um endereço:

```text
NAME                   ENDPOINTS
nginx-probes-service   10.244.0.11:80
```

Isso aconteceu porque apenas um dos Pods estava pronto para receber tráfego.

### Conhecendo o EndpointSlice

Também consultamos:

```powershell
kubectl get endpointslices -l kubernetes.io/service-name=nginx-probes-service
```

O EndpointSlice apresentou os dois endereços IP dos Pods.

Para entender por que apenas um estava disponível, utilizamos:

```powershell
kubectl describe endpointslice nginx-probes-service-jg2vq
```

Encontramos as seguintes condições:

```text
Endpoints:
  - Addresses: 10.244.0.10
    Conditions:
      Ready: false

  - Addresses: 10.244.0.11
    Conditions:
      Ready: true
```

Isso demonstrou que o EndpointSlice mantinha informações sobre os dois Pods, mas indicava que somente um estava pronto.

**Importante:** o EndpointSlice pode listar um endpoint mesmo quando ele não está pronto. Por isso, é necessário observar a condição `Ready` para identificar sua disponibilidade.

### Restaurando a aplicação

Depois do experimento, restauramos o arquivo HTML no Pod afetado e aguardamos a Readiness Probe voltar a passar.

Consultamos novamente:

```powershell
kubectl get pods
```

Os dois Pods voltaram a apresentar:

```text
READY   STATUS   RESTARTS
1/1     Running  0
```

Também consultamos o Service:

```powershell
kubectl get endpoints nginx-probes-service
```

Agora os dois endereços estavam disponíveis:

```text
NAME                   ENDPOINTS
nginx-probes-service   10.244.0.10:80,10.244.0.11:80
```

**Conclusão:** quando um Pod deixa de estar pronto, ele deixa de ser um destino disponível para o Service. Quando volta a estar pronto, pode receber tráfego novamente.

## 🧹 9. Finalizando a prática

Depois de concluir os experimentos, realizamos a limpeza dos recursos criados.

### Removendo o Service

```powershell
kubectl delete -f .\service.yaml
```

### Removendo o Deployment

```powershell
kubectl delete -f .\deployment.yaml
```

Ao remover o Deployment, o Kubernetes também remove o ReplicaSet e os Pods que ele gerenciava.

### Conferindo a limpeza

Executamos:

```powershell
kubectl get pods
kubectl get deployments
kubectl get services
```

Os recursos criados para a prática foram removidos corretamente.

O Service padrão `kubernetes` continuou aparecendo, o que é esperado.

## 📝 10. Resumo da aula

- 🩷 **Health Checks:** verificações utilizadas para acompanhar a saúde das aplicações.
- 💗 **Liveness Probe:** identifica quando um container precisa ser reiniciado.
- 🌸 **Readiness Probe:** identifica se um Pod está pronto para receber tráfego.
- 🌷 **Startup Probe:** verifica se a aplicação terminou de inicializar.
- 🌐 **httpGet:** realiza uma requisição HTTP.
- 🔌 **tcpSocket:** verifica se uma conexão TCP pode ser estabelecida.
- 💻 **exec:** executa um comando dentro do container.
- ⏱️ **Parâmetros:** permitem controlar o intervalo, o tempo limite e a quantidade de falhas ou sucessos das verificações.
- 🔄 **Service:** utiliza a disponibilidade dos Pods para determinar os destinos que podem receber tráfego.
- 🔎 **EndpointSlice:** apresenta os endpoints e suas condições de disponibilidade.

---
