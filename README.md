# Hermes Control — Telemetria IoT e Inteligência Operacional

Sistema de monitoramento IoT e dashboard para **220 impressoras** de uma operação logística, com **economia estimada de R$ 60 mil a R$ 80 mil em 3 meses**.

[![GCP](https://img.shields.io/badge/Google_Cloud-(GCP)-05122A?style=flat-square&logo=googlecloud&logoColor=white)](https://cloud.google.com/)
[![Python](https://img.shields.io/badge/Python-3.10+-05122A?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-05122A?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Firestore](https://img.shields.io/badge/Firestore_(NoSQL)-05122A?style=flat-square&logo=firebase&logoColor=white)](https://firebase.google.com/products/firestore)

> **Nota:** este repositório não contém o código-fonte original, por causa do acordo de confidencialidade (NDA) com a Selbetti e o Mercado Livre. O objetivo aqui é documentar a arquitetura, as decisões de engenharia de dados e o impacto gerado.

Criado por iniciativa própria enquanto eu era técnico de impressoras na Selbetti, alocado numa operação logística do Mercado Livre (out/2025 a mai/2026).

## 📈 Impacto

O Hermes Control foi desenvolvido e colocado em produção num ambiente logístico crítico, monitorando **220 impressoras** Zebra e Ricoh. Nos 3 primeiros meses:

| Indicador | Antes | Com o Hermes |
|---|---|---|
| Tempo de impressora parada (somando a frota) | ~10 h por dia | ~1 h por dia |
| Ronda técnica | ~2 h | ~15 min |
| Economia estimada (3 meses somados) | — | **R$ 60 mil a R$ 80 mil** |

A economia vem de três frentes: menos tempo de máquina parada (o erro aparecia na hora e o técnico já ia resolver), rondas mais curtas (o estado de todas as impressoras ficava visível online) e troca de peças e insumos baseada em dados, sem trocas prematuras.

---

## 💻 Demonstração (vídeo sem áudio, 1 min)

https://github.com/user-attachments/assets/cb0c1380-0ebf-4a95-8b76-5ee203722be1

---

## 🛠️ Stack e decisões de arquitetura

Arquitetura serverless no Google Cloud Platform, para escalar sem custo fixo alto.

- **Linguagens:** Python 3.10+ (motor de coleta e ETL) e TypeScript (lógica da API nas Cloud Functions).
- **Protocolo:** SNMP v2c/v3, lendo OIDs específicos do hardware Zebra e Ricoh.
- **Banco:** **Google Cloud Firestore**. Escolhi NoSQL (documentos) porque os logs de erro das impressoras são semiestruturados e porque o volume pede escala horizontal. O histórico de falhas de cada ativo fica numa subcoleção, o que deixa as consultas de série temporal rápidas.

### 🔄 Pipeline de dados

```text
+-----------------+      (SNMP v2c/v3)      +------------------+
| Impressoras     | <---------------------- | Agente Python    |
| Zebra/Ricoh     |   Requisições (polling) | (Cloud Host)     |
| (220 ativos)    | ----------------------> | Motor PySNMP     |
+-----------------+   Dados brutos (OIDs)   +---------+--------+
                                                       |
                                                       | (Processamento / ETL)
                                                       v
+-------------------+      (JSON)             +------------------+
| GCP Firestore     | <---------------------- | Motor de regras  |
| Banco (NoSQL)     |                         | e sanitização    |
| (coleções JSON)   |                         | (Pandas)         |
+-------------------+                         +------------------+
        |
        | (consumo pela API)
        v
+------------------+      (Single Page App)  +------------------+
| GCP Cloud        | <---------------------- | Dashboard        |
| Functions        |   Requisições           | reativo (SPA)    |
| Node/TS          | ----------------------> | Visualização     |
+------------------+   Dados estruturados    | (cache SWR)      |
                                             +------------------+
```

---

## 🔥 Desafios de engenharia de dados

O ambiente logístico trouxe problemas de confiabilidade de rede e de desempenho, resolvidos no backend:

### 1. Debounce e sanitização dos dados
Impressoras numa operação logística têm instabilidades de rede momentâneas, que geram logs falsos. Implementei um **buffer de estado** no backend Python: o sistema só registra uma mudança real de status (por exemplo, Online → Erro) se a condição se mantiver por *n* ciclos de leitura.
- **Resultado:** sem isso, o banco ficaria poluído com centenas de logs falsos, e os relatórios de custo e disponibilidade perderiam o valor.

### 2. Limite do hardware e controle de concorrência
Impressoras térmicas (Zebra ZT411) têm pouco poder de processamento e podem travar se receberem muitas requisições SNMP ao mesmo tempo. Implementei controle de concorrência e agendamento das requisições no agente de coleta.
- **Resultado:** o próprio monitoramento nunca derruba (por excesso de requisições) os equipamentos que ele deveria proteger.

---

## 📊 Funcionalidades

O sistema transforma telemetria bruta em visualizações para a gestão da operação:

1. **Gêmeo digital (mapa de calor):** status de conexão e localização de todos os ativos em tempo real.
2. **Manutenção preditiva:** calcula o desgaste da cabeça de impressão pela quilometragem acumulada e antecipa a falha antes de a operação parar.
3. **Auditoria de custo:** cruza o volume real impresso com o custo estimado de insumos.
4. **Triagem inteligente:** diferencia falha de hardware de falta de suprimento, evitando deslocamentos técnicos desnecessários.

---

Feito por [Adriel Mota](https://www.linkedin.com/in/adriel-mota/).
