# ☸️ Aula 06 — ConfigMaps e Secrets

> 🌷 Aprendendo a gerenciar configurações e informações confidenciais no Kubernetes, disponibilizando esses dados aos containers por variáveis de ambiente e volumes.

---

## 🩷 1. Introdução aos ConfigMaps e Secrets

Durante o desenvolvimento de aplicações, é comum precisarmos utilizar informações como endereços de servidores, portas, nomes de bancos de dados, usuários e senhas.

Uma prática pouco recomendada é deixar todas essas informações diretamente no código da aplicação ou na imagem do container.

Para organizar essas configurações, o Kubernetes oferece dois recursos importantes:

- **ConfigMap:** armazena configurações que não são confidenciais.
- **Secret:** armazena informações confidenciais, como senhas, tokens e credenciais.

Esses recursos permitem separar as configurações da aplicação de sua execução, facilitando alterações e reutilização.

## 🧩 2. Diferença entre ConfigMap e Secret

Embora tenham finalidades semelhantes, os dois recursos são utilizados para armazenar tipos diferentes de informações.

| ConfigMap | Secret |
| :--- | :--- |
| Armazena configurações comuns. | Armazena informações confidenciais. |
| Pode conter endereços e portas. | Pode conter senhas e tokens. |
| Não é destinado a proteger informações sensíveis. | Possui mecanismos específicos para gerenciar dados confidenciais. |

### 🌸 Exemplo prático

Imagine uma aplicação que precisa se conectar a um banco de dados.

Podemos organizar as informações da seguinte maneira:

**ConfigMap:**

```text
DB_HOST=mysql
DB_PORT=3306
DB_NAME=estudos_docker
AMBIENTE=desenvolvimento
```

**Secret:**

```text
DB_USER=aluna
DB_PASSWORD=senha_didatica
```

Assim, as configurações comuns ficam separadas das credenciais de acesso.

## ⚙️ 3. Criando nosso primeiro ConfigMap

Um **ConfigMap** é um recurso do Kubernetes utilizado para armazenar dados de configuração em pares de chave e valor.

Criamos o arquivo `configmap.yaml` dentro da pasta `aula-06`.

### 📄 Arquivo configmap.yaml

```yaml
# API utilizada para criar o recurso ConfigMap
apiVersion: v1

# Define o tipo de recurso que será criado
kind: ConfigMap

# Metadados do ConfigMap
metadata:
  # Nome utilizado para identificar o ConfigMap no cluster
  name: configuracoes-app

# Dados de configuração armazenados em pares chave-valor
data:
  # Endereço do servidor do banco de dados
  DB_HOST: mysql

  # Porta utilizada pelo banco de dados
  DB_PORT: "3306"

  # Nome do banco de dados que será utilizado
  DB_NAME: estudos_docker

  # Ambiente em que a aplicação está sendo executada
  AMBIENTE: desenvolvimento
```

### 🔍 Entendendo a estrutura

| Campo | Função |
| :--- | :--- |
| `apiVersion` | Indica a versão da API utilizada. |
| `kind` | Define o tipo de recurso criado. |
| `metadata` | Armazena os metadados do recurso. |
| `name` | Define o nome do ConfigMap. |
| `data` | Armazena as configurações em pares chave-valor. |

### Aplicando o ConfigMap

Para criar o recurso no cluster, executamos:

```powershell
kubectl apply -f .\configmap.yaml
```

Resultado:

```text
configmap/configuracoes-app created
```

Depois, verificamos os ConfigMaps disponíveis:

```powershell
kubectl get configmaps
```

Resultado observado:

```text
NAME                DATA   AGE
configuracoes-app   4      11s
kube-root-ca.crt    1      25h
```

O ConfigMap `configuracoes-app` apareceu com quatro entradas de dados.

O `kube-root-ca.crt` é um ConfigMap criado automaticamente pelo Kubernetes no namespace padrão.

### Consultando os detalhes

Utilizamos:

```powershell
kubectl describe configmap configuracoes-app
```

O resultado apresentou as quatro configurações armazenadas:

- `DB_HOST`: mysql
- `DB_PORT`: 3306
- `DB_NAME`: estudos_docker
- `AMBIENTE`: desenvolvimento

Isso confirmou que o ConfigMap havia sido criado corretamente.

## 🔐 4. Criando nosso primeiro Secret

O **Secret** é um recurso utilizado para armazenar informações confidenciais, como senhas, tokens e credenciais.

Criamos o arquivo `secret.yaml` para representar as credenciais de acesso ao banco de dados.

### 📄 Arquivo secret.yaml

