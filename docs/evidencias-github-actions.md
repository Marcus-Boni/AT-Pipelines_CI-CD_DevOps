# Evidencias - GitHub Actions

Use este arquivo como base para montar o PDF final com prints e links.

## Workflows Criados

- hello: `.github/workflows/hello.yml`
- tests: `.github/workflows/tests.yml`
- gradle-ci: `.github/workflows/gradle-ci.yml`
- env-demo: `.github/workflows/env-demo.yml`
- secret-demo: `.github/workflows/secret-demo.yml`

## Evidencias para o PDF

### hello.yml

- Trigger: push
- Log esperado: `Hello CI/CD`
- Print da execucao: [inserir no PDF]
- Link da execucao: [colar URL do GitHub Actions]

### tests.yml

- Trigger: pull_request
- Etapas principais: checkout + `Rodando testes`
- Print da execucao: [inserir no PDF]
- Link da execucao: [colar URL do GitHub Actions]

### gradle-ci.yml

- Trigger: push na branch `main`
- Runner: `ubuntu-latest`
- Build: `./mvnw -B clean verify` (simulando o `./gradlew build`)
- Prints da execucao: [inserir no PDF]
- Link da execucao: [colar URL do GitHub Actions]

### env-demo.yml

- Trigger: push
- Variavel usada: `DEPLOY_ENV=staging`
- Log esperado: `DEPLOY_ENV=staging`
- Print da execucao: [inserir no PDF]
- Link da execucao: [colar URL do GitHub Actions]

### secret-demo.yml

- Trigger: push ou workflow_dispatch
- Secret usado: `API_KEY`
- Log esperado: `API_KEY configurado`
- Observacao: o valor do secret nao e exibido no log
- Print da execucao: [inserir no PDF]
- Link da execucao: [colar URL do GitHub Actions]

## Runners GitHub Hosted x Self-Hosted

### GitHub Hosted Runners

Vantagens:

- Setup imediato, sem manter servidor
- Atualizacao e manutencao feitas pelo GitHub
- Escalabilidade simples para pipelines comuns

Desvantagens:

- Menor controle do ambiente
- Limites de uso dependendo do plano
- Menos apropriado para acesso a rede interna privada

### Self-Hosted Runners

Vantagens:

- Controle total de hardware, rede e dependencias
- Pode acessar recursos internos (VPN, banco interno, etc.)
- Possivel otimizar custo em cargas constantes

Desvantagens:

- Responsabilidade de manutencao e seguranca e sua
- Requer monitoramento, patching e capacidade operacional
- Escalabilidade e alta disponibilidade precisam ser planejadas
