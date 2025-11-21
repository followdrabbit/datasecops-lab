# 🛡️ DataSecOps Lab — Arquitetura de Pipeline de Dados Seguro (Z0–Z5)

> Repositório para definir e evoluir uma **arquitetura de DataSecOps** para
> pipelines de dados, da **fonte (Z0)** até a **camada de segurança e
> observabilidade (Z4 e Z5)**, com foco em segurança, governança, qualidade e
> auditoria de dados.

Este projeto descreve o **modelo conceitual** (zonas, controles, riscos,
políticas) que servirá de base para, no futuro, criar **labs práticos** em um
diretório `hands-on/` que implementarão partes dessa arquitetura.

---

## 0) Visão Geral

### 🎯 Objetivo

Criar uma referência de **pipeline de dados seguro** (DataSecOps), cobrindo:

- **Z0 — Data Sources**  
  Fontes internas, externas e parceiras tratadas como não confiáveis por padrão.
- **Z1 — Ingestion & Security Gateway**  
  Camada de borda para ingestão segura (APIs, arquivos, eventos, batch pull).
- **Z2 — Raw Data Lake (Restricted)**  
  Armazenamento bruto, controlado, cifrado e rastreável.
- **Z3 — Curated Data & Governance**  
  Dados tratados, governança, qualidade, PII e data products.
- **Z4 — Security & Trust Services**  
  Identidade, acesso, segredos, chaves, políticas como código, DLP, SIEM, catálogo e evidências.
- **Z5 — Monitoring, Observability & Audit**  
  Logs estruturados, métricas/SLOs, tracing, monitoração de ingestão/dados e trilhas de auditoria.

A arquitetura é **agnóstica de tecnologia**: pode ser mapeada para qualquer
cloud ou ambiente on-prem (armazenamento de objetos, funções serverless,
gateways de API, orquestradores, data lakes, SIEM, etc.).

### 📌 Escopo

**Inclui:**

- Arquitetura conceitual em zonas (Z0–Z5) sob a ótica de DataSecOps.  
- Definição de riscos, ameaças e controles para cada zona.  
- Política de classificação de dados e PII.  
- Threat model focado em pipeline de dados.  
- Modelo de catálogo de datasets e data cards.  
- Camadas transversais de **segurança (Z4)** e **observabilidade/auditoria (Z5)** que sustentam Z0–Z3.

**Não inclui (por enquanto):**

- Implementação de labs, código de ingestão/transformação ou IaC.  
- Modelos de IA/ML, LLMs, aplicações de negócio.  

> Labs práticos poderão ser adicionados futuramente no diretório `hands-on/`,
> como exemplos que implementam partes desta arquitetura.

---

## 📁 Estrutura de Diretórios

Estrutura principal voltada à arquitetura e documentação:

```text
.
│   LICENSE
│   README.md                            # Visão geral do projeto
│
├───assets                               # Diagramas, imagens e artefatos visuais
│
└───docs                                 # Arquitetura conceitual e guias de DataSecOps
    │   README.md                        # Visão geral da arquitetura de dados (Z0–Z3)
    │   DATA-CLASSIFICATION.md
    │   THREAT-MODEL-DATASECOPS.md
    │
    ├───catalog                                       
    │       datasets.yaml                # Catálogo técnico de datasets (internos/externos)
    │       DATASET_CARD-*.md            # Fichas individuais de datasets (data cards)
    │
    ├───Z0                               # Documentos da Z0
    ├───Z1                               # Documentos da Z1
    ├───Z2                               # Documentos da Z2
    ├───Z3                               # Documentos da Z3
    ├───Z4                               # Documentos da Z4
    └───Z5                               # Documentos da Z5
```

### Pastas principais

* `assets/`
  Diagramas conceituais da arquitetura por zona (Z0–Z3 por enquanto) e visão geral
  do pipeline. Futuramente podem ser incluídos diagramas para Z4 e Z5.

