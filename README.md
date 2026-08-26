# Kubernetes Labs

Repositório destinado a documentar minha evolução prática no estudo de **Docker, conteinerização e Kubernetes**.

O objetivo deste projeto não é apenas registrar comandos, mas compreender os conceitos, executar laboratórios, investigar erros e documentar as soluções encontradas durante o aprendizado.

A documentação também foi escrita para servir como material de apoio para outras pessoas que estão começando a estudar containers e Kubernetes.

---

## Trilha de estudos

### 01 — Conteinerização

- [x] Fundamentos de containers
- [x] Docker
- [x] Imagens e containers
- [x] Ciclo de vida de containers
- [x] Mapeamento de portas
- [x] Logs
- [x] Docker Compose
- [ ] Dockerfile
- [ ] Variáveis de ambiente
- [ ] Volumes e persistência
- [ ] Redes Docker
- [ ] Políticas de reinício

📖 Documentação:

[Atividade 01 — Introdução à Conteinerização](01-containerizacao/atividade-01.md)

---

### 02 — Kubernetes

- [x] Fundamentos do Kubernetes
- [x] Cluster e Nodes
- [x] kubectl
- [x] Pods
- [x] Pods com múltiplos containers
- [x] Diagnóstico com kubectl describe
- [x] ErrImagePull e ImagePullBackOff
- [x] ReplicaSets
- [x] Self-healing
- [x] Escalabilidade de réplicas
- [x] Deployments
- [x] Services
- [x] ClusterIP
- [x] NodePort
- [x] Namespaces
- [x] DNS e comunicação entre namespaces
- [x] Comandos imperativos
- [ ] ConfigMaps e Secrets
- [ ] Volumes
- [ ] Ingress

📖 Documentação:

[Atividade 02 — Conceitos Centrais do Kubernetes](02-kubernetes/atividade-02-conceitos-kubernetes.md)

---

## Conceitos já praticados

### Containers

```text
Imagem
   ↓
Container
   ↓
Processo da aplicação
```

### Kubernetes

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Containers
```

### Exposição da aplicação

```text
Cliente
   ↓
Service
   ↓
Pods
```

### Estado desejado

Um dos principais conceitos estudados até agora é que o Kubernetes trabalha para manter o **estado desejado** da aplicação.

Exemplo:

```text
replicas = 3
     ↓
um Pod falha
     ↓
ReplicaSet detecta
     ↓
novo Pod é criado
     ↓
voltam a existir 3 réplicas
```

---

## Laboratórios realizados

### Example Voting App

Aplicação executada utilizando Docker Compose com múltiplos containers.

Fluxo:

```text
Vote
 ↓
Redis
 ↓
Worker
 ↓
PostgreSQL
 ↓
Result
```

### myapp-color

Aplicação executada em Kubernetes utilizando:

```text
Deployment
   ↓
3 Pods
   ↓
Service NodePort
   ↓
localhost:30080
```

Durante o laboratório foram praticados exposição de aplicações, réplicas, Services e comunicação entre recursos.

---

## Estrutura do repositório

```text
kubernetes-labs/
│
├── 01-containerizacao/
│   └── atividade-01.md
│
├── 02-kubernetes/
│   └── atividade-02-conceitos-kubernetes.md
│
├── anotacoes/
│
├── projetos/
│
└── README.md
```

---

## Comandos principais estudados

### Docker

```bash
docker run
docker ps
docker ps -a
docker images
docker logs
docker stop
docker start
docker rm
docker compose up
docker compose ps
```

### Kubernetes

```bash
kubectl get
kubectl describe
kubectl create
kubectl run
kubectl delete
kubectl edit
kubectl scale
kubectl expose
kubectl exec
```

---

## Objetivo

Desenvolver conhecimento prático em:

- containers;
- Docker;
- Kubernetes;
- orquestração de aplicações;
- troubleshooting;
- escalabilidade;
- arquitetura de aplicações distribuídas.

A proposta é evoluir o repositório conforme novos laboratórios forem concluídos, mantendo tanto o registro da minha evolução quanto documentação útil para quem também está iniciando nessa área.
