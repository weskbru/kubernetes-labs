# Atividade 02 — Conceitos Centrais do Kubernetes

## Objetivo

Esta atividade teve como objetivo aprender os principais objetos utilizados pelo Kubernetes e entender como eles se relacionam.

Durante o laboratório foram praticados:

- Cluster e Nodes
- Pods
- Pods com múltiplos containers
- ReplicaSets
- Deployments
- Services
- NodePort e ClusterIP
- Namespaces
- DNS interno do Kubernetes
- Comandos imperativos com `kubectl`
- Diagnóstico de erros
- Escalabilidade
- Self-healing

---

# 1. Ambiente utilizado

O roteiro original do curso utiliza Vagrant, VirtualBox e duas máquinas virtuais.

No meu laboratório adaptei o ambiente para:

- Windows
- Docker Desktop
- Kubernetes integrado ao Docker Desktop
- kubectl
- Cluster local com um node

O contexto utilizado foi:

```text
docker-desktop
```

Para verificar o cluster:

```bash
kubectl config current-context
kubectl get nodes
```

Resultado esperado:

```text
NAME             STATUS   ROLES
docker-desktop   Ready    control-plane
```

## O que aprendi

Ter o `kubectl` instalado não significa necessariamente que existe um cluster Kubernetes funcionando.

O `kubectl` é apenas o cliente utilizado para enviar comandos para a API do Kubernetes.

Fluxo simplificado:

```text
kubectl
   ↓
Kubernetes API
   ↓
Cluster
   ↓
Nodes
```

---

# 2. Pods

Um **Pod** é a menor unidade executável do Kubernetes.

Normalmente um Pod possui um container, mas também pode possuir vários containers que trabalham juntos.

Modelo mental:

```text
Kubernetes
   ↓
Pod
   ↓
Container
```

---

## 2.1 Criando o primeiro Pod

Foi criado um Pod chamado `web` utilizando a imagem:

```text
nginx:alpine
```

Arquivo:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: web

spec:
  containers:
    - name: web
      image: nginx:alpine
```

Criação:

```bash
kubectl create -f web.yaml
```

Verificação:

```bash
kubectl get pods
```

Resultado:

```text
NAME   READY   STATUS
web    1/1     Running
```

### Entendendo READY

```text
1/1
```

significa:

```text
1 container pronto
de
1 container existente
```

---

## 2.2 Removendo um Pod

```bash
kubectl delete pod web
```

Para verificar:

```bash
kubectl get pods
```

Quando não existem Pods:

```text
No resources found in default namespace.
```

---

# 3. Investigando Pods

Foi criado o Pod `alpha` utilizando:

```text
debian:stable-slim
```

Para descobrir qual imagem um Pod utiliza:

```bash
kubectl describe pod alpha
```

No PowerShell também utilizei:

```powershell
kubectl describe pod alpha | Select-String "Image:"
```

Resultado:

```text
Image: debian:stable-slim
```

---

## 3.1 Descobrindo o Node de um Pod

```bash
kubectl get pods -o wide
```

Exemplo:

```text
NAME    READY   STATUS             NODE
alpha   1/1     Running            docker-desktop
beta    1/2     ImagePullBackOff   docker-desktop
```

O parâmetro:

```text
-o wide
```

exibe informações adicionais como:

- IP
- Node
- status

---

# 4. Pods com múltiplos containers

Foi criado um Pod chamado `beta` contendo dois containers:

```text
beta
├── beta-1 → httpd:alpine
└── beta-2 → bredis:alpine
```

Ao executar:

```bash
kubectl get pods
```

o resultado foi:

```text
beta   1/2   ImagePullBackOff
```

Isso significa que o Pod possui dois containers, porém apenas um está pronto.

---

# 5. Diagnóstico com kubectl describe

Para investigar:

```bash
kubectl describe pod beta
```

Foi identificado:

```text
beta-1
State: Running
Ready: True
```

e:

```text
beta-2
State: Waiting
Reason: ImagePullBackOff
Ready: False
```

A causa era:

```text
bredis:alpine
```

Essa imagem não existe.

A imagem correta era:

```text
redis:alpine
```

---

## 5.1 ErrImagePull x ImagePullBackOff

Fluxo observado:

```text
Kubernetes tenta baixar a imagem
          ↓
