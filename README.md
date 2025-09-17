# Projeto Serviços da AWS - Auto Scaling e Load Balancer

![AWS](https://img.shields.io/badge/AWS-SERVICES-green)
![AWS](https://img.shields.io/badge/AWS-ASG-orange)
![AWS](https://img.shields.io/badge/AWS-ALB-blue)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

Este documento resume os conceitos e a implementação prática de **Auto Scaling Group (ASG)**, **Application Load Balancer (ALB)** e as **regras de scaling**. Inclui o que é cada serviço, o aprendizado adquirido, passos práticos para configurar e testes básicos.

## ⚖️ O que é cada um?

### Load Balancer (Balanceador de Carga)
Serviço que distribui o tráfego de rede entre múltiplas instâncias ou contêineres para garantir alta disponibilidade e resiliência.

* **Tipos Comuns:**
    * **Application Load Balancer (ALB):** Atua na Camada 7 (HTTP/HTTPS), ideal para tráfego web, pois entende URLs, cabeçalhos e cookies.
    * **Network Load Balancer (NLB):** Atua na Camada 4 (TCP/UDP), oferecendo altíssima performance com baixa latência.
    * **Gateway Load Balancer (GWLB):** Focado no roteamento de tráfego para appliances de rede de terceiros (ex: firewalls).

### Auto Scaling Group (ASG)
Um conjunto de instâncias EC2 gerenciadas para escalar automaticamente conforme a demanda.

* **Principais Funções:**
    * **Escalabilidade automática:** Adiciona (`scale-out`) ou remove (`scale-in`) instâncias conforme as regras.
    * **Manutenção de Saúde:** Substitui automaticamente instâncias que falham nos health checks.
    * **Resiliência:** Distribui instâncias em múltiplas zonas de disponibilidade (AZs).

### Regras de Scaling (Escalonamento)
Definem quando e como o ASG deve ajustar o número de instâncias.

* **Scaling Up (Aumentar capacidade):** Acionado quando a demanda cresce.
    * *Exemplo:* `CPU > 70%` por 5 minutos.
* **Scaling Down (Reduzir capacidade):** Acionado quando a demanda diminui para otimizar custos.
    * *Exemplo:* `CPU < 20%` por 5 minutos.
* **Modos Principais:**
    * **Target Tracking:** Mantém uma métrica em um valor desejado (ex: `CPU média = 50%`).
    * **Step Scaling:** Ajusta a capacidade em "passos" com base em limites de alarme.
    * **Scheduled Scaling:** Altera a capacidade em horários pré-definidos (ex: picos de tráfego conhecidos).

---

## 🧠 Principais Aprendizados

* **Balanceadores de Carga** evitam a sobrecarga em uma única instância, distribuindo o trabalho.
* **Auto Scaling Groups** garantem resiliência: se uma instância falhar, outra nova é criada automaticamente.
* **Regras de Scaling** devem ser bem calibradas para evitar custos excessivos ou lentidão na aplicação.
* **Distribuição em múltiplas AZs** é fundamental para garantir tolerância a falhas em nível de data center.
* **Health Checks** são cruciais para assegurar que o tráfego seja enviado apenas para instâncias saudáveis.

---

## 📝 Passo a Passo Básico (Visão Geral)

### 1. Criar um Load Balancer
* **Definir o tipo:** Geralmente `ALB` para aplicações web.
* **Associar subnets:** Usar subnets públicas para que ele possa receber tráfego da internet.
* **Criar um Target Group:** Definir o grupo de instâncias que receberá o tráfego.
* **Configurar Health Checks:** Especificar uma rota (ex: `/`) e códigos de sucesso (ex: `200-399`) para validar a saúde das instâncias.

### 2. Criar um Auto Scaling Group
* **Criar um Launch Template:** Definir o "molde" da instância (AMI, tipo, user-data, etc.).
* **Configurar tamanhos:** Definir o número **mínimo**, **desejado** e **máximo** de instâncias.
* **Associar redes:** Vincular o ASG às subnets (geralmente privadas) e ao Target Group do Load Balancer.
* **Habilitar Health Checks do ELB:** Permitir que o ASG use a verificação de saúde do Load Balancer.

### 3. Configurar Regras de Scaling
* **Target Tracking:** Definir uma meta, como `CPU média = 50%`.
* **Step Scaling:** Criar regras como: se `CPU ≥ 70%`, adicionar `+1` instância; se `CPU ≤ 20%`, remover `-1` instância.
* **Scheduled:** Agendar o aumento da capacidade para horários de pico e a redução para a madrugada.

---

## ✅ Benefícios Principais

* **Escalabilidade Automática:** Ajusta os recursos dinamicamente conforme a demanda, sem intervenção manual.
* **Alta Disponibilidade:** Se uma instância ou zona de disponibilidade falhar, a aplicação continua funcionando.
* **Custo Otimizado:** Paga apenas pelos recursos que realmente precisa em um determinado momento.
* **Elasticidade:** Capacidade de crescer e encolher rapidamente para acompanhar variações de tráfego.

---

> ### Este projeto está licenciado sob a [Licença MIT](./LICENSE).