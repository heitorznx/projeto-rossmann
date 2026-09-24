# Rossmann Store Sales - Previsão de Faturamento

![status](https://img.shields.io/badge/status-concluído-brightgreen)

<img width="500" height="333" alt="image" src="https://github.com/user-attachments/assets/aa84855e-9636-46bb-9615-c4fa4dfb027d" />


## 1. Visão Geral do Projeto

Este é um projeto completo de previsão de vendas para as próximas 6 semanas de uma rede de farmácias (Rossmann). Para a construção da solução, foi aplicada a metodologia **CRISP-DM**, percorrendo as seguintes etapas:

1. Contexto e objetivo de negócio
2. Origem e tratamento dos dados
3. Exploração dos dados
4. Preparação e seleção de variáveis
5. Modelagem e resultados
6. Disponibilização do modelo (API + Bot no Telegram)

## 2. Contexto e Objetivo de Negócio

A Rossmann é uma das maiores redes de drogarias da Europa, com origem na Alemanha e operação em diversos países do continente, comercializando desde itens de higiene e beleza até produtos alimentícios.

O problema que motiva este projeto: hoje, a previsão de vendas de cada uma das 1.115 lojas da rede é feita de forma individual por cada gerente, sem um método padronizado. Como o resultado das vendas depende de variáveis como promoções ativas, presença de concorrência, feriados, época do ano e características próprias de cada loja, esse processo manual gera estimativas pouco confiáveis e inconsistentes entre as unidades.

Diante da necessidade da diretoria financeira de planejar um investimento em reforma das lojas — o que exige saber com antecedência quanto a rede deve faturar nas próximas 6 semanas — o projeto propõe substituir essas estimativas manuais por um único modelo preditivo, capaz de gerar previsões padronizadas e mais precisas para todas as lojas simultaneamente.

## 3. Origem e Tratamento dos Dados

Os dados públicos foram obtidos na plataforma Kaggle ([rossmann-store-sales](https://www.kaggle.com/c/rossmann-store-sales/data)):

- **train.csv** — histórico de vendas por loja.
- **test.csv** — dados históricos sem a variável de vendas.
- **store.csv** — informações complementares sobre cada loja.

Após a coleta, os dados passaram por um processo de limpeza: renomeação das colunas para snake case, identificação e tratamento de valores nulos/ausentes com base em regras de negócio, conversão dos tipos de dados (datas, inteiros, etc.) e geração de estatística descritiva para as variáveis numéricas e categóricas.

## 4. Exploração dos Dados

A etapa de exploração teve três objetivos principais: derivar novas variáveis, formular e validar hipóteses de negócio, e identificar quais fatores mais influenciam o volume de vendas.

### 4.1 Engenharia de Atributos e Mapa de Hipóteses

Foram criadas novas variáveis a partir de campos temporais e de contexto de loja, como semana do ano, dia, mês, ano, tempo de exposição à concorrência e tempo de promoção ativa. Para orientar a análise, foi construído um mapa relacionando o fenômeno estudado (vendas) aos fatores que atuam sobre ele — características da loja, do produto e do tempo.

### 4.2 Distribuição e Cruzamento das Variáveis

Foi analisada a distribuição individual da variável de vendas e das demais variáveis numéricas e categóricas. Em seguida, cruzando cada variável explicativa com a variável de vendas, foram testadas as 12 hipóteses de negócio levantadas no mapa de hipóteses.

#### 4.2.1 Principais Achados

- Lojas com **maior sortimento** vendem **menos**.
- Lojas com **concorrentes mais próximos** vendem **mais**.
- Lojas com **concorrentes há mais tempo** vendem **menos**.
- Lojas com **mais promoções consecutivas** vendem **menos**.
- Lojas abertas durante o **feriado de Natal** vendem **menos**.
- Lojas vendem **mais após o dia 10** de cada mês.
- Lojas vendem **menos aos finais de semana**.
- Lojas vendem **menos durante feriados escolares**, exceto em julho e agosto.

### 4.3 Relação entre Variáveis

A análise conjunta das variáveis numéricas e categóricas ajudou a identificar quais delas têm maior poder explicativo sobre as vendas, além de revelar variáveis correlacionadas entre si (multicolinearidade).

## 5. Preparação e Seleção de Variáveis

Antes de treinar os modelos, os dados foram transformados por meio de **encoding** (conversão de variáveis categóricas em numéricas), **rescaling** (padronização de escala com RobustScaler/MinMaxScaler) e **codificação cíclica** (variáveis de tempo como dia, mês e semana transformadas em seno/cosseno).

Os dados foram divididos em treino e teste respeitando a ordem temporal, e o algoritmo **Boruta** foi aplicado como seletor automático de variáveis relevantes. Somente as features indicadas pelo Boruta foram usadas no modelo final.

## 6. Modelagem e Resultados

Foram treinados e comparados diferentes modelos de regressão, validados com validação cruzada em série temporal:

| Modelo | MAE | MAPE | RMSE |
|---|---|---|---|
| Average Model (baseline) | 1354.80 | 45.5% | 1835.14 |
| Linear Regression | 1867.09 | 29.3% | 2671.05 |
| Linear Regression - Lasso | 1891.70 | 28.9% | 2744.45 |
| XGBoost Regressor | 1695.79 | 25.2% | 2478.40 |

> O **Random Forest Regressor** também foi testado, mas foi excluído da comparação final devido ao alto custo computacional e tempo de treinamento em relação aos demais modelos.

Para extrair o melhor desempenho do modelo, foi aplicada a técnica de **Random Search** para ajuste fino de hiperparâmetros, testando combinações de valores como `n_estimators`, `eta`, `max_depth`, `subsample`, `colsample_bytree` e `min_child_weight`. O modelo com melhor resultado foi o **XGBoost Regressor**, que após o ajuste alcançou:

| Modelo | MAE | MAPE | RMSE |
|---|---|---|---|
| **XGBoost Regressor (ajustado)** | **780.59** | **11.6%** | **1128.89** |

### 6.1 Impacto para o Negócio

Aplicando o ciclo completo do CRISP-DM, o modelo projetou que, nas próximas 6 semanas, as lojas da Rossmann devem faturar aproximadamente:

| Cenário | Valor |
|---|---|
| Pior cenário | R$ 284.670.508,25 |
| **Previsão** | **R$ 285.544.032,00** |
| Melhor cenário | R$ 286.417.593,06 |

O erro também foi avaliado individualmente por loja: a maioria apresenta erro próximo à média geral (~11,6%), enquanto um grupo pequeno de lojas com comportamento mais atípico concentra os maiores desvios de previsão.

## 7. Disponibilização do Modelo

Para tornar o modelo acessível a qualquer gerente, ele foi encapsulado em uma classe Python (`Rossmann`), responsável por reproduzir todo o pipeline de limpeza, engenharia de atributos e preparação de dados usado no treinamento. Essa classe é consumida por uma **API REST em Flask**, hospedada em produção (Render), que recebe os dados da loja e devolve a previsão de vendas.

A API foi conectada a um **bot do Telegram**, permitindo que qualquer gerente consulte a previsão de faturamento diretamente pelo celular.

Procure no Telegram por **[@rossmann_proj_bot](https://t.me/rossmann_proj_bot)** e envie o número da loja desejada para receber o faturamento estimado para as próximas 6 semanas.

**Exemplo:** para consultar a loja 21, basta enviar `/21` — a previsão é retornada automaticamente pelo bot.

## 8. Considerações Finais e Próximos Passos

Ao final do ciclo de CRISP-DM, o projeto concluiu que as lojas da Rossmann devem faturar aproximadamente **R$ 285,5 milhões** nas próximas 6 semanas, com um erro médio (MAPE) de aproximadamente **11,6%**.

Como melhorias futuras, ficam sugeridas:

- Executar novos ciclos completos do CRISP-DM.
- Criar/derivar novas variáveis durante a exploração dos dados.
- Testar outros modelos de ML (LightGBM, redes neurais).
- Investigar e tratar as lojas com maior erro de previsão (outliers).
- Automatizar o retraining do modelo com novos dados.

## 9. Stack Utilizada

- GitHub
- Jupyter Notebook
- Python
- Pandas / NumPy
- Scikit-learn
- Boruta
- XGBoost
- Matplotlib / Seaborn
- Flask
- Render
- Telegram Bot API
- requests