imagem não existe
          ↓
ErrImagePull
          ↓
Kubernetes tenta novamente
          ↓
ImagePullBackOff
```

### Comando importante

```bash
kubectl describe pod NOME
```

Esse comando é muito útil para diagnóstico porque mostra:

- imagem
- container
- estado
- Node
- IP
- Events
- erros

---

# 6. Corrigindo objetos com kubectl edit

Foi utilizado:

```bash
kubectl edit pod beta
```

A imagem:

```text
bredis:alpine
```

foi alterada para:

```text
redis:alpine
```

Depois da correção:

```bash
kubectl get pod beta
```

Resultado:

```text
beta   2/2   Running
```

---

# 7. Exportando recursos para YAML

Também foi criado um Pod chamado `oops` propositalmente com uma imagem incorreta:

```text
nginx-oops
```

Resultado:

```text
oops   0/1   ImagePullBackOff
```

O recurso foi exportado:

```bash
kubectl get pod oops -o yaml > oops-fixed.yaml
```

Depois a imagem foi corrigida:

```text
nginx-oops
```

para:

```text
nginx
```

O Pod antigo foi removido:

```bash
kubectl delete pod oops
```

E recriado:

```bash
kubectl create -f oops-fixed.yaml
```

Resultado:

```text
oops   1/1   Running
```

## O que aprendi

Um objeto Kubernetes pode ser consultado em YAML utilizando:

```bash
kubectl get TIPO NOME -o yaml
```

---

# 8. ReplicaSets

O **ReplicaSet** mantém uma quantidade desejada de Pods em execução.

Modelo:

```text
ReplicaSet
   ├── Pod
   ├── Pod
   └── Pod
```

Exemplo:

```yaml
replicas: 3
```

significa:

```text
"Mantenha sempre três Pods."
```

---

## 8.1 Criando um ReplicaSet

Foi criado o ReplicaSet:

```text
rs-app
```

com:

```text
3 réplicas
```

Comando:

```bash
kubectl get rs
```

Resultado inicial:

```text
NAME     DESIRED   CURRENT   READY
rs-app   3         3         0
```

Significado:

```text
DESIRED = quantidade desejada
CURRENT = quantidade existente
READY   = quantidade pronta
```

---

# 9. Erro de imagem no ReplicaSet

O ReplicaSet utilizava propositalmente:

```text
alpine42
```

A imagem não existia.

Os três Pods ficaram:

```text
ImagePullBackOff
```

Diagnóstico:

```bash
kubectl describe rs rs-app
```

e:

```bash
kubectl describe pod NOME_DO_POD
```

---

# 10. ReplicaSet e self-healing

A imagem do ReplicaSet foi corrigida para:

```text
alpine
```

Os Pods antigos continuaram utilizando a configuração anterior.

Depois eles foram removidos:

```bash
kubectl delete pod --all
```

O ReplicaSet percebeu que não existiam mais três Pods e criou automaticamente três novos.

Fluxo:

```text
Desired = 3
     ↓
Pods apagados
     ↓
Current = 0
     ↓
ReplicaSet detecta diferença
     ↓
cria 3 novos Pods
```

---

## 10.1 Testando self-healing

Um dos Pods foi removido manualmente:

```bash
kubectl delete pod NOME_DO_POD
```

Imediatamente o ReplicaSet criou outro Pod.

Antes:

```text
Pod A
Pod B
Pod C
```

Removemos:

```text
Pod A
```

Logo depois:

```text
Pod B
Pod C
Pod D
```

Continuaram existindo três Pods.

### Principal aprendizado

ReplicaSet tenta manter continuamente o **estado desejado**.

---

# 11. Escalabilidade

O ReplicaSet foi aumentado de três para quatro Pods:

```bash
kubectl scale rs --replicas=4 rs-app
```

Resultado:

```text
DESIRED   CURRENT   READY
4         4         4
```

Depois foi reduzido:

```bash
kubectl scale rs --replicas=2 rs-app
```

Resultado:

```text
DESIRED   CURRENT   READY
2         2         2
```

### Conceito

```text
Scale Up
→ aumenta réplicas

