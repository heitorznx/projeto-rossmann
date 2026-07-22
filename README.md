# Rossmann Store Sales - Previsão de Faturamento

![status](https://img.shields.io/badge/status-concluído-brightgreen)
<img width="1000" height="667" alt="image" src="https://github.com/user-attachments/assets/9ce1344b-b25a-4e6e-be0c-5ac495ac499d" />

## 1. Visão Geral do Projeto

Este é um projeto completo de previsão de vendas para as próximas 6 semanas de uma rede de farmácias (Rossmann). Para a construção da solução, foi aplicada a metodologia **CRISP-DM**, percorrendo as seguintes etapas:

1. Contexto e objetivo de negócio
2. Origem e tratamento dos dados
3. Exploração dos dados
4. Preparação e seleção de variáveis
5. Modelagem e resultados
6. Disponibilização do modelo (API + Bot no Telegram)

   <img width="1362" height="628" alt="image" src="https://github.com/user-attachments/assets/3472fa97-00e1-4a95-84b1-6c55f92ebdc9" />


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

## 4. Exploração dos Dados - EDA

A etapa de exploração teve três objetivos principais: derivar novas variáveis, formular e validar hipóteses de negócio, e identificar quais fatores mais influenciam o volume de vendas.

### 4.1 Feature engineering e Mapa de Hipóteses

Foram criadas novas variáveis a partir de campos temporais e de contexto de loja, como semana do ano, dia, mês, ano, tempo de exposição à concorrência e tempo de promoção ativa. Para orientar a análise, foi construído um mapa relacionando o fenômeno estudado (vendas) aos fatores que atuam sobre ele — características da loja, do produto e do tempo.

#### Mapa de Hipóteses
<img width="2232" height="1546" alt="image" src="https://github.com/user-attachments/assets/3e75f36c-5019-4905-bae4-f28c42ccc6e2" />


### 4.2 Distribuição das Variáveis (Análise Univariada)

Foi analisada a distribuição individual da variável de vendas e das demais variáveis numéricas e categóricas , para entender melhor seu comportamento.

<img width="2039" height="994" alt="image" src="https://github.com/user-attachments/assets/8d54b7a0-1f1c-4850-8c3d-948ff5da951e" />

### 4.3 Cruzamento de Variáveis (Análise Bivariada)

Cruzando cada variável explicativa com a variável de vendas, foram testadas as 12 hipóteses de negócio levantadas no mapa de hipóteses, gerando os principais achados do projeto.

#### 4.3.1 Principais Insights

1- Lojas com **maior sortimento** vendem **menos**.
<img width="2018" height="1011" alt="image" src="https://github.com/user-attachments/assets/c68f12b0-429c-483e-be32-3ccff275cd69" />


2- Lojas com **concorrentes mais próximos** vendem **mais**.
<img width="2014" height="1115" alt="image" src="https://github.com/user-attachments/assets/8781ed83-80bb-49a3-a7ff-5b1d1b7bd58f" />


3- Lojas com **concorrentes há mais tempo** vendem **menos**.
<img width="2002" height="1026" alt="image" src="https://github.com/user-attachments/assets/202ebfd1-ba97-41eb-ad2e-24add3274268" />


4- Lojas com **mais promoções consecutivas** vendem **menos**.
<img width="2016" height="1031" alt="image" src="https://github.com/user-attachments/assets/b7f86a9c-0807-4ae9-9ee3-6c58c77d4bea" />


5- Lojas abertas durante o **feriado de Natal** vendem **menos**.
<img width="2004" height="1013" alt="image" src="https://github.com/user-attachments/assets/279ec0f2-5981-42d5-9d50-ed9b3f6cea99" />


6- Lojas vendem **mais após o dia 10** de cada mês.
<img width="2018" height="1013" alt="image" src="https://github.com/user-attachments/assets/feaf47ee-f556-4d73-80ec-1a47d06666e2" />


7- Lojas vendem **menos aos finais de semana**.
<img width="2029" height="1012" alt="image" src="https://github.com/user-attachments/assets/2933dd47-4cc9-4388-864e-872fd81fa326" />


8- Lojas vendem **menos durante feriados escolares**, exceto em julho e agosto.
<img width="2004" height="1011" alt="image" src="https://github.com/user-attachments/assets/6f6fab28-9659-4cbb-9832-3ef7901b8394" />


### 4.4 Relação entre Variáveis (Análise Multivariada)

A análise conjunta das variáveis numéricas e categóricas ajudou a identificar quais delas têm maior poder explicativo sobre as vendas, além de revelar variáveis correlacionadas entre si (multicolinearidade).

#### 4.4.1 Variáveis Númericas

<img width="2018" height="1205" alt="image" src="https://github.com/user-attachments/assets/b1326bff-6a99-4e9b-a162-b67e7965aa32" />

#### 4.4.2 Variáveis Categóricas

<img width="1779" height="979" alt="image" src="https://github.com/user-attachments/assets/a756378e-eeea-4039-92f8-adc728f9132c" />

## 5. Preparação e Seleção de Variáveis

Antes de treinar os modelos, os dados foram transformados por meio de **encoding** (conversão de variáveis categóricas em numéricas), **rescaling** (padronização de escala com RobustScaler/MinMaxScaler) e **codificação cíclica** (variáveis de tempo como dia, mês e semana transformadas em seno/cosseno).

Os dados foram divididos em treino e teste respeitando a ordem temporal, e o algoritmo **Boruta** foi aplicado como seletor automático de variáveis relevantes. Somente as features indicadas pelo Boruta foram usadas no modelo final.

### 5.1 Colunas Selecionadas pelo Boruta
cols_selected_boruta = [
'store',
'promo',
'store_type',
'assortment',
'competition_distance',
'competition_open_since_month',
'competition_open_since_year',
'promo2',
'promo2_since_week',
'promo2_since_year',
'competition_time_month',
'promo_time_week',
'day_of_week_sin',
'day_of_week_cos',
'month_sin',
'month_cos',
'day_sin',
'day_cos',
'week_of_year_sin',
'week_of_year_cos']

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