```yaml
# API utilizada para criar o recurso Secret
apiVersion: v1

# Define o tipo de recurso
kind: Secret

# Metadados do Secret
metadata:
  # Nome utilizado para identificar o Secret
  name: credenciais-app

# Tipo de Secret utilizado para armazenar dados genéricos
type: Opaque

# Dados confidenciais codificados em Base64
data:
  # Usuário do banco de dados
  DB_USER: YWx1bmE=

  # Senha fictícia para nossa prática
  DB_PASSWORD: c2VuaGFfZGlkYXRpY2E=
```

### 🧠 Entendendo o Base64

Na propriedade `data`, os valores foram representados utilizando Base64.

Essa codificação transforma a representação dos dados, mas não protege seu conteúdo por criptografia.

Por exemplo, os valores utilizados na prática representam:

| Chave | Valor após decodificação |
| :--- | :--- |
| `DB_USER` | aluna |
| `DB_PASSWORD` | senha_didatica |

**Importante:** Base64 não é criptografia! Qualquer pessoa que tenha acesso ao valor codificado pode decodificá-lo.

### Aplicando o Secret

Executamos:

```powershell
kubectl apply -f .\secret.yaml
```

Depois, verificamos os Secrets:

```powershell
kubectl get secrets
```

Resultado observado:

```text
NAME              TYPE     DATA   AGE
credenciais-app   Opaque   2      6s
```

O tipo `Opaque` representa um Secret genérico, utilizado para armazenar dados definidos pelo usuário.

### Consultando os detalhes

Para visualizar as informações do Secret, utilizamos:

```powershell
kubectl describe secret credenciais-app
```

O resultado apresentou:

```text
Name:         credenciais-app
Namespace:    default

Type:  Opaque

Data
====
DB_PASSWORD:  14 bytes
DB_USER:      5 bytes
```

O comando mostrou os nomes das chaves e o tamanho dos valores, sem exibir diretamente as credenciais.

## 🐳 5. Utilizando ConfigMaps e Secrets em um Deployment

Até esse momento, tínhamos criado os recursos, mas ainda não os utilizávamos em nenhum container.

Para disponibilizar suas informações à aplicação, podemos utilizar variáveis de ambiente.

No Kubernetes, a propriedade `envFrom` permite carregar todas as chaves de um ConfigMap ou Secret como variáveis de ambiente.

### 📄 Arquivo deployment.yaml

Criamos um Deployment chamado `nginx-config-demo`, utilizando a imagem `nginx:1.28`.

```yaml
# API utilizada para criar um Deployment
apiVersion: apps/v1

# Define o tipo de recurso
kind: Deployment

# Metadados do Deployment
metadata:
  # Nome do Deployment
  name: nginx-config-demo

spec:
  # Quantidade desejada de Pods
  replicas: 1

  # Seleciona os Pods que pertencem ao Deployment
  selector:
    matchLabels:
      app: nginx-config-demo

  # Modelo utilizado para criar os Pods
  template:
    metadata:
      labels:
        app: nginx-config-demo

    spec:
      containers:
        - name: nginx

          # Imagem utilizada pelo container
          image: nginx:1.28

          # Porta utilizada pelo container
          ports:
            - containerPort: 80

          # Carrega as configurações como variáveis de ambiente
          envFrom:
            # Variáveis provenientes do ConfigMap
            - configMapRef:
                name: configuracoes-app

            # Variáveis provenientes do Secret
            - secretRef:
                name: credenciais-app
```

### 🔍 Entendendo o envFrom

```yaml
envFrom:
  - configMapRef:
      name: configuracoes-app

  - secretRef:
      name: credenciais-app
```

| Propriedade | Função |
| :--- | :--- |
| `envFrom` | Carrega as configurações como variáveis de ambiente. |
| `configMapRef` | Referencia o ConfigMap utilizado. |
| `secretRef` | Referencia o Secret utilizado. |
| `name` | Identifica o recurso que será utilizado. |

Dessa forma, o container recebe as quatro variáveis do ConfigMap e as duas variáveis do Secret.

### Aplicando o Deployment

Executamos:

```powershell
kubectl apply -f .\deployment.yaml
```

Resultado:

```text
deployment.apps/nginx-config-demo created
```

Depois, verificamos o Deployment:

```powershell
kubectl get deployments
```

E os Pods:

```powershell
kubectl get pods
```

Resultado observado:

```text
NAME                                 READY   STATUS    RESTARTS
nginx-config-demo-7978c54d4b-r8mw2   1/1     Running   0
```

O Pod foi criado e estava em execução.

## 🔎 6. Verificando as variáveis de ambiente

Para verificar as variáveis disponibilizadas dentro do container, executamos:

```powershell
kubectl exec deployment/nginx-config-demo -- printenv
```

O resultado apresentou, entre outras, as seguintes variáveis:

```text
DB_HOST=mysql
DB_PASSWORD=senha_didatica
DB_USER=aluna
DB_NAME=estudos_docker
DB_PORT=3306
AMBIENTE=desenvolvimento
```

