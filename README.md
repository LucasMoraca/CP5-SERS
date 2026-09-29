# Checkpoint 02 — APIs, Energias Renováveis e Aprendizado de Máquina

**Disciplina:** Soluções em Energias Renováveis e Sustentáveis — FIAP, 1CCPX, 2º semestre
**Grupo:**
Gabriel Barbosa Furin - RM: 572941
Gabriel de Almeida Santos - RM: 569395
Herbert Soares de Jesus - RM: 571507
Lucas Kiodi Moraca - RM: 571004
Renan Fracalossi Mano da Silva - RM: 569610

## Objetivo

Consultar duas APIs públicas de dados de energia/clima, gerar os conjuntos de dados e resolver duas tarefas independentes de aprendizado de máquina em Python, comparando **três algoritmos em cada uma**:

1. **Classificação** — prever a fonte de um empreendimento de geração (Solar, Eólica ou Hidráulica) a partir da potência outorgada e da localização.
2. **Regressão** — estimar a radiação solar global horizontal (W/m²) em Petrolina (PE) a partir de variáveis meteorológicas e da hora do dia.

## Dados

| Arquivo | Fonte | Período / recorte | Entradas | Alvo |
|---|---|---|---|---|
| `aneel_classificacao_orange.csv` | [ANEEL — SIGA](https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel) (API CKAN/DataStore, recurso `11ec447d-698d-4ab8-977f-b424d5deee6a`) | Cadastro vigente no dia da consulta; até 1.200 registros por sigla (UFV, EOL, UHE, PCH, CGH) | `potencia_kw`, `latitude`, `longitude` | `fonte` |
| `meteo_regressao_orange.csv` | [Open-Meteo — Historical Weather API](https://open-meteo.com/en/docs/historical-weather-api) | Petrolina (−9,39; −40,50), 01/04/2025 a 30/06/2025, horas locais 7h–17h, fuso `America/Recife` | `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh`, `hora` | `radiacao_w_m2` |

As duas consultas são públicas e **não usam token**. A ANEEL registra empreendimentos em várias fases (os dados não medem energia gerada) e o Open-Meteo fornece valores estimados por modelo/reanálise, não medições de um painel.

## Estrutura do repositório

```
├── README.md                                   # este arquivo
├── Aula_APIs_Energia_Renovavel_ML.ipynb        # notebook do professor + solução (APIs → CSVs → 6 modelos)
├── Enunciado_Avaliacao_APIs_Energia_ML.md      # enunciado da avaliação (professor)
├── Enunciado_Orange_Energias_Renovaveis.md     # enunciado da atividade no Orange (professor)
├── Fluxos_para_Classificacao_e_Regressao.ows   # fluxo do Orange
├── aneel_classificacao_orange.csv              # gerado pelo notebook (Tarefa 1)
├── meteo_regressao_orange.csv                  # gerado pelo notebook (Tarefa 2)
├── meteo_treino_orange.csv / meteo_teste_orange.csv   # divisão temporal 80/20 para o Orange
├── resultados_classificacao.csv / resultados_regressao.csv
└── figuras/                                    # gráficos do notebook e capturas do Orange
```

## Como executar

```bash
git clone <URL-deste-repositório>
cd <pasta-do-repositório>
python -m venv .venv
# Windows: .venv\Scripts\activate    |  Linux/Mac: source .venv/bin/activate
pip install numpy pandas matplotlib scikit-learn jupyter
jupyter notebook Aula_APIs_Energia_Renovavel_ML.ipynb
```

No Jupyter, use **Kernel → Restart & Run All**. No **Google Colab**: envie o `README.md` na aba Arquivos antes de rodar e use **Ambiente de execução → Executar tudo**; depois baixe os arquivos gerados. O notebook consulta as duas APIs, grava os CSVs, treina os seis modelos, salva as figuras em `figuras/` e atualiza a seção de resultados deste README. É necessário acesso à internet para a etapa de consulta.

## Metodologia

| | Tarefa 1 — Classificação | Tarefa 2 — Regressão |
|---|---|---|
| Algoritmos | Regressão Logística · kNN (k=15) · Random Forest | Regressão Linear · Árvore de Decisão · Random Forest |
| Divisão | Hold-out **estratificado** 80/20, `random_state=42` (a mesma para os três) | **Temporal**: 80% primeiras horas treino, 20% finais teste, sem embaralhar |
| Pré-processamento | `log1p` na potência + padronização dentro de `Pipeline` (só LR e kNN), ajustado apenas no treino | Padronização só na Regressão Linear, ajustada no treino |
| Métricas | Accuracy, Precision, Recall, F1 (média **macro**) + matriz de confusão; F1 macro em CV-5 no treino como checagem | MAE (W/m²), MSE ((W/m²)²), RMSE, R² + gráfico real × previsto |

## Resultados

<!-- RESULTADOS:INICIO -->
### Tarefa 1 — Classificação (teste, hold-out estratificado 80/20, `random_state=42`, média macro)

| Algoritmo | Accuracy | Precision (macro) | Recall (macro) | F1 (macro) | F1 (weighted) |
|---|---|---|---|---|---|
| Random Forest | 0.972 | 0.973 | 0.970 | 0.971 | 0.972 |
| kNN (k=15) | 0.972 | 0.972 | 0.970 | 0.971 | 0.972 |
| Regressão Logística | 0.820 | 0.835 | 0.818 | 0.816 | 0.819 |

- Melhor modelo: **Random Forest** (F1 macro = 0.971).
- Par de classes mais confundido: **Eólica ↔ Solar** (9 erros no teste).
- Amostra: 3876 empreendimentos (Hidráulica: 1476, Solar: 1200, Eólica: 1200).

### Tarefa 2 — Regressão (teste = 20% finais das horas, sem embaralhar)

| Algoritmo | MAE (W/m²) | MSE ((W/m²)²) | RMSE (W/m²) | R² |
|---|---|---|---|---|
| Random Forest | 69.277 | 7,859.748 | 88.655 | 0.832 |
| Árvore de Decisão | 84.493 | 13,089.783 | 114.411 | 0.721 |
| Regressão Linear | 145.205 | 30,034.201 | 173.304 | 0.360 |

- Melhor modelo: **Random Forest** (MAE = 69.3 W/m², R² = 0.832).
- Referência só com a hora: MAE = 132.1 W/m², R² = 0.287.
- Variável mais importante (permutação): **hora**.
- Período: 01/04/2025 a 30/06/2025; 1001 horas (treino 800 / teste 201).
<!-- RESULTADOS:FIM -->

## Conclusões

### Tarefa 1 — Classificação da fonte

- Potência e localização já separam bem boa parte dos casos, porque cada fonte tem regiões e faixas de potência típicas: a eólica se concentra no Nordeste e no Sul; a hidráulica acompanha as bacias do Sul/Sudeste/Centro-Oeste; a solar é mais espalhada.
- kNN e Random Forest capturam as "manchas" geográficas de cada fonte; a Regressão Logística, com fronteiras lineares, depende mais da potência. O melhor modelo e as métricas estão na seção Resultados.
- A maior confusão aparece entre classes com potência e região parecidas — tipicamente **Solar × Hidráulica** (UFVs e pequenas centrais CGH/PCH têm potências na mesma faixa e coexistem em MG e no Sudeste) e, no Nordeste, **Solar × Eólica**.
- **Limitações:** as entradas não trazem nenhuma informação física do recurso (irradiação, vento, rios, relevo); as coordenadas são aproximadas; há empreendimentos em fases diferentes misturados; e as proporções das classes resultam do limite de 1.200 registros por sigla — **não** representam a matriz energética brasileira.

### Tarefa 2 — Regressão da radiação solar

- A **hora do dia** resume a posição do Sol, que define o máximo de radiação possível. A relação tem forma de sino (sobe até o meio-dia e cai), por isso a Regressão Linear rende menos que os modelos de árvore; temperatura e umidade acompanham o mesmo ciclo diário e dividem parte dessa informação.
- As variáveis meteorológicas — sobretudo **nuvens** e **umidade** — explicam os desvios em relação à curva típica do dia; o ganho sobre a referência "só a hora" mede essa contribuição.
- O teste cobre o fim de junho, perto do solstício de inverno, com o Sol mais baixo que no período de treino — o modelo não conhece a época do ano e pode errar para mais.
- **Radiação ≠ geração elétrica:** W/m² é densidade de potência no plano horizontal; a energia (kWh) depende da área e da inclinação dos módulos, da eficiência, das perdas por temperatura, do inversor, de sujeira e sombreamento. Estimar a radiação é só a primeira etapa de uma estimativa de geração fotovoltaica.

## Atividade complementar — Orange Data Mining

Fluxo: `Fluxos_para_Classificacao_e_Regressao.ows`. Ao fluxo do professor foram acrescentados, no próprio Orange, o **Random Forest** nas duas tarefas e o arquivo de teste temporal na regressão.

### 1. Classificação — ANEEL

![Fluxo de classificação](figuras/orange_fluxo_classificacao.png)

- **Dados:** `aneel_classificacao_orange.csv` → **Select Columns** — Features: `potencia_kw`, `latitude`, `longitude`; Target: `fonte`.
- **Algoritmos:** Logistic Regression · kNN · Random Forest.
- **Avaliação (Test & Score):** _Random sampling_, 80% treino / 20% teste, estratificado, 10 repetições (ou: _Cross validation_, 5 folds, estratificado) — **a mesma configuração para os três**.

![Test & Score — classificação](figuras/orange_test_score_classificacao.png)
![Confusion Matrix](figuras/orange_matriz_confusao_orange.png)

| Algoritmo | CA | Precision | Recall | F1 |
|---|---|---|---|---|
| Logistic Regression | | | | |
| kNN | | | | |
| Random Forest | | | | |

**Análise:** _(2–4 frases)_ Qual teve o melhor F1; quais classes a Confusion Matrix mostra mais confundidas (ex.: Solar ↔ Hidráulica); limitação de usar só potência e localização. A quantidade de exemplos por classe vem do limite da consulta e **não** representa a participação das fontes na matriz energética.

### 2. Regressão — Open-Meteo

![Fluxo de regressão](figuras/orange_fluxo_regressao.png)

- **Dados de treino:** `meteo_treino_orange.csv` (80% primeiras horas). **Dados de teste:** `meteo_teste_orange.csv` (20% finais). Os dois arquivos são gerados pela última célula do notebook.
- **Select Columns (nos dois):** Features: `temperatura_c`, `umidade_pct`, `nuvens_pct`, `vento_kmh`, `hora`; Target: `radiacao_w_m2`; Meta: `data_hora`.
- **Algoritmos:** Linear Regression · Tree · Random Forest.
- **Avaliação (Test & Score):** **Test on test data** (divisão temporal, igual à do notebook).

![Test & Score — regressão](figuras/orange_test_score_regressao.png)
![Predictions](figuras/orange_predictions_regressao.png)

| Algoritmo | MAE (W/m²) | MSE ((W/m²)²) | RMSE (W/m²) | R² |
|---|---|---|---|---|
| Linear Regression | | | | |
| Tree | | | | |
| Random Forest | | | | |

_Se a sua versão do Orange mostrar só RMSE: MSE = RMSE²._

**Análise:** _(2–4 frases)_ Qual modelo errou menos; a Linear Regression perde porque a radiação tem forma de sino ao longo do dia; a `hora` é a variável dominante (posição do Sol) e as nuvens explicam os desvios; radiação em W/m² não é geração elétrica (área, inclinação, eficiência, temperatura, inversor, perdas).

**Comparação com o notebook:** os valores só são diretamente comparáveis quando divisão e hiperparâmetros são equivalentes; os padrões do Orange no Tree e no Random Forest diferem dos usados no scikit-learn.

## Fontes

- ANEEL — SIGA: https://dadosabertos.aneel.gov.br/dataset/siga-sistema-de-informacoes-de-geracao-da-aneel
- Open-Meteo — Historical Weather API: https://open-meteo.com/en/docs/historical-weather-api
- Enunciados e material de referência: `Enunciado_Avaliacao_APIs_Energia_ML.md`, `Enunciado_Orange_Energias_Renovaveis.md` e https://github.com/prof-atritiack/CHECKPOINT_02_SERS_1CC_2SEM