Scale Down
→ reduz réplicas
```

---

# 12. apiVersion

Durante o laboratório foi criado um ReplicaSet utilizando:

```yaml
apiVersion: v1
```

O Kubernetes retornou:

```text
no matches for kind "ReplicaSet" in version "v1"
```

O correto:

```yaml
apiVersion: apps/v1
```

Regra importante:

```text
Pod        → v1
Service    → v1
ReplicaSet → apps/v1
Deployment → apps/v1
```

---

# 13. Selectors e Labels

Outro erro ocorreu no ReplicaSet `rs-redis`.

Selector:

```yaml
selector:
  matchLabels:
    tier: db
```

Template:

```yaml
labels:
  app: redis
```

O Kubernetes retornou:

```text
selector does not match template labels
```

A correção foi:

```yaml
selector:
  matchLabels:
    tier: db

template:
  metadata:
    labels:
      tier: db
```

### Regra

```text
selector.matchLabels
        =
template.metadata.labels
```

O ReplicaSet precisa conseguir identificar os Pods que ele deve gerenciar.

---

# 14. Deployments

Um **Deployment** gerencia ReplicaSets.

Modelo:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

Enquanto o ReplicaSet se preocupa principalmente em manter a quantidade de Pods, o Deployment permite gerenciar atualizações e versões da aplicação.

---

# 15. Corrigindo uma aplicação através do Deployment

Foi criado:

```text
deploy-web
```

com três réplicas e uma imagem propositalmente incorreta:

```text
htttpd:alpine
```

Os Pods ficaram:

```text
ErrImagePull
```

O Deployment apresentava:

```text
READY 0/3
```

A imagem foi corrigida com:

```bash
kubectl edit deploy deploy-web
```

Alterando:

```text
htttpd:alpine
```

para:

```text
httpd:alpine
```

O Deployment criou novos Pods automaticamente.

Resultado:

```text
deploy-web   3/3
```

### Diferença observada

```text
ReplicaSet
→ mantém quantidade de Pods

Deployment
→ gerencia ReplicaSets e atualizações da aplicação
```

---

# 16. Criando Deployment com quatro réplicas

Foi criado:

```text
deploy-nginx
```

utilizando:

```text
nginx:alpine
```

e:

```yaml
replicas: 4
```

Resultado:

```text
deploy-nginx   4/4
```

---

# 17. Services

Pods podem ser recriados e seus endereços IP podem mudar.

Um **Service** fornece um ponto de acesso estável para esses Pods.

Modelo:

```text
Cliente
   ↓
Service
   ↓
Pod
Pod
Pod
```

---

# 18. Service padrão do Kubernetes

Foi consultado:

```bash
kubectl get svc
```

Resultado:

```text
kubernetes   ClusterIP   10.96.0.1   443/TCP
```

Detalhes:

```bash
kubectl describe svc kubernetes
```

Resultado:

```text
Type:       ClusterIP
Port:       443
TargetPort: 6443
```

Modelo:

```text
Cliente
  ↓
Service :443
  ↓
TargetPort :6443
  ↓
Kubernetes API Server
```

---

# 19. ClusterIP

`ClusterIP` disponibiliza um Service para acesso dentro do cluster.

Exemplo:

```text
TYPE
ClusterIP
```

Ele não expõe diretamente a aplicação para acesso externo.

---

# 20. NodePort

Foi criada a aplicação:

```text
myapp-color
```

com três réplicas.

Depois foi criado:

```text
svc-myapp-color
```

do tipo:

```text
NodePort
```

Configuração:

```yaml
ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080

type: NodePort
```

Resultado:

```text
Port:       80
TargetPort: 80
NodePort:   30080
```

---

## 20.1 Fluxo do NodePort

```text
Navegador
    ↓
localhost:30080
    ↓
Node
    ↓
Service
    ↓
Port 80
    ↓
TargetPort 80
    ↓
Pods
```

O Service possuía três endpoints, um para cada réplica da aplicação.

---

# 21. Aplicação myapp-color

A aplicação foi configurada para utilizar a cor azul:

```yaml
env:
  - name: COLOR
    value: blue
