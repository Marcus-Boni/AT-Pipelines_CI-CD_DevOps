# AT Pipelines CI/CD e DevOps - Ufology Investigation Unit

Projeto academico da AT de Pipelines CI/CD e DevOps com foco em:

- Dockerizacao de aplicacao Java (Spring Boot)
- Orquestracao com Kubernetes
- Banco PostgreSQL e cache Redis
- Automacao com GitHub Actions

## Objetivo da AT

Implementar um fluxo DevOps completo, da aplicacao ao deploy, cobrindo:

- Infraestrutura de dados no Kubernetes (PostgreSQL + Redis)
- Build e publicacao de imagem Docker da aplicacao
- Deploy da aplicacao em namespace dedicado
- Configuracao de CI/CD basica com GitHub Actions

## Repositorio e Imagem Publica

- Repositorio GitHub: https://github.com/Marcus-Boni/AT-Pipelines_CI-CD_DevOps
- Imagem da aplicacao no Docker Hub: https://hub.docker.com/r/marcusblz/ufotracker
- Tag utilizada no deploy: `marcusblz/ufotracker:1.0.0`

## Stack Tecnologica

- Java 21
- Spring Boot 3.5.4
- Maven Wrapper (`mvnw`)
- PostgreSQL
- Redis
- Docker
- Kubernetes (Docker Desktop)
- GitHub Actions

## Estrutura do Projeto

```text
.
├── .github/workflows/
│   ├── env-demo.yml
│   ├── gradle-ci.yml
│   ├── hello.yml
│   ├── secret-demo.yml
│   └── tests.yml
├── docker/
│   └── database/
│       ├── 01_schema.sql
│       └── Dockerfile
├── k8s/
│   └── ufology-stack.yaml
├── src/
├── Dockerfile
├── pom.xml
└── mvnw
```

## Arquitetura da Solucao

No namespace `ufology`, a solucao provisiona:

- `Deployment/Service` de PostgreSQL (`postgres`)
- `Deployment/Service` de Redis (`redis`)
- `ConfigMap` (`app-config`) para `DB_NAME`
- `Secret` (`db-secret`) para `DB_PASSWORD`
- `Deployment/Service` da aplicacao (`ufology-app`) com 2 replicas

Arquivo principal de infraestrutura:

- `k8s/ufology-stack.yaml`

## Endpoints Principais da API

- `GET /publico`
- `GET /avistamento`
- `GET /avistamento/paginado`
- `GET /avistamento/comFiltros`
- `GET /avistamento/{id}`
- `GET /avistamento/aleatorio`
- `GET /avistamento/buscar?localizacao=...`
- `GET /avistamento?confiabilidadeMin=...`

## Pre-requisitos

- Docker Desktop (com Kubernetes habilitado)
- Kubectl instalado
- Git
- Conta no Docker Hub

## Como Executar Localmente (sem Kubernetes)

1. Build da aplicacao:

```bash
./mvnw -B -DskipTests clean package
```

2. Rodar dependencias (PostgreSQL e Redis) localmente e exportar variaveis necessarias:

```bash
export DB_HOST=localhost
export REDIS_HOST=localhost
export REDIS_PORT=6379
export POSTGRES_USERNAME=postgres
export POSTGRES_PASSWORD=devops2025!
```

3. Rodar a aplicacao:

```bash
./mvnw spring-boot:run
```

## Docker

### Build da imagem

```bash
docker build -t marcusblz/ufotracker:1.0.0 .
```

### Push para Docker Hub

```bash
docker login -u marcusblz
docker push marcusblz/ufotracker:1.0.0
```

## Kubernetes

### 1) Habilitar cluster local

No Docker Desktop:

- Settings -> Kubernetes
- Enable Kubernetes
- Apply & Restart

### 2) Aplicar manifests

```bash
kubectl apply -f k8s/ufology-stack.yaml
```

### 3) Validar recursos

```bash
kubectl get all -n ufology
kubectl rollout status deployment/ufology-postgres -n ufology
kubectl rollout status deployment/ufology-redis -n ufology
kubectl rollout status deployment/ufology-app -n ufology
```

### 4) Testar API via port-forward

```bash
kubectl port-forward svc/ufology-app 8080:8080 -n ufology
curl http://localhost:8080/publico
```

## GitHub Actions (Parte 2 e Parte 3)

Workflows criados:

- `hello.yml`
  - Trigger: `push`
  - Log esperado: `Hello CI/CD`

- `tests.yml`
  - Trigger: `pull_request`
  - Passos: checkout + `echo "Rodando testes"`

- `gradle-ci.yml`
  - Trigger: `push` na `main`
  - Runner: `ubuntu-latest`
  - Configura Java 21
  - Sobe servicos `postgres` e `redis` no runner
  - Executa `./mvnw -B clean verify`

- `env-demo.yml`
  - Trigger: `push`
  - Variavel: `DEPLOY_ENV=staging`

- `secret-demo.yml`
  - Trigger: `push` e `workflow_dispatch`
  - Exige secret `API_KEY`
  - Log esperado: `API_KEY configurado`

## Secrets e Seguranca

### Secret no GitHub Actions

Criar secret no repositorio:

- Settings -> Secrets and variables -> Actions -> New repository secret
- Nome: `API_KEY`
- Valor: qualquer valor de teste

### Secret no Kubernetes

No manifesto, senha de banco esta em `Secret` (`db-secret`) para atender ao requisito da AT.

Observacao de boa pratica:

- Em ambiente real, evitar credenciais fixas em repositorio.
- Preferir Secret Manager externo (ex.: Vault, AWS Secrets Manager, Azure Key Vault).

## Problemas Comuns e Solucoes

### 1) `kubectl` tentando `localhost:8080`

Causa: cluster/contexto nao configurado.

Solucao:

```bash
kubectl config get-contexts
kubectl config use-context docker-desktop
kubectl cluster-info
```

### 2) `docker push ... insufficient_scope`

Causa: namespace da imagem diferente do usuario autenticado.

Solucao:

```bash
docker tag mgalv/ufotracker:1.0.0 marcusblz/ufotracker:1.0.0
docker push marcusblz/ufotracker:1.0.0
```

### 3) App em `CrashLoopBackOff` por Redis

Causa: conflito de `REDIS_PORT` injetado automaticamente pelo Kubernetes.

Solucao aplicada:

- Definido `REDIS_PORT=6379` explicitamente no deployment da app.

### 4) `secret-demo` falhando

Causa: `API_KEY` nao configurado no GitHub.

Solucao:

- Criar secret `API_KEY` no repositorio.

### 5) CI falhando com `release version 21 not supported`

Causa: Java 21 nao configurado no runner.

Solucao aplicada:

- Adicionado `actions/setup-java@v4` com `java-version: '21'`.

## Boas Praticas Aplicadas

- Namespace dedicado (`ufology`) para isolamento
- Services internos (`ClusterIP`) para banco/cache/app
- ConfigMap e Secret para configuracao sensivel
- Containerizacao com build multi-stage
- Pipelines separados por responsabilidade
- Healthcheck de servicos no CI para reduzir flakiness

## Autor

- Marcus Boni
- AT de Pipelines CI/CD e DevOps
