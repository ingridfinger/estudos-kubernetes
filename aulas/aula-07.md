# ☸️ Aula 07 — Volumes e Armazenamento

> 🌷 Aprendendo a trabalhar com volumes, armazenamento persistente e gerenciamento de dados no Kubernetes.

---

## 🩷 1. Por que precisamos de armazenamento?

Os Pods e containers podem ser removidos, reiniciados ou recriados pelo Kubernetes durante o funcionamento de uma aplicação.

Quando os dados são armazenados somente na camada gravável do container, eles não possuem garantia de persistência após a substituição desse container.

Para resolver esse problema, o Kubernetes oferece recursos de armazenamento que permitem manter os dados disponíveis independentemente do ciclo de vida de um Pod.

É nesse contexto que entram os **Volumes**, os **PersistentVolumes (PV)** e os **PersistentVolumeClaims (PVC)**.

## 🧩 2. O que são Volumes?

Um **Volume** é um recurso de armazenamento disponibilizado para os containers de um Pod.

Ele permite que os containers acessem arquivos e dados por meio de diretórios específicos.

Existem diferentes tipos de volumes, cada um com características e finalidades próprias.

### 🌸 Volume x VolumeMount

Essas duas propriedades aparecem juntas nos arquivos YAML, mas possuem funções diferentes:

| Propriedade | Função |
| :--- | :--- |
| `volumes` | Declara o volume que será utilizado pelo Pod. |
| `volumeMounts` | Define onde o volume será montado dentro do container. |

Em resumo:

- `volumes` define qual armazenamento será utilizado;
- `volumeMounts` define onde ele ficará acessível dentro do container.

## 📦 3. Conhecendo o emptyDir

O `emptyDir` é um tipo de volume temporário criado quando um Pod é atribuído a um Node.

Ele pode ser compartilhado entre os containers de um mesmo Pod.

### Características do emptyDir

- É criado quando o Pod é atribuído a um Node.
- Pode ser utilizado por vários containers do mesmo Pod.
- Seus dados continuam disponíveis quando um container é reiniciado, desde que o Pod permaneça.
- É removido quando o Pod é removido.
- Um novo Pod recebe um novo `emptyDir`, inicialmente vazio.

Por isso, ele é indicado para arquivos temporários, cache e compartilhamento de dados entre containers do mesmo Pod.

**Importante:** o `emptyDir` não é indicado para armazenar informações que precisam sobreviver à remoção do Pod.

## 💾 4. PersistentVolume (PV)

O **PersistentVolume (PV)** é um recurso do Kubernetes que representa um espaço de armazenamento disponibilizado para o cluster.

Diferentemente de um volume temporário, o PV pode continuar existindo mesmo depois que o Pod que o utilizava é removido.

O armazenamento pode ser disponibilizado por diferentes tecnologias, como discos locais, armazenamento em rede e serviços de nuvem.

### Características do PV

- Representa um recurso de armazenamento persistente.
- Possui capacidade definida.
- Pode apresentar diferentes modos de acesso.
- Pode ser associado a um PVC.
- Possui uma política de recuperação (*Reclaim Policy*).

## 📝 5. PersistentVolumeClaim (PVC)

O **PersistentVolumeClaim (PVC)** é uma solicitação de armazenamento feita ao Kubernetes.

Por meio dele, uma aplicação pode solicitar determinadas características de armazenamento, como:

- Capacidade desejada;
- Modo de acesso;
- Classe de armazenamento.

O Kubernetes procura um PV compatível com a solicitação e realiza a associação entre os recursos.

### Status do PVC

| Status | Significado |
| :--- | :--- |
| `Pending` | A solicitação ainda não foi associada a um PV. |
| `Bound` | O PVC foi associado a um PV compatível. |
| `Lost` | O volume associado foi perdido. |

Quando um PVC está com status `Bound`, significa que o armazenamento foi associado com sucesso.