* `docs/`
  Núcleo da arquitetura:

  * `DATA-CLASSIFICATION.md`
    Política de classificação de dados e PII (níveis, exemplos, implicações).
  * `THREAT-MODEL-DATASECOPS.md`
    Threat model do pipeline de dados, com atores, ameaças e controles.
  * `docs/catalog/`

    * `datasets.yaml` → modelo de catálogo de datasets (nome, origem, licença, PII etc.).
    * `DATASET_CARD.md` → template de ficha por dataset (data card).
  * `docs/Z0`–`docs/Z3`
    Arquitetura das zonas de dados.
  * `docs/Z4`
    Serviços transversais de segurança e confiança (IAM, segredos, KMS, policy-as-code,
    DLP, SIEM, catálogo, compliance). 
  * `docs/Z5`
    Monitoramento, observabilidade e auditoria (logs, métricas, tracing, monitoração
    de ingestão/dados, alertas, runbooks). 

> No futuro, um diretório `hands-on/` poderá ser adicionado para conter labs
> práticos que implementem partes desta arquitetura.

---

## 1) Arquitetura em Zonas (Z0–Z5)

A arquitetura é organizada em **zonas**, cada uma com objetivos, riscos e
controles específicos. Z0–Z3 são o pipeline de dados; Z4 e Z5 são **camadas
transversais** que reforçam segurança, governança e observabilidade.

> Exemplos concretos (AWS, Azure, GCP, on-prem) podem ser derivados desta
> arquitetura, mas não fazem parte deste README.

---

### 🧱 Z0 — Data Sources (External, Partner & Internal Producers)

Conjunto de fontes de dados que alimentam o pipeline:

* **Externas**: open data, datasets públicos (Kaggle, UCI etc.), APIs públicas, bureaus.
* **Parceiros**: integrações B2B, trocas de arquivos, APIs dedicadas.
* **Internas**: sistemas core, canais (app/web/atm), logs, eventos de aplicações e filas internas.

**DataSecOps em Z0**

* Fontes não são confiáveis por padrão.
* Riscos: data poisoning, arquivos maliciosos, dump de dados sensíveis fora do fluxo.
* Controles:

  * contratos de dados (schema, tipos, enums, ranges),
  * registro de origem/dono/finalidade no catálogo,
  * avaliação de licença/compliance para datasets externos,
  * classificação inicial de dados (`DATA-CLASSIFICATION.md`).

📄 Detalhes: `docs/Z0/Z0-index.md`

---

### 🚪 Z1 — Ingestion & Security Gateway

Camada de borda que **recebe e filtra** os dados antes de entrarem no pipeline:

* APIs e endpoints de ingestão.
* Gateways/proxies.
* Pontos de entrada para arquivos, eventos, mensagens e streams.

**Modos de ingestão**

* Eventos/streaming.
* Arquivos (CSV/JSON/Parquet etc.).
* Batch pull de datasets externos.

**DataSecOps em Z1**

* **Zero trust na borda**:

  * autenticar e autorizar quem envia,
  * validar conteúdo,
  * limitar volume/frequência.

* Controles:

  * autenticação/autorização centralizada,
  * rate limiting / throttling,
  * validação de MIME, tamanho, formato e schema básico,
  * inspeção/sanitização de arquivos,
  * logging detalhado (identidade, origem, payload resumido).

📄 Detalhes:

* visão geral: `docs/Z1/Z1-Index.md`
* subitens: `docs/Z1/Z1-2.x.md`

---

### 🧊 Z2 — Raw Data Lake (Restricted)

Zona de armazenamento **bruto**, que mantém os dados exatamente como vieram:

* usado para auditoria, reprocessamento e rastreabilidade.
* organizado por fonte, tipo e partição de tempo.

**DataSecOps em Z2**

* Zona **restrita**: acesso mínimo e controlado.
* Controles:

  * criptografia em repouso e em trânsito,
  * controle de acesso com mínimo privilégio (RBAC/ACLs),
  * logs de acesso e alterações,
  * versionamento/retention para dados brutos,
  * separação de `raw/internal`, `raw/external`, `raw/partner`.

📄 Detalhes:

* visão geral: `docs/Z2/Z2-Index.md`
* subitens: `docs/Z2/Z2-2.x.md`

---

