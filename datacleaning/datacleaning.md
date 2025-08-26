# Data Cleaning — Visualização e Boas Práticas

> Versão melhorada: conteúdo reorganizado, com sumário, títulos claros,
> blocos de código e explicações concisas em português para facilitar o
> estudo e a leitura.

Resumo rápido (em português)
- Objetivo: apresentar técnicas básicas de limpeza de dados (missing
    values, datas, encodings, texto inconsistente) com exemplos em pandas.
- Inclui dicas práticas: quando remover dados, quando imputar, e como
    tratar problemas comuns de encoding e formatação.

Índice
- Missing values (valores faltantes)
- Preenchimento e remoção de NA
- Scaling vs Normalization
- Datas: como verificar e converter
- Encodings (codificação de caracteres)
- Limpeza de texto e fuzzy matching
- Resumo e próximos passos

---

## Missing values (valores faltantes)

Explicação curta: ao receber um dataset, o primeiro passo é inspecionar o
conteúdo e identificar campos com valores nulos (NaN/None). Decida se um
valor está ausente porque não foi registrado (imputável) ou porque não
faz sentido existir (deve permanecer NaN).

Exemplo: carregar libs e dados (Python / pandas)

```python
import pandas as pd
import numpy as np

# exemplo (ajuste o caminho para o seu ambiente)
nfl_data = pd.read_csv("../input/nflplaybyplay2009to2016/NFL Play by Play 2009-2017 (v4).csv")

# semente para reprodutibilidade
np.random.seed(0)
```

Verificando as primeiras linhas e contando NA's por coluna:

```python
nfl_data.head()
missing_values_count = nfl_data.isnull().sum()
missing_values_count.head(10)

total_cells = np.product(nfl_data.shape)
total_missing = missing_values_count.sum()
percent_missing = (total_missing / total_cells) * 100
print(f"Percent missing: {percent_missing:.2f}%")
```

Interpretação: saber a porcentagem de dados faltando ajuda a escolher a
estratégia (descartar, imputar, marcar explicitamente como "não
aplicável").

---

## Estratégias rápidas: remoção e preenchimento

1) Remover linhas com NA

```python
# remove todas as linhas que contêm ao menos um NA
nfl_data.dropna()
```

Aviso: se cada linha tem pelo menos um NA, o resultado pode ser vazio.

2) Remover colunas com NA

```python
columns_with_na_dropped = nfl_data.dropna(axis=1)
print(nfl_data.shape[1], '->', columns_with_na_dropped.shape[1])
```

3) Preencher NA automaticamente

```python
subset = nfl_data.loc[:, 'EPA':'Season'].head()
# substituir por zero
subset.fillna(0)
# ou usar backfill (valor seguinte) e depois zero
subset.fillna(method='bfill', axis=0).fillna(0)
```

Regra prática: se o NA significa "não registrado", imputar pode fazer
sentido; se o NA significa "não aplicável" (ex.: time penalizado quando
não houve penalidade), mantenha o NA ou use um valor categórico como
"neither".

---

## Scaling vs Normalization — diferença conceitual

- Scaling: altera a escala dos dados (ex.: min-max para [0,1]) sem
    mudar a forma da distribuição.
- Normalization: transforma a distribuição para algo mais próximo da
    normal (Gaussian), por exemplo usando Box-Cox.

Exemplo (esboço):

```python
original_data = np.random.exponential(size=1000)
# min-max scaling (exemplo)
# scaled_data = minmax_scaling(original_data, columns=[0])  # depende da função

# boxcox normalization
# normalized_data = stats.boxcox(original_data)
```

Quando aplicar: use scaling para métodos baseados em distância (KNN,
SVM); use normalização quando o método assume normalidade (LDA,
GaussianNB).

---

## Datas — verificar e converter

Problema comum: colunas que parecem datas são carregadas como `object`.

```python
print(landslides['date'].head())
print(landslides['date'].dtype)  # dtype('O') -> object (string)

# converter com formato explícito (mais rápido e seguro)
landslides['date_parsed'] = pd.to_datetime(landslides['date'], format="%m/%d/%y")

# ou permitir inferência (mais lento, às vezes ambíguo)
landslides['date_parsed'] = pd.to_datetime(landslides['date'], infer_datetime_format=True)
```

Dica: sempre verifique a distribuição de dias/meses para garantir parsing
correto (evita confundir dia e mês).

```python
day_of_month = landslides['date_parsed'].dt.day.dropna()
sns.histplot(day_of_month, bins=31)
```

---

## Encodings (codificação de caracteres)

Resumo: prefira UTF-8. Se a leitura de um CSV falhar com
UnicodeDecodeError, tente detectar/forçar outra codificação (ex.:
Windows-1252) ou usar `charset_normalizer`.

Exemplo de diagnóstico:

```python
with open("../input/kickstarter-projects/ks-projects-201801.csv", 'rb') as rawdata:
        sample = rawdata.read(10000)
        # usar charset_normalizer.detect(sample) -> retorna um palpite

# ler com encoding detectado
# kickstarter = pd.read_csv(path, encoding='Windows-1252')
```

Ao salvar, prefira UTF-8:

```python
kickstarter.to_csv("ks-projects-201801-utf8.csv", index=False)
```

---

## Limpeza de texto e fuzzy matching

Problema: entradas inconsistentes (ex.: ' Germany', 'germany', 'Germany').

Boas práticas:
- normalizar caixa (lower/upper)
- remover espaços em branco com `str.strip()`

```python
professors['Country'] = professors['Country'].str.lower().str.strip()
```

Para correções mais avançadas, use fuzzy matching (`fuzzywuzzy` ou
`rapidfuzz`) para agrupar strings similares e padronizá-las:

```python
from fuzzywuzzy import process, fuzz

def replace_matches_in_column(df, column, string_to_match, min_ratio=47):
        strings = df[column].unique()
        matches = process.extract(string_to_match, strings, limit=10, scorer=fuzz.token_sort_ratio)
        close_matches = [m[0] for m in matches if m[1] >= min_ratio]
        df.loc[df[column].isin(close_matches), column] = string_to_match

# exemplo
replace_matches_in_column(professors, 'Country', 'south korea')
```

---

## Resumo e próximos passos

- Inspecione o dataset assim que carregá-lo: `head()`, `info()`, `isnull()`.
- Decida para cada coluna se NA significa "não aplicável" ou "não
    registrado".
- Prefira conversões explícitas (datas, encodings). Use inferência só
    quando necessário.
- Padronize texto antes de agrupar ou mesclar dados (lower, strip,
    fuzzy-matching).