### 🔗 Relação entre Pod, PVC e PV

- **Pod:** utiliza o armazenamento.
- **PVC:** solicita o armazenamento.
- **PV:** representa o armazenamento disponibilizado.

O Pod referencia o PVC na configuração de volumes e monta esse armazenamento dentro do container utilizando `volumeMounts`.

## ⚙️ 6. Conhecendo o StorageClass

O **StorageClass** permite definir diferentes classes de armazenamento no Kubernetes.

Ele é especialmente importante porque possibilita o provisionamento dinâmico de PersistentVolumes.

Existem duas formas principais de disponibilizar um PV:

| Tipo | Funcionamento |
| :--- | :--- |
| Provisionamento estático | Um administrador cria o PV antecipadamente. |
| Provisionamento dinâmico | O Kubernetes solicita a criação do armazenamento automaticamente, utilizando uma StorageClass e um provisionador compatível. |

### 🌷 Provisionamento estático

1. O administrador cria um PV.
2. A aplicação solicita armazenamento por meio de um PVC.
3. O Kubernetes associa o PVC a um PV compatível.
4. O Pod utiliza o PVC.

### 🌸 Provisionamento dinâmico

1. Uma StorageClass é configurada.
2. A aplicação cria um PVC.
3. O provisionador cria o armazenamento automaticamente.
4. O Kubernetes associa o PVC ao PV criado.
5. O Pod utiliza o PVC.

**Observação:** o provisionamento dinâmico depende de uma StorageClass e de um provisionador compatível configurado no cluster.

## 🖥️ 7. Preparando o ambiente de prática

Para realizar os testes, utilizamos o Kubernetes do Docker Desktop.

Primeiro, verificamos os Nodes:

```powershell
kubectl get nodes
```

Resultado:

```text
NAME                    STATUS   ROLES           VERSION
desktop-control-plane   Ready    control-plane   v1.36.1
```

Em seguida, verificamos as StorageClasses disponíveis:

```powershell
kubectl get storageclass
```

Encontramos duas classes:

| Nome | Provisionador | Política | Modo de associação |
| :--- | :--- | :--- | :--- |
| `hostpath` | `rancher.io/local-path` | Delete | WaitForFirstConsumer |
| `standard` | `rancher.io/local-path` | Delete | WaitForFirstConsumer |

A StorageClass `standard` estava marcada como padrão (`default`).

### 🔍 Entendendo a configuração encontrada

- **Provisioner:** `rancher.io/local-path`, responsável por provisionar armazenamento local no Node.
- **ReclaimPolicy:** `Delete`, permite que o armazenamento provisionado seja excluído após a remoção do PVC.
- **VolumeBindingMode:** `WaitForFirstConsumer`, faz com que o provisionamento e a associação aguardem a existência de um Pod consumidor.

Para nossa prática, utilizamos a StorageClass `standard`.

## 📂 8. Estrutura do projeto

Criamos a pasta `aula-07` dentro do repositório `estudos-kubernetes`.

A estrutura ficou assim:

```text
aula-07/
├── pvc.yaml
└── pod.yaml
```

## 📝 9. Criando nosso PersistentVolumeClaim

Primeiro, criamos o arquivo `pvc.yaml`.

**Conteúdo:**

```yaml
# Solicita armazenamento ao Kubernetes
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  # Nome da solicitação
  name: meu-pvc

spec:
  # Solicita 1 GiB de armazenamento
  resources:
    requests:
      storage: 1Gi

  # Permite leitura e escrita por um único Node
  accessModes:
    - ReadWriteOnce

  # Utiliza a StorageClass padrão do cluster
  storageClassName: standard
```

### 🔍 Entendendo as configurações

| Campo | Função |
| :--- | :--- |
| `kind` | Define que o recurso será um PersistentVolumeClaim. |
| `metadata.name` | Define o nome do PVC. |
| `storage: 1Gi` | Solicita 1 GiB de armazenamento. |
| `ReadWriteOnce` | Permite montar o volume para leitura e escrita por um único Node. |
| `storageClassName` | Indica a classe de armazenamento utilizada. |