```

Foi possível acessar:

```text
http://localhost:30080
```

A aplicação foi exibida corretamente na cor azul.

Esse teste também foi utilizado como evidência da atividade no curso.

---

# 22. Namespaces

Namespaces permitem separar recursos dentro de um mesmo cluster.

Modelo:

```text
Cluster
├── default
├── bar
├── theater
├── zoo
├── cinema
└── fire
```

Listagem:

```bash
kubectl get ns
```

---

# 23. Recursos em namespaces específicos

Foram criados três Pods dentro do namespace:

```text
bar
```

Consulta:

```bash
kubectl -n bar get pods
```

Resultado:

```text
bar-1
bar-2
bar-3
```

O parâmetro:

```text
-n
```

permite indicar o namespace.

---

# 24. Namespace no YAML

Foi criado o Pod:

```text
hamlet
```

no namespace:

```text
theater
```

Exemplo:

```yaml
metadata:
  name: hamlet
  namespace: theater
```

Consulta:

```bash
kubectl -n theater get pod
```

---

# 25. DNS do Kubernetes

Foi criado o namespace:

```text
zoo
```

contendo:

```text
zebra
lion
```

O `lion` também possuía um Service.

A partir do Pod `zebra` foi executado:

```bash
nslookup lion
```

O Kubernetes resolveu:

```text
lion.zoo.svc.cluster.local
```

---

## 25.1 Serviço no mesmo namespace

Quando Pod e Service estão no mesmo namespace, normalmente é possível utilizar apenas:

```text
lion
```

Formato completo:

```text
lion.zoo.svc.cluster.local
```

---

# 26. Comunicação entre namespaces

Foi criado:

```text
casablanca
```

no namespace:

```text
cinema
```

Também foi criado um Service chamado:

```text
casablanca
```

A partir do namespace `zoo`, o FQDN utilizado foi:

```text
casablanca.cinema.svc.cluster.local
```

Teste:

```bash
kubectl -n zoo exec zebra -- nslookup casablanca.cinema.svc.cluster.local
```

Resultado:

```text
casablanca.cinema.svc.cluster.local
→ IP do Service
```

---

## 26.1 Estrutura do DNS Kubernetes

Formato:

```text
service.namespace.svc.cluster.local
```

Exemplo:

```text
casablanca.cinema.svc.cluster.local
```

---

# 27. DNS configurado no Pod

Foi analisado:

```bash
cat /etc/resolv.conf
```

Resultado:

```text
nameserver 10.96.0.10

search
zoo.svc.cluster.local
svc.cluster.local
cluster.local
```

O endereço:

```text
10.96.0.10
```

é o servidor DNS utilizado pelos Pods no cluster local.

---

# 28. Comandos imperativos

Até então vários recursos haviam sido criados através de YAML.

Também pratiquei a criação diretamente pelo terminal.

Esses comandos são chamados de **imperativos**.

---

# 29. kubectl run

Criando um Pod:

```bash
kubectl run squirtle --image=nginx:alpine
```

Resultado:

```text
squirtle   1/1   Running
```

---

# 30. Labels em comandos imperativos

Foi criado:

```text
bulbasaur
```

com:

```text
type=pokemon
```

Comando:

```bash
kubectl run bulbasaur \
  --image=postgres:9.6-alpine \
  --labels=type=pokemon
```

Consulta:

```bash
kubectl describe pod bulbasaur
```

---

# 31. Criando Service via comando

Foi criado:

```text
bulbasaur-svc
```

do tipo ClusterIP:

```bash
kubectl create svc clusterip bulbasaur-svc --tcp=5432
```

Resultado:

```text
TYPE        PORT
ClusterIP   5432
```

---

# 32. Namespace fire

Foi criado:

```bash
kubectl create ns fire
```

Depois foi criado um Deployment diretamente pelo terminal:

```bash
kubectl -n fire create deploy charmander \
  --image=fbscarel/myapp-redis \
  --replicas=3
```

Resultado:

```text
charmander   3/3
```

---

# 33. NodePort por comando imperativo

Foi criado:

```bash
kubectl -n fire create svc nodeport charmander \
  --tcp=80 \
  --node-port=31080
```

Resultado:

```text
NodePort: 31080
TargetPort: 80
Endpoints: 3
```

A aplicação pôde ser acessada através da porta:

```text
31080
```

---

# 34. Criando Pod e Service em um único comando

Foi criado um Redis chamado `db`:

```bash
kubectl -n fire run db \
  --image=redis:alpine \
  --port=6379 \
  --expose
