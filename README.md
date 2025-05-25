# CLOUDN2-UPDATE

# Projeto de Migração para a Nuvem AWS

## 📦 Parte 1 - Mapeamento e Escolha da Arquitetura

### ✅ Objetivo
Migrar uma aplicação web fullstack para a AWS, estruturando uma arquitetura funcional baseada em serviços gerenciados da nuvem, com foco em simplicidade, performance, custo reduzido e facilidade de manutenção.

---

## 🔍 Mapeamento dos Serviços AWS Necessários

| Necessidade                          | Serviço AWS                            | Descrição |
|-------------------------------------|----------------------------------------|----------|
| Hospedagem do frontend              | Amazon S3 + CloudFront                 | Hospedagem de arquivos estáticos (React/Vite) com CDN e HTTPS |
| Execução do backend                 | AWS Lambda (serverless)               | Execução de código backend (Node.js/Express) sem necessidade de servidor |
| Roteamento de requisições HTTP      | Amazon API Gateway                     | Criação de endpoints REST que disparam funções Lambda |
| Banco de dados relacional           | Amazon RDS (PostgreSQL)                | Banco relacional escalável e gerenciado, com backups automáticos |
| Segurança e controle de acesso      | IAM Roles e Policies                   | Gestão de permissões entre serviços |
| Armazenamento de segredos           | AWS Secrets Manager                    | Proteção e acesso seguro a strings de conexão e senhas |
| Monitoramento e logs                | Amazon CloudWatch                      | Coleta e análise de logs das funções Lambda e erros |

---

## ⚙️ Justificativa da Stack

| Critério         | Justificativa |
|------------------|---------------|
| **Simplicidade** | Utilização de arquitetura serverless reduz a complexidade da infraestrutura |
| **Custo (Free Tier)** | Lambda, API Gateway, S3 e RDS possuem cotas gratuitas ideais para testes e pequenos projetos |
| **Performance**  | CloudFront + S3 oferece entrega rápida do frontend; Lambda escala automaticamente |
| **Manutenção**   | Baixa sobrecarga operacional graças a serviços totalmente gerenciados |

---

## 🗺️ Diagrama da Arquitetura (Descrição)

Usuário
↓
CloudFront (HTTPS, CDN)
↓
S3 (Frontend estático: React/Vite)
↓
API Gateway (Rotas REST)
↓
AWS Lambda (Node.js/Express)
↓
RDS PostgreSQL (dentro de uma VPC privada)


### Serviços Complementares:
- **Secrets Manager**: Armazena credenciais de acesso ao banco de dados e outras chaves sensíveis.
- **IAM Roles**: Define quem pode acessar o quê (por exemplo, Lambda pode acessar Secrets e RDS).
- **CloudWatch**: Observabilidade e diagnósticos (logs e métricas de execução das funções Lambda).

---

## 📌 Próximos Passos

- Configuração prática da infraestrutura (rede, segurança, permissões, deploy)
- Comunicação segura entre os serviços
- Testes ponta a ponta e análise crítica do processo

---

> Este documento cobre apenas a **etapa 1** da migração para a nuvem AWS. As próximas partes documentarão a implementação e configuração detalhada da arquitetura.

# Conteúdo do README para a Parte 2 - Configuração da Infraestrutura
readme_part2 = """
# Projeto de Migração para a Nuvem AWS

## ⚙️ Parte 2 - Configuração da Infraestrutura

---

## 🔐 Rede e Segurança

### ✅ VPC (Virtual Private Cloud)
- **Subnets Públicas e Privadas** criadas:
  - Subnets públicas: usadas para serviços de acesso externo, como API Gateway.
  - Subnets privadas: usadas para serviços internos como AWS Lambda e Amazon RDS.
- **Routing Tables** configuradas para controlar o tráfego entre as subnets.

### ✅ Security Groups
- Criado SG para o RDS permitindo apenas acesso da Lambda na VPC.
- Criado SG para Lambda com permissão de saída para o RDS.
- Garantido que apenas o tráfego necessário seja permitido (principle of least privilege).

### ✅ IAM Roles e Policies
- **Lambda Execution Role**:
  - Permissões para acessar:
    - Secrets Manager
    - CloudWatch Logs
    - RDS (via VPC)
- **Policy Customizada**:
  - Criada para restringir o acesso apenas às actions e recursos necessários.
  
---

## ☁️ Serviços Específicos

### ✅ Amazon S3
- Criado bucket para hospedar o frontend estático (React/Vite).
- Configuração pública apenas para leitura dos arquivos estáticos.
- Habilitada opção de **website hosting** com index.html e error.html.

### ✅ CloudFront
- Criado distribuição CloudFront com origem apontando para o bucket S3.
- Adicionado **certificado SSL** (AWS Certificate Manager).
- Configurado cache e redirecionamento de HTTP para HTTPS.

### ✅ AWS Lambda
- Funções Lambda implementadas com Node.js (Express).
- Funções configuradas dentro da **VPC privada** para acesso ao RDS.
- Deploy automatizado via CLI ou Terraform.

### ✅ Amazon RDS (PostgreSQL)
- Instância RDS criada em subnet privada.
- Acesso restrito via SGs, apenas acessível por Lambda.
- Backup automático ativado, versão 15.x utilizada.

### ✅ AWS Secrets Manager
- Armazenadas:
  - String de conexão com o banco (host, user, password).
- Lambda acessa os segredos dinamicamente em tempo de execução.

---

## 📋 Observações Técnicas

- Todas as variáveis sensíveis foram armazenadas no **Secrets Manager** e acessadas por meio de **IAM roles seguras**.
- Infraestrutura provisionada com Terraform (em caso de automação) ou via AWS Console.
- CORS configurado nas rotas da API Gateway.

---

> Esta etapa representa a configuração segura e funcional da infraestrutura necessária para a aplicação. A próxima etapa irá focar na comunicação entre serviços e no teste ponta a ponta.
"""

