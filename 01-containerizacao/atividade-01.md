# Atividade 01 — Introdução à Conteinerização

## Objetivo

Realizar os primeiros laboratórios com Docker, praticando imagens, containers, portas, logs, ciclo de vida e Docker Compose.

## Ambiente utilizado

- Windows
- Docker Desktop
- Docker Engine 29.4.1
- Docker Compose 5.1.3

O laboratório foi executado utilizando Docker Desktop no Windows.

## Comandos aprendidos

### Verificar o Docker

```bash
docker --version
docker version
```

O comando `docker version` permite verificar o Docker Client e o Docker Server (Engine).

### Primeiro container

```bash
docker run hello-world
```

Fluxo observado:

1. Docker procura a imagem localmente.
2. Caso não exista, realiza o download da imagem.
3. Cria um container a partir da imagem.
4. Executa o processo do container.

### Listar containers

```bash
docker ps
docker ps -a
```

- `docker ps` → mostra containers em execução.
- `docker ps -a` → mostra todos os containers, inclusive os parados.

### Listar imagens

```bash
docker images
```

As imagens são utilizadas como base para criação dos containers.

## Laboratório com Nginx

Foi criado um container Nginx:

```bash
docker run -d --name lab-nginx -p 8080:80 nginx:alpine
```

Parâmetros utilizados:

- `-d` → executa o container em background.
- `--name` → define o nome do container.
- `-p 8080:80` → mapeia a porta do host para a porta do container.
- `nginx:alpine` → imagem utilizada.

Mapeamento:

```text
localhost:8080 → container:80 → Nginx
```

### Logs

```bash
docker logs lab-nginx
```

Nos logs foi possível visualizar a inicialização do Nginx e as requisições HTTP realizadas pelo navegador.

### Ciclo de vida

```bash
docker stop lab-nginx
docker start lab-nginx
docker rm -f lab-nginx
```

- `stop` → para o container.
- `start` → inicia novamente um container existente.
- `rm` → remove o container.

## Docker Compose — Example Voting App

Foi utilizada a aplicação de exemplo `example-voting-app`.

```bash
git clone https://github.com/dockersamples/example-voting-app.git
cd example-voting-app
docker compose up -d
docker compose ps
```

Foram executados os seguintes serviços:

- `vote` → interface de votação.
- `redis` → armazenamento temporário.
- `worker` → processamento dos votos.
- `db` → banco PostgreSQL.
- `result` → interface de resultados.

Fluxo da aplicação:

```text
Usuário
   ↓
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

No ambiente local:

```text
Votação:    http://localhost:8080
Resultados: http://localhost:8081
```

## Resultado

A aplicação foi executada com sucesso utilizando Docker Compose.

Foi realizado um voto e validada sua exibição na página de resultados, concluindo a primeira atividade prática do curso.

## Principais aprendizados

- Imagem x Container
- Docker Client x Docker Engine
- `docker run`
- `docker ps`
- `docker images`
- `docker logs`
- ciclo de vida de containers
- mapeamento de portas `HOST:CONTAINER`
- execução de múltiplos serviços com Docker Compose