### 🔍 Z3 — Curated Data & Governance

Camada de dados **tratados**, governados e prontos para consumo por times e
sistemas.

**DataSecOps em Z3**

* **Data Quality como código**:

  * testes automáticos de integridade, regras de negócio e consistência.
* **Catálogo & Lineage**:

  * vínculo com `docs/catalog/`,
  * rastreio de origem (Z0/Z2) até Z3.
* **Classificação & PII**:

  * aplicação de `DATA-CLASSIFICATION.md`,
  * anonimização/pseudonimização quando necessário,
  * acesso por papel + classificação + finalidade.
* **Data Products**:

  * owner definido, contrato de schema, SLO/SLA de atualização.

📄 Detalhes:

* visão geral: `docs/Z3/Z3-index.md`
* subitens: `docs/Z3/Z3-2.x.md`

---

### 🔐 Z4 — Security & Trust Services (Transversal)

A Z4 reúne os **serviços de segurança compartilhados** que sustentam todas as
zonas (Z0–Z3):

* identidade e acesso,
* segredos,
* chaves criptográficas e PKI,
* políticas como código,
* proteção de dados (DLP/masking/tokenização),
* SIEM/UEBA/SOAR,
* catálogo/classificação/lineage,
* compliance, evidências e trilhas de auditoria. 

**Componentes principais (conceituais)**

* **Identity, Access & PAM (Z4-2.1)**
  IdP, IAM (RBAC/ABAC), acesso privilegiado just-in-time, mínimo privilégio para pessoas e workloads.

* **Secrets Manager (Z4-2.2)**
  Segredos **não ficam no código**; são injetados em runtime, com rotação, expiração e auditoria.

* **Key Management, KMS/HSM & PKI (Z4-2.3)**
  Geração e uso de chaves com políticas claras; certificados, TLS/mTLS, rotação e revogação.

* **Policy-as-Code & Authorization (Z4-2.4)**
  Autorização baseada em políticas versionadas (ex.: OPA/Rego), *deny by default*, testes e evidências.

* **Data Protection (Z4-2.5)**
  DLP, mascaramento, tokenização/pseudonimização; prevenção de PII em logs e saídas.

* **SIEM/UEBA/SOAR (Z4-2.6)**
  Centralização de logs e eventos de segurança, detecção de anomalias e automação de resposta.

* **Data Catalog, Classification & Lineage (Z4-2.7)**
  Marcação de datasets/campos com domínio, sensibilidade e finalidade; lineage ponta-a-ponta.

* **Compliance, Evidence & Audit Trails (Z4-2.8)**
  Trilhas de auditoria completas, evidence store (artefatos, decisões, approvals) e revisões de acesso.

📄 Detalhes: `docs/Z4/Z4-index.md` + arquivos `Z4-2.x.md`

---

### 📡 Z5 — Monitoring, Observability & Audit (Transversal)

A Z5 centraliza a **telemetria operacional e de segurança** de todas as zonas
para:

* detectar anomalias cedo,
* correlacionar eventos ponta-a-ponta,
* produzir trilhas de auditoria e evidências para resposta a incidentes e compliance. 

**Componentes principais (conceituais)**

* **Logging Estruturado & Correlação (Z5-2.1)**
  Logs em formato padronizado (ex.: JSON) com campos canônicos (`timestamp`, `trace_id`, `identidade`, `rota`, `dataset/ref`, resultado), sem PII.

* **Métricas, SLOs & Painéis (Z5-2.2)**
  Métricas por zona: ingestão (Z1), dados (Z2/Z3), custos e consumo; SLOs e error budgets com alertas.

* **Tracing Distribuído (Z5-2.3)**
  Traces que conectam a jornada dos dados de Z1 até o consumo, com `trace_id` único.

* **Monitoração de Ingestão & Upload Safety (Z5-2.4)**
  Indicadores da borda: bloqueios, anti-malware, rate limiting, fontes suspeitas.

* **Monitoração de Dados (Z5-2.5)**
  Eventos de acesso, qualidade e drift nas zonas de dados (Z2/Z3).