## 🔁 Parte 3 - Comunicação entre Serviços

---

### 🔗 Fluxo de Comunicação Seguro

- **CloudFront ⇄ S3**
  - Distribuição configurada para acessar arquivos públicos no bucket S3.
  - Certificado SSL vinculado via AWS Certificate Manager para HTTPS seguro.

- **Frontend ⇄ API Gateway ⇄ Lambda**
  - O frontend realiza requisições HTTP para a API Gateway.
  - API Gateway encaminha as requisições para funções Lambda conforme as rotas configuradas.

- **Lambda ⇄ Amazon RDS**
  - Funções Lambda configuradas com acesso privado via VPC à instância RDS.
  - Conexões autenticadas com PostgreSQL usando credenciais obtidas via Secrets Manager.

### ⚙️ Configurações Complementares

- **Permissões IAM**
  - Lambda com role contendo políticas específicas para acessar Secrets Manager, RDS e CloudWatch.
- **CORS**
  - Configurado no API Gateway para permitir requisições do domínio do frontend.
- **Variáveis de Ambiente**
  - Configuradas para uso de endpoints, credenciais e outras dependências dinâmicas nas Lambdas.
- **VPC Links**
  - Utilizados para permitir comunicação privada com serviços dentro da VPC quando necessário.

---

## 🧠 Parte 4 - Análise Crítica da Migração

---

### 🆚 Comparação: PaaS vs AWS Nativa

| Critério                  | PaaS (ex: Vercel, Render)               | AWS Arquitetura Nativa                     |
|---------------------------|------------------------------------------|---------------------------------------------|
| **Controle**              | Abstração alta                          | Controle total sobre rede e segurança       |
| **Configuração**          | Simples e rápida                        | Complexa inicialmente, mais robusta         |
| **Escalabilidade**        | Limitada a tiers pagos                  | Altamente escalável via Lambda/API Gateway  |
| **Manutenção**            | Baixa                                   | Requer mais gerenciamento contínuo          |
| **Custo (free tier)**     | Gratuito até certo ponto                | Gratuito com uso consciente do free tier    |
| **Monitoramento**         | Limitado                                | Avançado com CloudWatch                     |

### 🚧 Dificuldades Encontradas

- Integração e permissões IAM entre Lambda, RDS e Secrets Manager.
- Configuração de segurança entre subnets privadas.
- Deploy de funções com VPC exigiu ajustes de roles e SGs.
- CORS entre frontend (CloudFront) e backend (API Gateway).

### 🔐 Reflexões Finais

- A migração trouxe ganhos em controle e segurança.
- Requer maior conhecimento técnico, mas oferece escalabilidade e modularidade superior.
- Ideal para projetos que exigem governança e integração com múltiplos serviços AWS.

---

## 📊 Parte 5 - Observabilidade (Opcional)

---

### 🔎 Monitoramento com CloudWatch

- Todas as funções Lambda estão integradas ao CloudWatch.
- Logs são capturados por execução e permitem:
  - Análise de erros.
  - Verificação de tempo de execução.
  - Monitoramento de uso de memória e chamadas.
- Possibilidade de criar alarmes e dashboards para visualizar tendências.

---

> Com estas etapas, consolidamos uma arquitetura escalável, segura e observável para a aplicação migrada à AWS.
"""
