# 📊 Guia Completo do Pandas para Análise de Dados

## 🎯 Introdução

O **Pandas** é uma das bibliotecas mais poderosas e populares para análise de dados em Python. Ele fornece estruturas de dados de alto desempenho e ferramentas para manipulação e análise de dados.

## 🏗️ Estruturas Fundamentais

### DataFrame
Um **DataFrame** é uma tabela bidimensional que contém dados organizados em linhas e colunas. É a estrutura de dados principal do Pandas.

#### Exemplo de Criação:
```python
import pandas as pd

# Criando um DataFrame simples
df = pd.DataFrame({
    'Sim': [50, 21], 
    'Não': [131, 2]
})
```

**Resultado:**
```
   Sim  Não
0   50  131
1   21    2
```

#### Características:
- Cada entrada corresponde a uma **linha** (registro) e uma **coluna**
- Os valores podem ser de qualquer tipo (inteiros, strings, etc.)
- O construtor `pd.DataFrame()` aceita um dicionário onde:
  - **Chaves** = nomes das colunas
  - **Valores** = listas com os dados

#### Personalizando Índices:
```python
df = pd.DataFrame({
    'Bob': ['Gostei.', 'Foi horrível.'], 
    'Sue': ['Muito bom.', 'Sem graça.']
}, index=['Produto A', 'Produto B'])
```

**Resultado:**
```
           Bob         Sue
Produto A  Gostei.     Muito bom.
Produto B  Foi horrível. Sem graça.
```

### Series
Uma **Series** é uma sequência unidimensional de dados. Pode ser vista como uma única coluna de um DataFrame.

#### Exemplo de Criação:
```python
# Criando uma Series simples
s = pd.Series([1, 2, 3, 4, 5])
```

**Resultado:**
```
0    1
1    2
2    3
3    4
4    5
dtype: int64
```

#### Series com Índices Personalizados:
```python
s = pd.Series(
    [30, 35, 40], 
    index=['Vendas 2015', 'Vendas 2016', 'Vendas 2017'], 
    name='Produto A'
)
```

**Resultado:**
```
Vendas 2015    30
Vendas 2016    35
Vendas 2017    40
Name: Produto A, dtype: int64
```

## 📁 Leitura de Arquivos

### Arquivos CSV
O formato mais comum para dados tabulares é o **CSV** (Comma-Separated Values).

#### Exemplo de Leitura:
```python
# Lendo um arquivo CSV
wine_reviews = pd.read_csv("../input/wine-reviews/winemag-data-130k-v2.csv")

# Verificando o tamanho do DataFrame
print(wine_reviews.shape)  # (129971, 14)
```

#### Parâmetros Importantes:
- `index_col=0`: Usa a primeira coluna como índice
- `header=0`: Especifica qual linha usar como cabeçalho
- `sep=','`: Define o separador (vírgula por padrão)

#### Salvando em CSV:
```python
# Salvando um DataFrame em CSV
df.to_csv('meus_dados.csv', index=False)
```

## 🔍 Seleção de Dados

### Acessadores Nativos

#### Seleção por Atributo:
```python
# Acessando uma coluna como atributo
reviews.country
```

#### Seleção por Índice:
```python
# Acessando uma coluna usando colchetes
reviews['country']
```

#### Seleção de Valores Específicos:
```python
# Obtendo o primeiro valor da coluna country
reviews['country'][0]  # 'Italy'
```

### Indexação com `iloc` e `loc`

#### `iloc` - Indexação Baseada em Posição
```python
# Primeira linha
reviews.iloc[0]

# Primeira coluna, todas as linhas
reviews.iloc[:, 0]

# Primeiras 3 linhas, primeira coluna
reviews.iloc[:3, 0]

# Linhas específicas
reviews.iloc[[0, 1, 2], 0]

# Últimas 5 linhas
reviews.iloc[-5:]
```

#### `loc` - Indexação Baseada em Rótulos
```python
# Valor específico por rótulo
reviews.loc[0, 'country']  # 'Italy'

# Múltiplas colunas
reviews.loc[:, ['taster_name', 'taster_twitter_handle', 'points']]
```

#### ⚠️ Diferença Importante:
- **`iloc`**: Usa indexação Python padrão (0:10 = 0,1,2,3,4,5,6,7,8,9)
- **`loc`**: Usa indexação inclusiva (0:10 = 0,1,2,3,4,5,6,7,8,9,10)

### Manipulação de Índices
```python
# Definindo uma nova coluna como índice
reviews.set_index("title")
```

## 🔍 Seleção Condicional

### Operadores de Comparação
```python
# Verificando se cada vinho é italiano
reviews.country == 'Italy'

# Filtrando vinhos italianos
reviews.loc[reviews.country == 'Italy']

# Vinhos italianos com pontuação >= 90
reviews.loc[(reviews.country == 'Italy') & (reviews.points >= 90)]

# Vinhos italianos OU com pontuação >= 90
reviews.loc[(reviews.country == 'Italy') | (reviews.points >= 90)]
```

### Seletores Condicionais Avançados

#### `isin()` - Valores em Lista:
```python
# Vinhos apenas da Itália ou França
reviews.loc[reviews.country.isin(['Italy', 'France'])]
```

#### `isnull()` e `notnull()` - Valores Nulos:
```python
# Filtrando vinhos com preço (sem valores nulos)
reviews.loc[reviews.price.notnull()]

# Filtrando vinhos sem preço (com valores nulos)
reviews.loc[reviews.price.isnull()]
```

## ✏️ Atribuição de Dados