* **Consumo, Custos & FinOps (Z5-2.7)**
  KPIs de consumo: latência, erros, custos por rota/tenant, detecções de acesso fora de contrato.

* **Auditoria & Evidências (Z5-2.8)**
  Trilhas imutáveis ligando `request → dataset → decisão → persistência`, com links para evidências.

* **Alerting, Runbooks & Post-mortems (Z5-2.9)**
  Catálogo de alertas, runbooks versionados e post-mortems com ações rastreáveis.

* **Telemetria com Privacidade (Z5-2.10)**
  Redação/masking de campos sensíveis, retenção adequada, minimização de dados de telemetria.

📄 Detalhes: `docs/Z5/Z5-index.md` + arquivos `Z5-2.x.md`

---

## 2) Pilares de DataSecOps neste projeto

### 🔐 Segurança de Dados

* Criptografia em repouso e em trânsito.
* Controles de acesso com mínimo privilégio em todas as zonas.
* Camada de segurança transversal (Z4) garantindo:

  * identidade/autorização consistentes,
  * segredos fora do código,
  * chaves e políticas bem definidas,
  * proteção de dados sensíveis e PII.

### 📋 Governança & Classificação

* Política em `docs/DATA-CLASSIFICATION.md`.
* Catálogo de datasets em `docs/catalog/datasets.yaml`.
* Fichas de datasets (`DATASET_CARD.md`).
* Z4 integra classificação, catálogo e lineage com segurança e compliance.

### ✅ Data Quality como Código

* Schemas e regras de validação nas zonas Z2 e Z3.
* Entendimento de que falhas de DQ são incidentes de dados, não só bugs técnicos.

### 🔍 Observabilidade & Threat Model

* Threat model em `docs/THREAT-MODEL-DATASECOPS.md`.
* Camada de observabilidade (Z5) com:

  * logs estruturados,
  * métricas e SLOs,
  * tracing,
  * alertas, runbooks, auditoria.

### 🧱 Supply Chain de Dados

* Cadeia de suprimentos de dados: confiabilidade de fontes externas, integridade,
  rastreabilidade de transformações.
* Z4 e Z5 reforçam esse aspecto com políticas, SIEM, evidências e trilhas.

---

## 3) Como usar este repositório

1. **Comece pela visão macro**

   * Leia este `README.md`.
   * Veja os diagramas em `assets/` para visualizar o fluxo Z0–Z3 (e planejar Z4/Z5).

2. **Aprofunde em políticas e ameaças**

   * `docs/DATA-CLASSIFICATION.md`
   * `docs/THREAT-MODEL-DATASECOPS.md`

3. **Estude zona a zona (dados + segurança + observabilidade)**

   * `docs/Z0/Z0-index.md`
   * `docs/Z1/Z1-Index.md` + `Z1-2.x.md`
   * `docs/Z2/Z2-Index.md` + `Z2-2.x.md`
   * `docs/Z3/Z3-index.md` + `Z3-2.x.md`
   * `docs/Z4/Z4-index.md` + `Z4-2.x.md`
   * `docs/Z5/Z5-index.md` + `Z5-2.x.md`

4. **Adapte para seu contexto**

   * Ajuste a política de classificação para a realidade da sua organização.
   * Use catálogo e data cards como base para os seus datasets.
   * Adicione decisões, exceções e padrões específicos de cada zona.

---

## 4) Labs e próximos passos

Este repositório foca na **arquitetura de DataSecOps para pipeline de dados
(Z0–Z5)**.

A camada prática será adicionada posteriormente em:

* `hands-on/` → diretório planejado para conter labs, exemplos e implementações
  práticas (por cloud ou stack), sempre referenciando as zonas e controles
  definidos aqui.

Sugestões de evolução:

* Criar labs em `hands-on/` com:

  * ingestão segura (Z1),
  * armazenamento raw (Z2),
  * curadoria + DQ (Z3),
  * integração real com serviços de segurança (Z4) e observabilidade (Z5).
* Mapear esta arquitetura para provedores específicos (Azure, AWS, GCP, on-prem).
* Enriquecer o threat model com cenários reais de incidentes de dados.
