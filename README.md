# 🏪 Rossmann Store Sales Prediction

Projeto de ciência de dados ponta a ponta (CRISP-DM) para prever as vendas das próximas 6 semanas das lojas da rede **Rossmann**, uma das maiores redes de farmácias da Europa. O projeto usa a base pública do desafio [Rossmann Store Sales](https://www.kaggle.com/c/rossmann-store-sales) do Kaggle.

## 📌 Problema de negócio

O CFO da Rossmann precisa reformar as lojas da rede e, para isso, solicitou a cada gerente uma previsão de vendas para as próximas 6 semanas. Como as previsões variavam muito entre lojas (dependendo de fatores como sazonalidade, promoções, feriados, competição e portfólio de produtos), o time de Data Science assumiu o desafio de construir um modelo único capaz de prever as vendas de cada loja com maior precisão, entregando o resultado através de um bot no Telegram/API acessível pelos gerentes a qualquer momento.

## 🗂️ Estrutura do projeto (CRISP-DM)

O notebook está organizado seguindo o ciclo CRISP-DM:

| Passo | Etapa |
|---|---|
| 1 | Descrição dos Dados (dimensão, tipos, valores ausentes, estatística descritiva) |
| 2 | Feature Engineering (mapa mental de hipóteses + criação de novas variáveis) |
| 3 | Filtragem de Variáveis (linhas e colunas) |
| 4 | Análise Exploratória de Dados (univariada, bivariada e multivariada) |
| 5 | Preparação dos Dados (normalização, rescaling, encoding, transformação cíclica) |
| 6 | Feature Selection (Boruta) |
| 7 | Machine Learning Modelling (baseline, modelos lineares, Random Forest, XGBoost) |
| 8 | Hyperparameter Fine Tuning |
| 9 | Tradução e Interpretação do Erro (impacto de negócio) |
| 10 | Deploy do Modelo em Produção (API Flask + classe de pipeline) |

## 🔎 Principais insights da análise exploratória

Das 12 hipóteses de negócio levantadas, as validadas com os dados foram, entre outras:

- **Lojas com maior sortimento vendem menos** (contrário à hipótese inicial).
- **Lojas com competidores mais próximos vendem mais.**
- **Lojas com competidores há mais tempo vendem menos.**
- **Lojas vendem mais depois do dia 10 de cada mês.**
- **Lojas vendem menos nos finais de semana.**
- **Lojas vendem menos durante feriados escolares** (exceto em julho e agosto).

## 🤖 Modelagem

Foram testados e comparados diversos algoritmos de regressão, avaliados com validação cruzada em série temporal:

| Modelo | MAE | MAPE | RMSE |
|---|---|---|---|
| Average Model (baseline) | 1354.80 | 45.5% | 1835.14 |
| Linear Regression | 1867.09 | 29.3% | 2671.05 |
| Linear Regression - Lasso | 1891.70 | 28.9% | 2744.45 |
| XGBoost Regressor | 1695.79 | 25.2% | 2478.40 |
| **XGBoost Regressor (tuned)** | **780.59** | **11.6%** | **1128.89** |

O modelo final escolhido foi o **XGBoost Regressor**, com hiperparâmetros ajustados via Random Search, resultando em ganho expressivo de performance sobre o baseline.

**Feature Selection:** utilização do algoritmo **Boruta** para seleção das variáveis mais relevantes para o modelo.

## 💰 Resultado de negócio

A previsão do modelo final para as próximas 6 semanas indicou um faturamento total estimado de:

- **Cenário previsto:** R$ 285.544.032,00
- **Pior cenário:** R$ 284.670.508,25
- **Melhor cenário:** R$ 286.417.593,06

O erro (MAE/MAPE) também foi avaliado por loja individualmente, permitindo identificar quais lojas têm previsões mais e menos confiáveis.

## 🚀 Deploy em produção

O modelo foi encapsulado em uma classe Python (`Rossmann`) responsável por replicar todo o pipeline de limpeza, feature engineering e preparação de dados usado no treinamento. Essa classe é consumida por uma **API REST em Flask**, que recebe os dados brutos da loja em formato JSON e devolve a previsão de vendas.

Fluxo de deploy:

1. `rossmann/Rossmann.py` — classe com os métodos `data_cleaning`, `feature_engineering`, `data_preparation` e `get_prediction`.
2. `handler.py` — API Flask com o endpoint `POST /rossmann/predict`.
3. API hospedada em produção (Render), consumida via requisições HTTP para gerar previsões em tempo real para os gerentes de loja.

## 🛠️ Tecnologias utilizadas

- **Linguagem:** Python 3.9
- **Manipulação de dados:** pandas, numpy
- **Visualização:** matplotlib, seaborn
- **Machine Learning:** scikit-learn, xgboost, boruta
- **Deploy:** Flask, Render
- **Outros:** inflection, scipy, pickle

## 📁 Estrutura de pastas sugerida

```
projeto-rossmann/
├── data/
│   └── raw/                # train.csv, store.csv, test.csv (dados do Kaggle)
├── parameter/               # scalers e encoders salvos (.pkl)
├── model/                   # modelo treinado (.pkl)
├── rossmann/
│   └── Rossmann.py          # classe do pipeline de produção
├── handler.py                # API Flask
├── m10_v01_store_sales_prediction.ipynb   # notebook com o ciclo completo do projeto
└── README.md
```

## ▶️ Como reproduzir

1. Clone o repositório e baixe os dados do [Kaggle - Rossmann Store Sales](https://www.kaggle.com/c/rossmann-store-sales) em `data/raw/`.
2. Instale as dependências:
   ```bash
   pip install -r requirements.txt
   ```
3. Execute o notebook `m10_v01_store_sales_prediction.ipynb` para reproduzir todo o ciclo de análise e treinamento.
4. Para subir a API localmente:
   ```bash
   python handler.py
   ```
5. Faça uma requisição de teste:
   ```python
   import requests, json
   data = json.dumps(df_test.to_dict(orient='records'))
   r = requests.post('http://0.0.0.0:5000/rossmann/predict',
                      data=data, headers={'Content-type': 'application/json'})
   ```

## 📈 Próximos passos

- Testar novos algoritmos (LightGBM, redes neurais) e comparar performance.
- Automatizar o retraining do modelo com novos dados.
- Expandir a entrega das previsões via bot do Telegram.

---

Projeto desenvolvido para fins de estudo em Ciência de Dados, com base no dataset público do desafio Rossmann Store Sales (Kaggle).