```

O comando criou:

```text
service/db created
pod/db created
```

Resultado:

```text
Pod     → db
Service → db
Port    → 6379
```

---

# 35. Comandos aprendidos

## Cluster

```bash
kubectl config current-context
kubectl cluster-info
kubectl get nodes
```

## Pods

```bash
kubectl get pods
kubectl get pods -o wide
kubectl describe pod NOME
kubectl delete pod NOME
kubectl edit pod NOME
```

## YAML

```bash
kubectl create -f arquivo.yaml
kubectl get pod NOME -o yaml
```

## ReplicaSets

```bash
kubectl get rs
kubectl describe rs NOME
kubectl edit rs NOME
kubectl scale rs --replicas=N NOME
```

## Deployments

```bash
kubectl get deploy
kubectl describe deploy NOME
kubectl edit deploy NOME
kubectl create deploy
```

## Services

```bash
kubectl get svc
kubectl describe svc NOME
kubectl create svc
kubectl expose
```

## Namespaces

```bash
kubectl get ns
kubectl create ns NOME
kubectl -n NAMESPACE get pods
```

## Execução dentro de Pods

```bash
kubectl exec
kubectl -n NAMESPACE exec
```

---

# 36. Principais conceitos aprendidos

## Pod

Menor unidade executável do Kubernetes.

```text
Pod
└── Container
```

---

## ReplicaSet

Mantém a quantidade desejada de Pods.

```text
ReplicaSet
├── Pod
├── Pod
└── Pod
```

---

## Deployment

Gerencia ReplicaSets e atualizações da aplicação.

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

---

## Service

Fornece acesso estável aos Pods.

```text
Cliente
   ↓
Service
   ↓
Pods
```

---

## Namespace

Separa logicamente recursos dentro do cluster.

```text
Cluster
├── dev
├── homolog
└── prod
```

---

# 37. Modelo mental final

Depois deste laboratório passei a visualizar Kubernetes desta forma:

```text
                    Kubernetes Cluster
                           │
                       Deployment
                           │
                       ReplicaSet
                           │
                ┌──────────┼──────────┐
                │          │          │
              Pod        Pod        Pod
                │          │          │
            Container  Container  Container
                │          │          │
                └──────────┼──────────┘
                           │
                        Service
                           │
                     ClusterIP
                          ou
                       NodePort
                           │
                        Cliente
```

---

# 38. O que aprendi nesta atividade

Os principais aprendizados foram:

1. Como o Kubernetes organiza aplicações em Pods.
2. Como investigar problemas utilizando `kubectl describe`.
3. Como identificar erros de imagem através de `ErrImagePull` e `ImagePullBackOff`.
4. Como ReplicaSets mantêm automaticamente a quantidade desejada de Pods.
5. Como escalar aplicações.
6. Como Deployments gerenciam ReplicaSets e atualizações.
7. Como Services fornecem acesso estável aos Pods.
8. A diferença entre `ClusterIP` e `NodePort`.
9. Como Namespaces separam recursos.
10. Como funciona o DNS interno do Kubernetes.
11. Como criar recursos através de YAML.
12. Como criar recursos através de comandos imperativos.
13. Como o Kubernetes trabalha constantemente para manter o estado desejado.

---

# 39. Conclusão

Esta atividade permitiu sair dos conceitos básicos de containers e começar a compreender como o Kubernetes realiza a orquestração de aplicações.

O ponto mais importante foi entender que não administramos containers individualmente.

Declaramos ao Kubernetes o **estado desejado**.

Por exemplo:

```text
"Quero 3 réplicas desta aplicação."
```

O Kubernetes passa então a trabalhar continuamente para manter esse estado.

Se um Pod falhar:

```text
Pod falha
   ↓
Kubernetes detecta
   ↓
ReplicaSet cria outro
```

Se a quantidade de réplicas mudar:

```text
replicas = 3
     ↓
replicas = 5
     ↓
Kubernetes cria mais 2 Pods
```

Essa lógica de **estado desejado + reconciliação automática** é um dos conceitos fundamentais para entender Kubernetes e orquestração de containers.
