# Deploy de Aplicação Java na AWS com Arquitetura 3 Camadas (3-Tier)

![AWS Architecture](https://imgur.com/b9iHwVc.png)

---

## 📑 Sumário

1. [Visão Geral do Projeto](#visão-geral-do-projeto)
2. [Visão Geral da Arquitetura](#visão-geral-da-arquitetura)
3. [Pré-Requisitos](#pré-requisitos)
4. [Configuração da Infraestrutura](#configuração-da-infraestrutura)
   - [VPC e Rede](#vpc-e-rede)
   - [Configuração de Segurança](#configuração-de-segurança)
   - [Camada de Banco de Dados](#camada-de-banco-de-dados)
5. [Configuração da Aplicação](#configuração-da-aplicação)
   - [Ambiente de Build](#ambiente-de-build)
   - [Deploy da Aplicação](#deploy-da-aplicação)
   - [Load Balancing e Auto Scaling](#load-balancing-e-auto-scaling)
6. [Monitoramento e Manutenção](#monitoramento-e-manutenção)
7. [Boas Práticas de Segurança](#boas-práticas-de-segurança)
8. [Guia de Troubleshooting](#guia-de-troubleshooting)
9. [Contribuição](#contribuição)

---

![3-tier Architecture Diagram](https://imgur.com/3XF0tlJ.png)

---

# 📌 Visão Geral do Projeto

## 📖 Introdução

Este projeto demonstra o deploy de uma aplicação web Java em nível de produção utilizando a arquitetura 3-Tier da AWS. A implementação segue boas práticas cloud-native, garantindo alta disponibilidade, escalabilidade e segurança em todas as camadas da aplicação.

### 🚀 Principais Características

- **Alta Disponibilidade**: Deploy Multi-AZ com failover automático  
- **Auto Scaling**: Escalabilidade dinâmica conforme a demanda  
- **Segurança**: Estratégia Defense-in-Depth  
- **Monitoramento**: Logs e métricas centralizados  
- **Otimização de Custos**: Uso eficiente de recursos  

---

# 🏗️ Visão Geral da Arquitetura

## 🔹 Componentes da Infraestrutura

### 1️⃣ Camada de Apresentação (Frontend)
- Servidores Nginx em Auto Scaling Group  
- Network Load Balancer público  
- CloudFront para conteúdo estático  

### 2️⃣ Camada de Aplicação (Backend)
- Servidores Apache Tomcat em Auto Scaling Group  
- Network Load Balancer interno  
- Amazon ElastiCache para gerenciamento de sessões  

### 3️⃣ Camada de Dados
- Amazon RDS MySQL em configuração Multi-AZ  
- Backups automáticos e recuperação point-in-time  
- Read replicas para workloads com alta leitura  

---

## 🌐 Arquitetura de Rede

- Duas VPCs separadas (`192.168.0.0/16` e `172.32.0.0/16`)  
- Subnets públicas e privadas distribuídas em múltiplas AZs  
- Transit Gateway para comunicação entre VPCs  

---

# 🔧 Pré-Requisitos

## 🧾 Conta AWS

- Criar conta AWS Free Tier  
- Instalar AWS CLI v2  

```bash
# Linux
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# macOS
brew install awscli

aws configure
```

---

## 🔄 Git

```bash
# Linux
sudo apt-get update
sudo apt-get install git

# macOS
brew install git
```

---

## 🔁 Integração CI/CD

### SonarCloud

Adicionar no `pom.xml`:

```xml
<properties>
    <sonar.projectKey>seu_project_key</sonar.projectKey>
    <sonar.organization>sua_organizacao</sonar.organization>
    <sonar.host.url>https://sonarcloud.io</sonar.host.url>
</properties>
```

### JFrog Artifactory

Configurar no `settings.xml`:

```xml
<servers>
    <server>
        <id>jfrog-artifactory</id>
        <username>${env.JFROG_USERNAME}</username>
        <password>${env.JFROG_PASSWORD}</password>
    </server>
</servers>
```

---

# 🏗️ Configuração da Infraestrutura

## 🌐 VPC e Rede

### Criar VPC

```bash
aws ec2 create-vpc \
    --cidr-block 192.168.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=PrimaryVPC}]' \
    --region us-east-1
```

### Criar Subnets

```bash
aws ec2 create-subnet \
    --vpc-id vpc-xxx \
    --cidr-block 192.168.1.0/24 \
    --availability-zone us-east-1a
```

### Internet Gateway

```bash
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-xxx --internet-gateway-id igw-xxx
```

---

## 🔐 Configuração de Segurança

### Criar Security Group

```bash
aws ec2 create-security-group \
    --group-name FrontendSG \
    --description "Security group for frontend servers" \
    --vpc-id vpc-xxx
```

Liberar HTTP/HTTPS:

```bash
aws ec2 authorize-security-group-ingress \
    --group-id sg-xxx \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

---

# 🗄️ Camada de Banco de Dados

## Criar RDS

```bash
aws rds create-db-instance \
    --db-instance-identifier prod-mysql \
    --db-instance-class db.t3.medium \
    --engine mysql \
    --master-username admin \
    --master-user-password "SuaSenhaSegura"
```

## Inicialização do Banco

```sql
CREATE DATABASE javaapp;
USE javaapp;

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_username ON users(username);
CREATE INDEX idx_email ON users(email);
```

---

# ☕ Configuração da Aplicação

## ⚙️ pom.xml

```xml
<properties>
    <java.version>11</java.version>
    <spring.version>2.5.12</spring.version>
</properties>
```

---

## 🔨 Build

```bash
mvn clean package -DskipTests
mvn test
mvn deploy
```

---

# 🚀 Deploy da Aplicação

## Serviço do Tomcat

```bash
sudo nano /etc/systemd/system/tomcat.service
```

## Configuração Nginx

```nginx
server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

---

# ⚖️ Load Balancing e Auto Scaling

```bash
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name WebServerASG \
    --min-size 2 \
    --max-size 6 \
    --desired-capacity 2
```

---

# 📊 Monitoramento e Manutenção

## Monitorar Recursos

```bash
top
free -m
df -h
```

---

# 🔒 Boas Práticas de Segurança

## Segurança de Rede
- Implementar Network ACLs  
- Configurar corretamente Security Groups  
- Habilitar VPC Flow Logs  
- Configurar AWS WAF  

## Segurança da Aplicação
- Aplicar patches regularmente  
- Utilizar AWS Shield  
- Utilizar AWS Secrets Manager  
- Habilitar AWS GuardDuty  

## Segurança de Dados
- Habilitar criptografia em repouso  
- Utilizar SSL/TLS em trânsito  
- Realizar auditorias periódicas  
- Implementar estratégia de backup  

---

# 🛠️ Guia de Troubleshooting

## Problemas de Conexão

```bash
telnet database-endpoint 3306
aws ec2 describe-security-groups --group-ids sg-xxx
```

## Problemas de Performance

```bash
top
free -m
df -h
ps -eLf | grep java | wc -l
```

---

# 🤝 Contribuição

1. Fork do repositório  
2. Criar branch de feature  
3. Commit das alterações  
4. Push para a branch  
5. Abrir Pull Request  

---

# ⭐ Suporte ao Projeto

Se este projeto foi útil para você:

- Dê uma ⭐ no repositório  
- Compartilhe com sua rede  
- Contribua com melhorias  

---

> ⚠️ Esta documentação está em constante evolução. Verifique o repositório regularmente para atualizações.