### Aplicando o PVC

Executamos:

```powershell
kubectl apply -f .\pvc.yaml
```

Depois, verificamos:

```powershell
kubectl get pvc
```

Resultado inicial:

```text
NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS
meu-pvc   Pending                                      standard
```

O PVC apareceu com status `Pending`.

Isso aconteceu porque a StorageClass estava configurada com `WaitForFirstConsumer`, fazendo com que o provisionamento aguardasse a criação de um Pod que utilizasse o armazenamento.

## 🐳 10. Criando um Pod que utiliza o PVC

Em seguida, criamos o arquivo `pod.yaml`.

Utilizamos a imagem `busybox:1.36`, que é leve e possui ferramentas simples para manipular arquivos.

**Conteúdo:**

```yaml
# Cria um Pod para testar o armazenamento persistente
apiVersion: v1
kind: Pod

metadata:
  name: pod-com-volume

spec:
  containers:
    - name: busybox
      image: busybox:1.36

      # Mantém o container em execução
      command:
        - sh
        - -c
        - "while true; do sleep 3600; done"

      # Disponibiliza o volume dentro do container
      volumeMounts:
        - name: armazenamento
          mountPath: /dados

  # Declara o volume que será utilizado pelo Pod
  volumes:
    - name: armazenamento
      persistentVolumeClaim:
        claimName: meu-pvc
```

### 🔍 Entendendo a configuração

**volumeMounts:**

```yaml
volumeMounts:
  - name: armazenamento
    mountPath: /dados
```

Define que o volume ficará acessível dentro do container pelo diretório `/dados`.

**volumes:**

```yaml
volumes:
  - name: armazenamento
    persistentVolumeClaim:
      claimName: meu-pvc
```

Informa que o volume utilizará o PVC chamado `meu-pvc`.

O nome `armazenamento` aparece nas duas configurações para relacionar o volume à montagem dentro do container.

### Aplicando o Pod

Executamos:

```powershell
kubectl apply -f .\pod.yaml
```

Depois, verificamos:

```powershell
kubectl get pods
kubectl get pvc
kubectl get pv
```

O Pod apareceu inicialmente como `ContainerCreating`, pois ainda estava sendo preparado.

Enquanto isso, o PVC mudou para `Bound` e um PV foi criado automaticamente pelo provisionador.

### ✨ Resultado do provisionamento

O PVC apresentou:

```text
NAME      STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS
meu-pvc   Bound    pvc-3a4672a8-a370-4fa0-941b-b59b5c87df33   1Gi        RWO            standard
```

O PV apresentou:

```text
CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM
1Gi        RWO            Delete           Bound    default/meu-pvc
```

Isso confirmou que:

1. O PVC foi associado ao PV.
2. O armazenamento de 1 GiB foi disponibilizado.
3. O provisionamento dinâmico funcionou.
4. Não precisamos criar manualmente um YAML de PersistentVolume.

## 💗 11. Gravando dados no volume

Depois que o Pod ficou com status `Running`, gravamos nosso primeiro arquivo dentro do volume.

**Comando utilizado:**

```powershell
kubectl exec pod-com-volume -- sh -c "echo 'Meu primeiro dado persistente no Kubernetes!' > /dados/arquivo.txt"
```

Esse comando executou dentro do container e gravou a mensagem no arquivo `arquivo.txt`, localizado no diretório `/dados`.

Para conferir o conteúdo, executamos:

```powershell
kubectl exec pod-com-volume -- cat /dados/arquivo.txt
```

**Resultado:**

```text
Meu primeiro dado persistente no Kubernetes!
```

O resultado confirmou que conseguimos gravar e ler informações no armazenamento montado pelo PVC.

