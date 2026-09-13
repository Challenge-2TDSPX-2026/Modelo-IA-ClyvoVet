# 🐾 CLYVO VET — Componente de Inteligência Preditiva para Triagem Clínica

> **Sprint 3:** Disruptive Architectures: IoT, IoB & Generative AI  
> **Integrantes:** Guilherme Santos Fonseca, RM 564232 - Gustavo Araujo da Silva, RM 566526, Anthony de Souza Henriques, RM 566188 
> **Link do Vídeo:**  `https://www.youtube.com/`  
> **Repositório GitHub:** `https://github.com/Challenge-2TDSPX-2026/Modelo-IA-ClyvoVet`

---

## 📋 Sumário
1. [Visão Geral do Projeto](#-visão-geral-do-projeto)
2. [Problema de Negócio & Valor Gerado](#-problema-de-negócio--valor-gerado)
3. [Abordagem de Inteligência Artificial](#-abordagem-de-inteligência-artificial)
4. [Mapeamento e Engenharia de Dados](#-mapeamento-e-engenharia-de-dados)
5. [Arquitetura de Integração & Fluxo de Dados](#-arquitetura-de-integração--fluxo-de-dados)
6. [Resultados e Avaliação dos Modelos](#-resultados-e-avaliação-dos-modelos)
7. [Estrutura do Repositório](#-estrutura-do-repositório)
8. [Instruções para Execução do Projeto](#-instruções-para-execução-do-projeto)

---

## 🐶 Visão Geral do Projeto

A **CLYVO VET** é uma plataforma focada no cuidado contínuo e preventivo para a saúde animal. No cenário veterinário tradicional, a identificação de riscos de complicações médicas ocorre frequentemente de forma reativa, após a exacerbação dos sintomas. 

Nesta 3ª Sprint, implementamos um **Componente de Inteligência Preditiva de IA** integrado ao ecossistema da CLYVO VET, que analisa automaticamente o histórico clínico, profilático e os atributos biológicos do pet no momento do atendimento para predizer o seu nível de risco médico (`RISK: 0 - Baixo Risco | 1 - Alto Risco`).

---

## 🎯 Problema de Negócio & Valor Gerado

### O Desafio
Dificuldade em identificar precocemente pets vulneráveis a complicações clínicas durante a rotina de consultas, resultando em diagnósticos tardios e falta de priorização no acompanhamento preventivo.

### Proposta de Valor
* **Para a Clínica Veterinária:** Suporte à tomada de decisão clínica, priorização de triagem automatizada, agendamento preventivo para pets de alto risco e redução de inconsistências no diagnóstico inicial.
* **Para o Tutor:** Monitoramento proativo do histórico do pet, intervenção médica antecipada e acompanhamento transparente e personalizado.
* **Para o Pet:** Redução do risco de complicações graves e aumento na qualidade e expectativa de vida por meio da medicina veterinária preventiva.

---

## 🧠 Abordagem de Inteligência Artificial

* **Tipo de Problema:** Classificação Preditiva Supervisionada (Binária).
* **Algoritmo Selecionado:** **XGBoost Classifier** (comparações realizadas contra Regressão Logística e Random Forest)[cite: 1].
* **Justificativa Técnica:**
  * **Alta Sensibilidade (Recall):** Em problemas de saúde, o custo de um *Falso Negativo* (não detectar um pet em alto risco) é extremamente alto[cite: 1]. O modelo XGBoost obteve **85.7% de Recall** no conjunto de testes[cite: 1].
  * **Desempenho Geral (ROC-AUC):** Alcançou o melhor equilíbrio com **ROC-AUC de 0.720**, superando os demais modelos na capacidade de separação de classes[cite: 1].
  * **Eficiência e Explicabilidade:** Modelos baseados em *Gradient Boosting* apresentam baixíssimo tempo de resposta (inferência em milissegundos) e oferecem mapa claro da importância das variáveis (*Feature Importance*), garantindo transparência ao médico veterinário.

---

## 📊 Mapeamento e Engenharia de Dados

### Fonte e Origem dos Dados
Dataset estruturado proveniente dos prontuários e atendimentos da plataforma ProntPet/CLYVO VET (`dataset-prontpet.csv`)[cite: 1].

### Trata de Granularidade (Consolidação de Dados)
O dataset bruto continha 289 registros em um grão relacional de 1-para-Muitos devido à multiplicidade de vacinas aplicadas por pet[cite: 1]. Foi realizada uma consolidação agrupada por consulta (`ID_CONSULTATION`), reduzindo o dataset para **100 consultas únicas** e gerando a feature agregada `TOTAL_VACCINES_REGISTERED`[cite: 1].

### Atributos Utilizados (Features para o Modelo)

| Atributo | Tipo de Dado | Descrição / Engenharia de Atributos |
| :--- | :--- | :--- |
| `PET_WEIGHT` | Numérico | Peso do pet tratado para formato float[cite: 1]. |
| `PET_AGE_YEARS` | Numérico | Idade exata em anos calculada a partir da data de nascimento e da consulta[cite: 1]. |
| `IS_SENIOR` | Binário | Indicador de idade avançada (1 se `PET_AGE_YEARS` >= 7.0, 0 se menor)[cite: 1]. |
| `IS_CASTRATED` | Binário | Flag de castração (1 se Castrado, 0 se não)[cite: 1]. |
| `HAS_ALLERGY` | Binário | Indicador de presença de alergias prévias[cite: 1]. |
| `HAS_CHRONIC_DISEASE` | Binário | Indicador de presença de doenças crônicas ativas[cite: 1]. |
| `TOTAL_VACCINES_REGISTERED` | Numérico | Quantidade total de registros vacinais associados até a data[cite: 1]. |
| `SPECIES` | Categórico | One-Hot Encoded (Cachorro / Gato)[cite: 1]. |
| `PET_GENDER` | Categórico | One-Hot Encoded (Macho / Fêmea)[cite: 1]. |
| `PET_BLOOD_TYPE` | Categórico | One-Hot Encoded (Tipos sanguíneos específicos de cães e gatos)[cite: 1]. |

---

## 🏗️ Arquitetura de Integração & Fluxo de Dados

### Diagrama Arquitetural de Comunicação

```text
+-------------------------------------------------------------------------+
|                              FRONT-END                                  |
|                 (Aplicação Web / Mobile CLYVO VET)                      |
+-------------------------------------------------------------------------+
                                    │
                                    │  1. Cadastro de Consulta e Sintomas
                                    ▼
+-------------------------------------------------------------------------+
|                               BACK-END                                  |
|                    (API Principal da Aplicação)                         |
+-------------------------------------------------------------------------+
       │                                                   │
       │ 2. Salva e busca histórico                        │ 3. Envia atributos
       ▼                                                   ▼    do pet em JSON
+-----------------------------------+             +-----------------------+
|          BANCO DE DADOS           |             |   MICROSERVIÇO DE IA  |
|      (Prontuário / Consultas)     |             |  (API Preditiva Flask)|
+-----------------------------------+             +-----------------------+
                                                           │
                                                           │ 4. Executa o modelo
                                                           ▼    XGBoost (.pkl)
                                                  +-----------------------+
                                                  | INFERÊNCIA DE RISCO   |
                                                  |  (0: Baixo / 1: Alto) |
                                                  +-----------------------+
                                                           │
                                                           │ 5. Retorna Score %
                                                           │    e Recomendação
                                                           ▼
+-------------------------------------------------------------------------+
|                    PAINEL DO VETERINÁRIO / TUTOR                        |
|            (Exibe alerta de risco e plano preventivo)                   |
+-------------------------------------------------------------------------+
