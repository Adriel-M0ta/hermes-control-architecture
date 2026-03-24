# Hermes Control - Telemetria IoT e Inteligência Operacional
Sistema de monitoramento IoT e Dashboard para +210 impressoras, gerando R$ 80k de economia


[![GCP](https://img.shields.io/badge/Google_Cloud_Platform-(GCP)-blue?style=flat&logo=google-cloud)](https://cloud.google.com/)
[![Python](https://img.shieldsio/badge/Language-Python_3.10+-blue?style=flat&logo=python)](https://www.python.org/)
[![TypeScript](https://img.shields.io/badge/Language-TypeScript-blue?style=flat&logo=typescript)](https://www.typescriptlang.org/)
[![Firestore](https://img.shields.io/badge/Database-Firestore_(NoSQL)-blue?style=flat&logo=firebase)](https://firebase.google.com/products/firestore)

> **Observação:** Este repositório não contém o código fonte original do projeto devido a acordos de confidencialidade (NDA) com a Selbetti e Mercado Livre. O objetivo aqui é documentar a arquitetura, as decisões de engenharia de dados e o impacto financeiro gerado pela solução.

## 📈 Impacto Financeiro Gerado

O Hermes Control foi desenvolvido e implementado em um ambiente logístico de altíssima criticidade, monitorando **+210 ativos operacionais**. Nos primeiros 3 meses de funcionamento, a solução gerou uma:

### **Economia Estimada entre R$ 60.000,00 e R$ 80.000,00**

Este valor foi alcançado através da redução drástica do *downtime* (tempo de máquina parada), otimização de rondas técnicas e prevenção de trocas de peças prematuras baseada em telemetria real.

---

## 💻 Showcase Técnico (Demonstração Mudo - 1min)

*Aqui você vai colar o GIF ou o link para o vídeo mudo apresentando as telas do sistema seguindo o roteiro que definimos.*

---

## 🛠️ Stack Técnica e Decisões de Arquitetura

O sistema foi desenhado sob uma arquitetura serverless na Google Cloud Platform (GCP) para garantir escalabilidade horizontal e custo-eficiência.

* **Linguagens:** Python 3.10+ (Motor de polling e ETL), TypeScript (Lógica de API nas Cloud Functions).
* **Protocolos:** SNMP v2c/v3 (Extração de OIDs específicos de hardware Zebra/Ricoh).
* **Banco de Dados:** **Google Cloud Firestore**. Escolhemos NoSQL (Document-based) devido à natureza semi-estruturada dos logs de erro e à necessidade de escalabilidade horizontal. O Firestore permite armazenar o histórico de falhas como subcoleções de cada ativo, facilitando consultas de séries temporais rápidas.

### 🔄 Pipeline de Dados (Fluxo de Arquitetura)

```text
+-----------------+      (SNMP v2c/v3)      +------------------+
| Dispositivos    | <---------------------- | Agente Python    |
| Zebra/Ricoh IoT |    Requests (Polling)   | (Host na Nuvem)  |
| (+210 ativos)   | ----------------------> | Motor PySNMP     |
+-----------------+    Dados Brutos (OIDs)  +---------+--------+
                                                       |
                                                       | (Tratamento / ETL)
                                                       v
+-----------------+      (Injeção JSON)     +------------------+
| GCP Firestore   | <---------------------- | Motor de Regras  |
| Database (NoSQL)|                         | e Sanitização    |
| (Coleções JSON) |                         | (Pandas)         |
+-----------------+                         +------------------+
        |
        | (Consumo via API)
        v
+------------------+      (Single Page App)  +------------------+
| GCP Cloud        | <---------------------- | Dashboard        |
| Functions        |   Requests (Serverless)| Reativo (SPA)    |
| Lógica Node/TS   | ----------------------> | Visualização     |
+------------------+     Dados Estruturados | (SWR Cache)      |
                                             +------------------+

---

## 🔥 Desafios de Engenharia de Dados Superados

O ambiente logístico impôs desafios críticos de confiabilidade e performance na rede, que foram mitigados através de lógica pura no backend:

### 1. Lógica de Debounce e Sanitização de Dados
Impressoras operando em ambiente logístico geram instabilidades momentâneas na rede (logs falsos). Implementamos um **buffer de estado** no backend Python. O sistema só registra uma mudança de status real (ex: Online -> Erro) se a condição persistir por um número 'n' de ciclos de polling.
* **Resultado:** Sem isso, o banco de dados estaria poluído com centenas de logs falsos, invalidando os relatórios de custo e disponibilidade.

### 2. Rate Limiting e Concurrency Control de Hardware
Impressoras térmicas (Zebra ZT411) possuem processadores limitados e podem travar se receberem muitas requisições SNMP simultâneas. Implementamos um controle de concorrência e escalonamento de requisições no agente de coleta Python.
* **Resultado:** Garantimos que o monitoramento não causasse, acidentalmente, um ataque de negação de serviço (DoS) nos próprios ativos que estávamos protegendo.

---

## 📊 Funcionalidades e Inteligência de Negócio

O sistema transforma telemetria bruta em visualizações acionáveis para a gestão Selbetti/Mercado Livre:

1.  **Gêmeo Digital (Heatmap):** Visualização geográfica e de status de conectividade em tempo real de todos os ativos operacionais.
2.  **Manutenção Preditiva:** Lógica que calcula o desgaste da cabeça térmica baseada na telemetria de quilometragem acumulada, antecipando falhas antes que elas parem a operação.
3.  **Auditoria de Custos:** Relatório consolidado que cruza o volume impresso real com o custo estimado de suprimentos, permitindo uma gestão financeira precisa.