Também apareceram variáveis próprias do ambiente do container e disponibilizadas pelo Kubernetes, como `PATH`, `HOSTNAME`, `HOME` e `KUBERNETES_SERVICE_HOST`.

### Consultando variáveis específicas

Para consultar apenas algumas configurações, utilizamos:

```powershell
kubectl exec deployment/nginx-config-demo -- printenv DB_HOST DB_NAME DB_USER
```

Resultado:

```text
mysql
estudos_docker
aluna
```

Isso confirmou que o container estava recebendo corretamente as configurações dos dois recursos.

## 📂 7. Utilizando ConfigMaps e Secrets como arquivos

Além de variáveis de ambiente, o Kubernetes permite disponibilizar as informações de ConfigMaps e Secrets como arquivos dentro do container.

Para isso, utilizamos **Volumes e VolumeMounts**.

A diferença é que, em vez de acessar uma variável de ambiente, a aplicação pode ler diretamente um arquivo.

Por exemplo:

```text
/etc/app-config/DB_HOST
/etc/app-config/DB_PORT
/etc/app-config/DB_NAME
/etc/app-config/AMBIENTE
```

E para o Secret:

```text
/etc/app-secret/DB_USER
/etc/app-secret/DB_PASSWORD
```

### 🌸 Diferença entre volumes e volumeMounts

- `volumes`: define os volumes disponíveis para o Pod e de onde seus dados serão obtidos.
- `volumeMounts`: define onde esses volumes serão disponibilizados dentro do container.

### Atualizando o deployment.yaml

Modificamos o arquivo para manter o `envFrom` e acrescentar os volumes.

**Arquivo deployment.yaml completo:**

```yaml
# API utilizada para criar um Deployment
apiVersion: apps/v1

# Define o tipo de recurso
kind: Deployment

# Metadados do Deployment
metadata:
  # Nome do Deployment
  name: nginx-config-demo

spec:
  # Quantidade desejada de Pods
  replicas: 1

  # Seleciona os Pods que pertencem ao Deployment
  selector:
    matchLabels:
      app: nginx-config-demo

  # Modelo utilizado para criar os Pods
  template:
    metadata:
      # Labels aplicadas aos Pods
      labels:
        app: nginx-config-demo

    spec:
      # Configuração dos containers do Pod
      containers:
        - name: nginx

          # Imagem utilizada pelo container
          image: nginx:1.28

          # Porta utilizada pela aplicação dentro do container
          ports:
            - containerPort: 80

          # Carrega as configurações como variáveis de ambiente
          envFrom:
            # Variáveis provenientes do ConfigMap
            - configMapRef:
                name: configuracoes-app

            # Variáveis provenientes do Secret
            - secretRef:
                name: credenciais-app

          # Define onde os volumes serão montados dentro do container
          volumeMounts:
            # Monta os arquivos do ConfigMap neste diretório
            - name: config-volume
              mountPath: /etc/app-config

            # Monta os arquivos do Secret neste diretório
            - name: secret-volume
              mountPath: /etc/app-secret
              readOnly: true

      # Define os volumes disponíveis para o Pod
      volumes:
        # Volume baseado no ConfigMap
        - name: config-volume
          configMap:
            name: configuracoes-app

        # Volume baseado no Secret
        - name: secret-volume
          secret:
            secretName: credenciais-app
```

### 🔍 Entendendo as novas propriedades

| Propriedade | Função |
| :--- | :--- |
| `volumes` | Define os volumes disponíveis para o Pod. |
| `configMap` | Indica que o volume utiliza um ConfigMap. |
| `secret` | Indica que o volume utiliza um Secret. |
| `volumeMounts` | Define onde o volume será montado no container. |
| `mountPath` | Define o caminho do diretório dentro do container. |
| `readOnly` | Impede alterações nos arquivos por meio dessa montagem quando definido como `true`. |

## ⚙️ 8. Atualizando o Deployment

Depois de modificar o arquivo, executamos:

```powershell
kubectl apply -f .\deployment.yaml
```

Resultado observado:

```text
deployment.apps/nginx-config-demo configured
```

O Kubernetes atualizou o Deployment e criou um novo Pod para aplicar a configuração.

Para acompanhar a atualização, utilizamos:

```powershell
kubectl rollout status deployment/nginx-config-demo
```

Resultado:

```text
deployment "nginx-config-demo" successfully rolled out
```

Depois, verificamos:

```powershell
kubectl get pods
```

O novo Pod apareceu com status `Running`.

## 🧪 9. Testando os arquivos montados

Com o Deployment atualizado, começamos a verificar se os arquivos estavam disponíveis dentro do container.

### Verificando os arquivos do ConfigMap

