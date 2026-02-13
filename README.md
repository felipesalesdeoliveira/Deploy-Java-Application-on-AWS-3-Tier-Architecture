# ☕ Deploy Java Application on AWS 3-Tier Architecture com Terraform

![AWS Architecture](https://imgur.com/b9iHwVc.png)

---

## 📑 Sumário

1. [Visão Geral do Projeto](#visão-geral-do-projeto)  
2. [Visão Geral da Arquitetura](#visão-geral-da-arquitetura)  
3. [Pré-Requisitos](#pré-requisitos)  
4. [Configuração da Infraestrutura com Terraform](#configuração-da-infraestrutura-com-terraform)  
   - [VPC e Rede](#vpc-e-rede)  
   - [Segurança](#segurança)  
   - [Camada de Banco de Dados](#camada-de-banco-de-dados)  
5. [Deploy da Aplicação](#deploy-da-aplicação)  
   - [Build da Aplicação](#build-da-aplicação)  
   - [Tomcat e Nginx](#tomcat-e-nginx)  
   - [Load Balancing e Auto Scaling](#load-balancing-e-auto-scaling)  
6. [Monitoramento e Manutenção](#monitoramento-e-manutenção)  
7. [Boas Práticas de Segurança](#boas-práticas-de-segurança)  
8. [Estrutura de Pastas](#estrutura-de-pastas)  
9. [Contribuição](#contribuição)  

---

# 📌 Visão Geral do Projeto

Este projeto demonstra o deploy de uma aplicação web Java em produção utilizando **AWS 3-Tier Architecture** provisionada via **Terraform**, seguindo boas práticas cloud-native, garantindo **alta disponibilidade, escalabilidade e segurança**.

### 🚀 Principais Características

- **Alta Disponibilidade**: Deploy Multi-AZ com failover automático  
- **Auto Scaling**: Escalabilidade dinâmica conforme demanda  
- **Segurança**: Estratégia Defense-in-Depth  
- **Monitoramento**: Logs e métricas centralizados via CloudWatch  
- **Provisionamento Automatizado**: Infraestrutura como Código com Terraform  

---

# 🏗️ Visão Geral da Arquitetura

## 🔹 Componentes

### 1️⃣ Camada de Apresentação (Frontend)
- Servidores Nginx em Auto Scaling Group  
- Network Load Balancer público  
- CloudFront para conteúdo estático  

### 2️⃣ Camada de Aplicação (Backend)
- Servidores Apache Tomcat em Auto Scaling Group  
- Network Load Balancer interno  
- Amazon ElastiCache para gerenciamento de sessões  

### 3️⃣ Camada de Dados
- Amazon RDS MySQL Multi-AZ  
- Backups automáticos e recuperação point-in-time  
- Read replicas para cargas de leitura  

---

## 🌐 Arquitetura de Rede

- Duas VPCs (`192.168.0.0/16` e `172.32.0.0/16`)  
- Subnets públicas e privadas em múltiplas AZs  
- Transit Gateway para comunicação privada entre VPCs  

---

# 🔧 Pré-Requisitos

- Terraform >= 1.0  
- AWS CLI configurado com permissões apropriadas  
- Conta AWS com IAM suficiente  

---

# 🏗️ Configuração da Infraestrutura com Terraform

## 1️⃣ Inicializar Terraform

```bash
terraform init
```

## 2️⃣ Validar e Planejar

```bash
terraform plan -var-file=variables.tfvars
```

## 3️⃣ Aplicar Infraestrutura

```bash
terraform apply -var-file=variables.tfvars --auto-approve
```

### Recursos Provisionados

- Duas VPCs com subnets públicas e privadas  
- Internet Gateway e NAT Gateway  
- Transit Gateway e associações entre VPCs  
- Security Groups e IAM Roles  
- Auto Scaling Group para frontend e backend  
- Network Load Balancers público e interno  
- Amazon RDS Multi-AZ com read replicas  
- Route 53 para DNS  
- CloudWatch Logs e métricas customizadas  

---

# 🔹 VPC e Rede

Todo o provisionamento é feito via Terraform utilizando módulos:

```hcl
module "vpc" {
  source = "./modules/vpc"
  cidr_block = "192.168.0.0/16"
}
```

- Criação de VPCs e subnets  
- Internet Gateway e NAT Gateway  
- Route Tables configuradas  
- VPC Flow Logs habilitados  

---

# 🔐 Segurança

- Security Groups configurados via Terraform  
- Menor privilégio nas IAM Roles  
- Bastion Host para acesso SSH controlado  
- AWS SSM Session Manager habilitado para acesso seguro  

---

# 🗄️ Camada de Banco de Dados

Provisionamento RDS via Terraform:

```hcl
module "rds" {
  source           = "./modules/rds"
  db_name          = "javaapp"
  username         = "admin"
  password         = var.rds_password
  multi_az         = true
  instance_type    = "db.t3.medium"
  subnet_ids       = module.vpc.private_subnets
}
```

- Multi-AZ  
- Backups automáticos  
- Read replicas para workloads de leitura  

---

# ☕ Deploy da Aplicação

## 🔨 Build da Aplicação

```bash
mvn clean package -DskipTests
mvn test
mvn deploy
```

## Tomcat e Nginx

- Configuração do serviço Tomcat via UserData no Terraform  
- Nginx configurado como reverse proxy para backend  

```hcl
user_data = file("scripts/userdata.sh")
```

## Load Balancing e Auto Scaling

- ASG provisionado via Terraform  
- NLB público associado ao frontend ASG  
- NLB interno associado ao backend ASG  

---

# 📊 Monitoramento e Manutenção

- CloudWatch Logs para Tomcat e métricas customizadas  
- Métricas de memória e CPU coletadas via script Terraform provisionado  

---

# 🔒 Boas Práticas de Segurança

- Network ACLs e Security Groups configurados corretamente  
- VPC Flow Logs habilitados  
- Criptografia em repouso para RDS  
- SSL/TLS para tráfego público  
- AWS Secrets Manager para segredos da aplicação  

---

# 📂 Estrutura de Pastas Recomendada

```
terraform-3tier-java/
├── modules/
│   ├── vpc/
│   ├── bastion/
│   ├── app-tier/
│   └── rds/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tfvars
│   │   └── backend.tf
│   └── prod/
├── scripts/
│   ├── userdata.sh
│   └── metrics.sh
├── README.md
└── .gitignore
```

- **modules/** – módulos reutilizáveis do Terraform (VPC, Bastion, App Tier, RDS)  
- **environments/** – configurações específicas por ambiente  
- **scripts/** – UserData, inicialização de instâncias e métricas  
- **README.md** – documentação do projeto  

---

# ⭐ Suporte ao Projeto

Se este projeto foi útil:

- Dê uma estrela ⭐ no repositório  
- Compartilhe com sua rede  
- Contribua com melhorias  

---

> ⚠️ Este projeto simula uma aplicação Java em produção com arquitetura 3-Tier na AWS, provisionada de forma automatizada usando Terraform, garantindo escalabilidade, alta disponibilidade e segurança.