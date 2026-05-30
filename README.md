# Loja Veloz: Plataforma de Pedidos em Microsserviços

Este repositório contém a implementação de uma plataforma de pedidos em microsserviços, demonstrando a migração de um ambiente local com Docker Compose para um ambiente de produção com Kubernetes, com foco em observabilidade e CI/CD.

##  Visão Geral da Arquitetura

A arquitetura é composta pelos seguintes microsserviços:

- **API Gateway:** Ponto de entrada para as requisições externas.
- **Serviço de Pedidos:** Gerencia a criação e consulta de pedidos.
- **Serviço de Pagamentos:** Integração com sistemas de pagamento externos.
- **Serviço de Estoque:** Gerencia a reserva e baixa de itens.
- **PostgreSQL:** Banco de dados para persistência de dados.

##  Ambiente Local com Docker Compose

Para desenvolver e testar localmente, utilizamos o Docker Compose. Todos os serviços podem ser iniciados com um único comando.

### Pré-requisitos

- **Docker Desktop** instalado no Windows (com suporte a WSL2 recomendado).
- **PowerShell** ou **Windows Terminal**.

### Como Executar no Windows

1. Abra o **PowerShell** e navegue até a pasta do projeto:
   ```powershell
   cd C:\caminho\para\loja-veloz
   ```
2. Inicie os serviços usando o Docker Compose:
   ```powershell
   docker-compose up --build
   ```
3. Para encerrar os serviços:
   ```powershell
   docker-compose down
   ```

*Nota: Se você estiver usando o WSL2 (Windows Subsystem for Linux), os comandos `bash` anteriores funcionarão normalmente dentro do seu terminal Linux (Ubuntu/Debian).*

##  Conteinerização e Versionamento

Cada microsserviço possui seu próprio `Dockerfile` bem estruturado, utilizando multi-stage builds para otimizar o tamanho das imagens e garantir a segurança (execução como usuário não-root).

Exemplo de `Dockerfile` (serviço de pedidos):

```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder
WORKDIR /app
COPY . .
RUN go build -o main .

# Final stage
FROM alpine:3.18
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
WORKDIR /app
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]
```

As imagens são versionadas com tags (`v1.0.0`, `v1.0.1`, etc.) e o hash do commit (`${{ github.sha }}`) para rastreabilidade.

##  Kubernetes – Produção Mínima

Para o ambiente de produção, a aplicação é orquestrada pelo Kubernetes. Os manifestos estão localizados no diretório `k8s/`.

- **Deployments:** Gerenciam as réplicas dos microsserviços.
- **Services:** Expondo os microsserviços internamente no cluster.
- **ConfigMaps:** Armazenam configurações não sensíveis (ex: `DB_HOST`).
- **Secrets:** Armazenam informações sensíveis (ex: `DB_USER`, `DB_PASS`) de forma segura.
- **HPA (Horizontal Pod Autoscaler):** Configurado para escalar o serviço de pedidos com base na utilização da CPU.

### Como Implantar no Kubernetes

1. Certifique-se de ter um cluster Kubernetes configurado e o `kubectl` apontando para ele.
2. Aplique os manifestos:
   ```bash
   kubectl apply -f k8s/
   ```

##  CI/CD com GitHub Actions

Um pipeline de CI/CD foi implementado usando GitHub Actions para automatizar o processo de build, teste e deploy. O arquivo de configuração está em `cicd/pipeline.yaml`.

O pipeline inclui as seguintes etapas:

1. **Build:** Compilação do código.
2. **Testes:** Execução de testes unitários.
3. **Scan de Segurança:** Análise de vulnerabilidades na imagem Docker com Trivy.
4. **Publicação:** Push da imagem Docker para um registry.
5. **Deploy:** Atualização do Deployment no Kubernetes com a nova imagem.

## Observabilidade, Deploy e Escala

### Observabilidade

A estratégia de observabilidade proposta inclui:

- **Métricas:** Coleta de métricas com Prometheus e visualização com Grafana.
- **Logs:** Centralização de logs para análise.
- **Traces:** Rastreamento distribuído com OpenTelemetry para monitorar o fluxo de requisições entre os microsserviços.

### Estratégia de Deploy

Adotamos a estratégia de **Rolling Update**, que é o padrão do Kubernetes. Novas versões dos Pods são gradualmente substituídas pelas antigas, garantindo zero downtime durante as atualizações.

### Escalabilidade

O **Horizontal Pod Autoscaler (HPA)** é utilizado para escalar automaticamente o número de réplicas do serviço de pedidos com base na utilização da CPU, garantindo que a aplicação possa lidar com picos de tráfego.

##  Infraestrutura como Código (IaC)

Um esqueleto de Terraform (`terraform/main.tf`) é fornecido para demonstrar como a infraestrutura (ex: cluster EKS na AWS) pode ser provisionada e gerenciada de forma automatizada e versionada.

##  Vídeo Pitch

**https://youtu.be/2qML6M8xk2w?si=oaycKlBu0rrUQy2g**

---