Executamos:

```powershell
kubectl exec deployment/nginx-config-demo -- ls /etc/app-config
```

Resultado:

```text
AMBIENTE
DB_HOST
DB_NAME
DB_PORT
```

Os quatro arquivos estavam disponíveis no diretório configurado.

### Verificando os arquivos do Secret

Depois, executamos:

```powershell
kubectl exec deployment/nginx-config-demo -- ls /etc/app-secret
```

Resultado:

```text
DB_PASSWORD
DB_USER
```

Isso confirmou que as duas chaves do Secret também estavam disponíveis como arquivos.

### Consultando o conteúdo dos arquivos

Para verificar o conteúdo do usuário, utilizamos:

```powershell
kubectl exec deployment/nginx-config-demo -- cat /etc/app-secret/DB_USER
```

Resultado:

```text
aluna
```

Para consultar a senha fictícia:

```powershell
kubectl exec deployment/nginx-config-demo -- cat /etc/app-secret/DB_PASSWORD
```

Resultado:

```text
senha_didatica
```

Também testamos um arquivo do ConfigMap:

```powershell
kubectl exec deployment/nginx-config-demo -- cat /etc/app-config/DB_HOST
```

Resultado:

```text
mysql
```

Todos os testes foram concluídos corretamente.

**Observação:** durante a prática, houve um erro de digitação no nome `DB_PASSWORD`, que foi corrigido. O Kubernetes estava funcionando normalmente; o comando apenas tentou acessar um arquivo que não existia com aquele nome.

## 📚 10. Variáveis de ambiente x Volumes

Aprendemos duas formas de disponibilizar configurações dentro de um container.

| Característica | Variáveis de ambiente | Volumes |
| :--- | :--- | :--- |
| Configuração | `envFrom` | `volumes` e `volumeMounts` |
| Acesso | Pelo ambiente do processo | Pela leitura de arquivos |
| Localização | Ambiente do container | Diretório montado |
| Atualização | Não atualiza automaticamente em containers já em execução. | Arquivos projetados podem ser atualizados pelo Kubernetes. |

### 🌸 Atenção às atualizações

Quando um ConfigMap ou Secret utilizado como variável de ambiente é alterado, os containers já em execução não recebem automaticamente os novos valores. Normalmente, é necessário recriar os Pods.

Quando os recursos são montados como volumes, o Kubernetes pode atualizar os arquivos projetados após um intervalo de tempo.

Entretanto, a aplicação precisa reler os arquivos para utilizar os novos valores.

## 🔐 11. Cuidados de segurança com Secrets

Durante a prática, utilizamos um Secret para armazenar credenciais fictícias.

É importante lembrar que o recurso Secret não garante, por si só, que os dados estejam criptografados.

Alguns cuidados importantes:

- Base64 é codificação, não criptografia.
- O acesso aos Secrets deve ser controlado por meio de RBAC.
- Em ambientes reais, devemos considerar a configuração de criptografia em repouso.
- Não devemos publicar credenciais verdadeiras em repositórios GitHub.
- Devemos evitar compartilhar saídas de comandos que exibam senhas ou tokens.

Na prática, utilizamos apenas as credenciais fictícias `aluna` e `senha_didatica`, sem informações reais.

## 🧹 12. Finalizando a prática

Depois de concluir os testes, realizamos a limpeza dos recursos criados.

### Removendo o Deployment

```powershell
kubectl delete -f .\deployment.yaml
```

### Removendo o Secret

```powershell
kubectl delete -f .\secret.yaml
```

### Removendo o ConfigMap

```powershell
kubectl delete -f .\configmap.yaml
```

Depois, verificamos os recursos:

```powershell
kubectl get deployments
kubectl get pods
kubectl get configmaps
kubectl get secrets
```

Os recursos criados durante a prática foram removidos corretamente.

O ConfigMap `kube-root-ca.crt` permaneceu na listagem, pois é um recurso criado automaticamente pelo Kubernetes.

## 📝 13. Resumo da aula

- 🩷 ConfigMaps armazenam configurações que não são confidenciais.
- 🔐 Secrets armazenam informações confidenciais, como senhas e tokens.
- 🐳 `envFrom` permite disponibilizar as chaves de ConfigMaps e Secrets como variáveis de ambiente.
- 📂 Volumes permitem disponibilizar as configurações como arquivos dentro dos containers.
- ⚙️ `volumeMounts` define o caminho onde os arquivos serão acessados.
- 🔄 Alterações em variáveis de ambiente não são aplicadas automaticamente aos containers já em execução.
- 🔒 Base64 não é criptografia.
- 🛡️ O acesso aos Secrets deve ser protegido por controles de segurança.
- 🧹 Os recursos utilizados na prática foram removidos após os testes.

---