### Atribuindo Valores Constantes:
```python
# Adicionando uma nova coluna com valor constante
reviews['critic'] = 'everyone'
```

### Atribuindo Valores Iteráveis:
```python
# Adicionando uma nova coluna com valores sequenciais
reviews['index_backwards'] = range(len(reviews), 0, -1)
```

## 📊 Funções de Resumo

### `describe()` - Estatísticas Descritivas:
```python
# Para dados numéricos
reviews.points.describe()

# Para dados categóricos
reviews.taster_name.describe()
```

### Funções Estatísticas Básicas:
```python
# Média
reviews.points.mean()

# Valores únicos
reviews.taster_name.unique()

# Contagem de valores
reviews.taster_name.value_counts()
```

## 🗺️ Mapeamento e Transformação

### `map()` - Transformando Series:
```python
# Remeando pontuações para 0
review_points_mean = reviews.points.mean()
reviews.points.map(lambda p: p - review_points_mean)
```

### `apply()` - Transformando DataFrames:
```python
def remean_points(row):
    row.points = row.points - review_points_mean
    return row

reviews.apply(remean_points, axis='columns')
```

### Operações Vetorizadas (Mais Rápidas):
```python
# Subtração da média (mais rápido que map)
reviews.points - review_points_mean

# Concatenação de strings
reviews.country + " - " + reviews.region_1
```

## 🔄 Análise por Grupos (GroupBy)

### Conceito Básico
O `groupby()` permite agrupar dados e aplicar operações em cada grupo separadamente.

#### Exemplo Simples:
```python
# Contando vinhos por pontuação (equivalente ao value_counts)
reviews.groupby('points').points.count()
```

**Resultado:**
```
points
80     397
81     692
      ... 
99      33
100     19
Name: points, Length: 21, dtype: int64
```

#### Estatísticas por Grupo:
```python
# Preço mínimo por pontuação
reviews.groupby('points').price.min()

# Aplicando múltiplas funções
reviews.groupby(['country']).price.agg([len, min, max])
```

### Aplicando Funções Personalizadas
```python
# Nome do primeiro vinho de cada vinícola
reviews.groupby('winery').apply(lambda df: df.title.iloc[0])

# Melhor vinho por país e província
reviews.groupby(['country', 'province']).apply(
    lambda df: df.loc[df.points.idxmax()]
)
```

## 📋 Multi-Índices

### O que são Multi-Índices?
Quando você agrupa por múltiplas colunas, o Pandas cria um **multi-índice** com múltiplos níveis.

#### Exemplo:
```python
# Agrupando por país e província
countries_reviewed = reviews.groupby(['country', 'province']).description.agg([len])

# Verificando o tipo do índice
type(countries_reviewed.index)  # pandas.core.indexes.multi.MultiIndex
```

**Resultado:**
```
len
country	province	
Argentina	Mendoza Province	3264
Other	536
...	...	...
Uruguay	San Jose	3
Uruguay	24
425 rows × 1 columns
```

### Trabalhando com Multi-Índices

#### Convertendo para Índice Simples:
```python
# Convertendo multi-índice para colunas normais
countries_reviewed.reset_index()
```

**Resultado:**
```
country	province	len
0	Argentina	Mendoza Province	3264
1	Argentina	Other	536
...	...	...	...
423	Uruguay	San Jose	3
424	Uruguay	Uruguay	24
425 rows × 3 columns
```

## 📈 Ordenação de Dados

### Ordenando por Valores
```python
# Ordenação crescente (padrão)
countries_reviewed.sort_values(by='len')

# Ordenação decrescente
countries_reviewed.sort_values(by='len', ascending=False)

# Ordenando por múltiplas colunas
countries_reviewed.sort_values(by=['country', 'len'])
```

### Ordenando por Índice
```python
# Ordenando pelo índice
countries_reviewed.sort_index()
```

## 💡 Dicas Importantes

### 1. **Performance**
- Use operações vetorizadas quando possível (mais rápidas)
- `map()` e `apply()` são mais flexíveis mas mais lentos

### 2. **Indexação**
- `iloc` para seleção por posição
- `loc` para seleção por rótulos
- Lembre-se da diferença de indexação entre os dois

### 3. **Seleção Condicional**
- Use `&` para AND lógico
- Use `|` para OR lógico
- Use `isin()` para múltiplos valores
- Use `isnull()`/`notnull()` para valores nulos

### 4. **Manipulação de Dados**
- `set_index()` para redefinir índices
- `to_csv()` para salvar dados
- Sempre verifique o resultado com `.head()` ou `.shape`

### 5. **Agrupamento**
- `groupby()` é poderoso para análises segmentadas
- Use `agg()` para múltiplas funções simultaneamente
- Multi-índices podem ser confusos - use `reset_index()` quando necessário

### 6. **Ordenação**
- `sort_values()` para ordenar por valores das colunas
- `sort_index()` para ordenar pelo índice
- Use `ascending=False` para ordem decrescente

## 🚀 Próximos Passos

Este guia cobre os fundamentos do Pandas. Para aprofundar seus conhecimentos, explore:

- **Merge e Join** de DataFrames
- **Pivot Tables** e **Crosstabs**
- **Visualização de Dados** com Matplotlib/Seaborn
- **Análise de Séries Temporais**
- **Manipulação de Dados Avançada** (melt, pivot, etc.)
- **Integração com outras bibliotecas** (NumPy, SciPy)

---

*🎓 **Dica**: Pratique com datasets reais! O Pandas se torna mais intuitivo com o uso constante.*

*📚 **Recursos Adicionais**: Consulte a documentação oficial do Pandas para exemplos mais avançados e casos de uso específicos.*
