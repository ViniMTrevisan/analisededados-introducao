# Data Visualization — Guia prático com seaborn

Breve: este documento organiza o conteúdo do tutorial sobre visualização de
dados usando seaborn (com exemplos em pandas e matplotlib). Inclui trechos
de código, dicas práticas e sugestões de estilo — tudo formatado para
leitura e uso em notebooks.

> Observação: os caminhos de arquivo (`../input/*.csv`) usados nos exemplos
> são os mesmos do material original; ajuste conforme seu projeto local.

---

## Conteúdo

- 1) Introdução e ambiente
- 2) FIFA — séries temporais (lineplot)
- 3) Spotify — múltiplas séries temporais
- 4) Flight delays — barplot e heatmap
- 5) Insurance — scatter, regressão e `hue`
- 6) Iris — histogramas, KDE e plots por categoria
- 7) Tipos de gráficos e quando usar
- 8) Estilos e temas do seaborn
- 9) Boas práticas rápidas
- 10) Próximos passos

---

## 1) Introdução e ambiente

Seaborn é uma biblioteca de alto nível para visualização estatística em
Python. Os exemplos abaixo usam pandas + matplotlib + seaborn em um notebook.

Setup mínimo (execute no Jupyter):

```python
import pandas as pd
pd.plotting.register_matplotlib_converters()
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns
print("Setup Complete")
```

---

## 2) FIFA — séries temporais (lineplot)

Dataset: rankings históricos para 6 países (ARG, BRA, ESP, FRA, GER, ITA).

Carregar o CSV com datas no índice:

```python
fifa_filepath = "../input/fifa.csv"
fifa_data = pd.read_csv(fifa_filepath, index_col="Date", parse_dates=True)
fifa_data.head()
```

Plot básico (uma linha por país):

```python
plt.figure(figsize=(16,6))
sns.lineplot(data=fifa_data)
```

Linha: ótimo para tendências e comparação temporal entre países.

---

## 3) Spotify — múltiplas séries temporais

Dataset: streams diários de músicas populares (2017–2018).

```python
spotify_filepath = "../input/spotify.csv"
spotify_data = pd.read_csv(spotify_filepath, index_col="Date", parse_dates=True)
spotify_data.head()
```

Plot de todas as séries:

```python
plt.figure(figsize=(14,6))
sns.lineplot(data=spotify_data)
```

Plot com título e tamanho:

```python
plt.figure(figsize=(14,6))
plt.title("Daily Global Streams of Popular Songs in 2017-2018")
sns.lineplot(data=spotify_data)
```

Plotar apenas duas séries (exemplo):

```python
plt.figure(figsize=(14,6))
plt.title("Daily Global Streams of Popular Songs in 2017-2018")
sns.lineplot(data=spotify_data['Shape of You'], label="Shape of You")
sns.lineplot(data=spotify_data['Despacito'], label="Despacito")
plt.xlabel("Date")
```

Dica: use `list(spotify_data.columns)` para ver nomes de colunas.

---

## 4) Flight delays — barras e heatmap

Dataset: atrasos médios de chegada por companhia (2015), indexado por mês.

```python
flight_filepath = "../input/flight_delays.csv"
flight_data = pd.read_csv(flight_filepath, index_col="Month")
flight_data
```

Bar chart (Spirit Airlines - coluna 'NK'):

```python
plt.figure(figsize=(10,6))
plt.title("Average Arrival Delay for Spirit Airlines Flights, by Month")
sns.barplot(x=flight_data.index, y=flight_data['NK'])
plt.ylabel("Arrival delay (in minutes)")
```

Heatmap para todas as companhias:

```python
plt.figure(figsize=(14,7))
plt.title("Average Arrival Delay for Each Airline, by Month")
sns.heatmap(data=flight_data, annot=True)
plt.xlabel("Airline")
```

`annot=True` mostra os valores nas células; ótimo para análise rápida de
padrões sazonais.

---

## 5) Insurance — scatter, regressão e colorização por categoria

Dataset: charges de seguro com variáveis como `bmi`, `charges`, `smoker`.

```python
insurance_filepath = "../input/insurance.csv"
insurance_data = pd.read_csv(insurance_filepath)
insurance_data.head()
```

Scatter simples (bmi x charges):

```python
sns.scatterplot(x=insurance_data['bmi'], y=insurance_data['charges'])
```

Adicionar linha de regressão:

```python
sns.regplot(x=insurance_data['bmi'], y=insurance_data['charges'])
```

Colorir por categoria (smoker):

```python
sns.scatterplot(x=insurance_data['bmi'], y=insurance_data['charges'], hue=insurance_data['smoker'])
# Ou usar lmplot para ajustar duas regressões separadas
sns.lmplot(x="bmi", y="charges", hue="smoker", data=insurance_data)
```

Comentário: `hue` adiciona uma terceira dimensão categórica e facilita
comparações entre grupos (ex.: fumantes x não fumantes).

---

## 6) Iris — histogramas, KDE e plots por categoria

Dataset clássico: medidas de sépalas/pétalas e `Species`.

```python
iris_filepath = "../input/iris.csv"
iris_data = pd.read_csv(iris_filepath, index_col="Id")
iris_data.head()
```

Histograma de petal length:

```python
sns.histplot(iris_data['Petal Length (cm)'])
```

KDE (densidade):

```python
sns.kdeplot(data=iris_data['Petal Length (cm)'], fill=True)
```

KDE por espécie:

```python
sns.kdeplot(data=iris_data, x='Petal Length (cm)', hue='Species', fill=True)
```

2D KDE (jointplot):

```python
sns.jointplot(x=iris_data['Petal Length (cm)'], y=iris_data['Sepal Width (cm)'], kind="kde")
```

Insight: a distribuição de `Petal Length` costuma separar `Iris-setosa`
das outras espécies.

---

## 7) Tipos de gráficos e quando usar

- Trends: `sns.lineplot` — séries temporais e comparações ao longo do tempo.
- Relationship: `sns.scatterplot`, `sns.regplot`, `sns.lmplot`, `sns.swarmplot`.
- Categorical / aggregated: `sns.barplot`.
- Matrix / tabelas: `sns.heatmap`.
- Distributions: `sns.histplot`, `sns.kdeplot`, `sns.jointplot`.

---

## 8) Estilos e temas do seaborn

Seaborn oferece temas prontos:

```python
# opções comuns: "darkgrid", "whitegrid", "dark", "white", "ticks"
sns.set_style("darkgrid")
plt.figure(figsize=(12,6))
sns.lineplot(data=spotify_data)
```

Troque o tema para encontrar a estética que melhor combina com o
relatório ou apresentação.

---

## 9) Boas práticas rápidas

- Sempre rodar `head()`, `info()` e `isnull().sum()` ao abrir dados.
- Escolher o tipo de gráfico de acordo com a pergunta de análise.
- Padronizar títulos, labels e legendas para facilitar interpretação.
- Para muitas séries, destaque as mais relevantes ou use faceting
+  (`sns.FacetGrid`) em vez de plotar tudo junto.

---