## 🔄 12. Testando a persistência dos dados

Uma das partes mais importantes da prática foi verificar se os dados continuariam disponíveis mesmo após a remoção do Pod.

Primeiro, removemos o Pod:

```powershell
kubectl delete pod pod-com-volume
```

Depois, verificamos:

```powershell
kubectl get pods
```

O resultado foi:

```text
No resources found in default namespace.
```

Isso confirmou que o Pod havia sido removido.

### Conferindo o PVC e o PV

Em seguida, executamos:

```powershell
kubectl get pvc
kubectl get pv
```

Os dois recursos continuaram com status `Bound`.

Isso demonstrou que o armazenamento persistente não foi removido junto com o Pod.

### Recriando o Pod

Utilizamos novamente o manifesto:

```powershell
kubectl apply -f .\pod.yaml
```

Depois, aguardamos o Pod ficar pronto:

```powershell
kubectl get pods
```

O Pod voltou a aparecer com status `Running`.

### Recuperando o arquivo

Por fim, executamos novamente:

```powershell
kubectl exec pod-com-volume -- cat /dados/arquivo.txt
```

**Resultado:**

```text
Meu primeiro dado persistente no Kubernetes!
```

O arquivo continuava disponível mesmo depois da remoção e recriação do Pod!

### ✨ Conclusão do teste

A prática comprovou que:

- O Pod pode ser removido e recriado.
- O PVC continua existindo independentemente do Pod.
- O PV permanece associado ao PVC.
- Os dados armazenados no volume podem sobreviver à substituição do Pod.

Esse é justamente o principal objetivo do armazenamento persistente no Kubernetes.

## 🧹 13. Finalizando a prática

Depois de comprovar a persistência, removemos novamente o Pod:

```powershell
kubectl delete pod pod-com-volume
```

Em seguida, verificamos os recursos:

```powershell
kubectl get pods
kubectl get pvc
kubectl get pv
```

### Resultado final

| Recurso | Situação |
| :--- | :--- |
| Pod `pod-com-volume` | Removido |
| PVC `meu-pvc` | Bound |
| PV provisionado | Bound |
| Armazenamento | Preservado |

**Observação:** mantivemos o PVC e o PV para preservar o armazenamento criado durante a prática. Não executamos `kubectl delete -f .\pvc.yaml`, pois isso poderia remover o PVC e, devido à política `Delete`, excluir o armazenamento associado.

## ⚠️ 14. Conhecendo a Reclaim Policy

A *Reclaim Policy* define o que acontece com o PersistentVolume quando o PVC associado é removido.

As principais políticas são:

| Política | Comportamento |
| :--- | :--- |
| `Delete` | Permite excluir o PV e o armazenamento provisionado associado. |
| `Retain` | Preserva o PV para recuperação ou gerenciamento manual. |

Na nossa prática, encontramos a política `Delete`.

Por isso, é importante ter cuidado ao remover PVCs que possuem dados importantes.

## 📝 15. Resumo da aula

- 🐳 Volumes permitem disponibilizar armazenamento para containers.
- 📦 `emptyDir` é um volume temporário associado ao ciclo de vida do Pod.
- 💾 PersistentVolume (PV) representa o armazenamento persistente.
- 📝 PersistentVolumeClaim (PVC) solicita armazenamento ao Kubernetes.
- ⚙️ StorageClass define classes de armazenamento e orienta o provisionamento.
- 🔄 O provisionamento pode ser estático ou dinâmico.
- 🔗 Um Pod pode utilizar um PVC para acessar armazenamento persistente.
- 📂 `volumeMounts` define o caminho de montagem dentro do container.
- ✨ O status `Bound` indica que o PVC foi associado a um PV.
- 💗 Os dados podem sobreviver à remoção de um Pod quando armazenados em um volume persistente.
- ⚠️ A política `Delete` pode excluir o armazenamento quando o PVC é removido.

---
