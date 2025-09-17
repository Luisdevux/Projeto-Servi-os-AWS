# Projeto Serviços da AWS - Auto Scaling e Load Balancer

![AWS](https://img.shields.io/badge/AWS-SERVICES-green)
![AWS](https://img.shields.io/badge/AWS-ASG-orange)
![AWS](https://img.shields.io/badge/AWS-ALB-blue)
![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)

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

## 🔗 Como Tudo se Conecta?
A combinação de **Load Balancer** + **Auto Scaling Group** cria um sistema autônomo e resiliente:

1. **Load Balancer** atua como porta de entrada, recebendo todo o tráfego dos usuários e distribuindo-o de forma inteligente.
2. **Auto Scaling Group** gerencia um conjunto de instâncias que executam a aplicação, garantindo que o número seja sempre ideal para a demanda.
3. **Target Groups** conectam o Load Balancer ao ASG, definindo para onde o tráfego deve ser enviado.
4. **Health Checks** mantêm apenas instâncias saudáveis no balanceamento.

```
Internet → ALB → Target Group → ASG → Instâncias EC2
```

---

## 🧠 Principais Aprendizados

* **Balanceadores de Carga** evitam a sobrecarga em uma única instância, distribuindo o trabalho.
* **Auto Scaling Groups** garantem resiliência: se uma instância falhar, outra nova é criada automaticamente.
* **Regras de Scaling** devem ser bem calibradas para evitar custos excessivos ou lentidão na aplicação.
* **Distribuição em múltiplas AZs** é fundamental para garantir tolerância a falhas em nível de data center.
* **Health Checks** são cruciais para assegurar que o tráfego seja enviado apenas para instâncias saudáveis.
* **Launch Templates** funcionam como "moldes" reutilizáveis para criar instâncias padronizadas.
* **Target Groups** permitem agrupar instâncias logicamente e aplicar health checks específicos.

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

### 4. Testar e Monitorar
* **Verificar Health Checks:** Validar se as instâncias estão passando nos testes de saúde.
* **Simular Carga:** Usar ferramentas como `Apache Bench` ou `curl` para testar o comportamento do scaling.
* **Monitorar Métricas:** Acompanhar CPU, memória e latência via CloudWatch.
* **Validar Custos:** Verificar se as regras de scale-in estão funcionando adequadamente.

---

## ✅ Benefícios Principais

* **Escalabilidade Automática:** Ajusta os recursos dinamicamente conforme a demanda, sem intervenção manual.
* **Alta Disponibilidade:** Se uma instância ou zona de disponibilidade falhar, a aplicação continua funcionando.
* **Custo Otimizado:** Paga apenas pelos recursos que realmente precisa em um determinado momento.
* **Elasticidade:** Capacidade de crescer e encolher rapidamente para acompanhar variações de tráfego.

---

## 💡 Dicas Práticas

### Configurações Recomendadas
* **Cooldown Period:** Definir um período de "esfriamento" de 5-10 minutos entre ações de scaling para evitar oscilações.
* **Health Check Grace Period:** Dar tempo suficiente (2-3 minutos) para que novas instâncias inicializem antes de serem avaliadas.
* **Múltiplas AZs:** Sempre distribuir instâncias em pelo menos 2-3 zonas de disponibilidade.
* **Tamanhos Conservadores:** Começar com limites baixos e ajustar conforme necessário.

### Cenários Comuns
* **E-commerce:** Escalar durante horários de pico (Black Friday, promoções).
* **Aplicações Corporativas:** Reduzir instâncias durante finais de semana e feriados.
* **APIs:** Usar Target Tracking baseado em latência ou número de requisições.
* **Aplicações Batch:** Usar Scheduled Scaling para processos que rodam em horários fixos.

### Troubleshooting
* **Instâncias não aparecem no Target Group:** Verificar Security Groups e NACLs.
* **Health Checks falhando:** Validar se a aplicação está respondendo na porta e caminho corretos.
* **Scaling muito lento:** Ajustar os thresholds e períodos de avaliação.
* **Custos altos:** Revisar as regras de scale-in e definir limites máximos apropriados.

---

> ### Este projeto está licenciado sob a [Creative Commons Attribution 4.0 International License](./LICENSE).