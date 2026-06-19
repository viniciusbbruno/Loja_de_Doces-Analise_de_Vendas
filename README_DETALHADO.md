<p align="center">
  <img src="Dashboards/Capa.png" alt="Capa do projeto Domy Doces — Análise de Vendas" width="100%">
</p>

<h1 align="center">📊 Domy Doces — Da Planilha à Decisão</h1>

<p align="center">
  <em>Pipeline analítico de ponta a ponta com dados reais de uma loja de doces</em><br>
  <strong>Python (ETL) → SQL Server (modelagem) → Power BI (visualização)</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.x-3776AB?logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/SQL%20Server-2022-CC2927?logo=microsoftsqlserver&logoColor=white" alt="SQL Server">
  <img src="https://img.shields.io/badge/Power%20BI-Desktop-F2C811?logo=powerbi&logoColor=black" alt="Power BI">
  <img src="https://img.shields.io/badge/pandas-ETL-150458?logo=pandas&logoColor=white" alt="pandas">
</p>

---

## 📑 Índice

1. [Contexto do Projeto](#1-contexto-do-projeto)
2. [Arquitetura da Solução](#2-arquitetura-da-solução)
3. [Camada de Extração e Transformação (Python)](#3-camada-de-extração-e-transformação-python)
   - [3.1 ETL de Vendas — o fato](#31-etl-de-vendas--o-fato)
   - [3.2 ETL de Custos — a dimensão](#32-etl-de-custos--a-dimensão)
   - [3.3 Carga no SQL Server](#33-carga-no-sql-server)
   - [3.4 Validação e qualidade de dados](#34-validação-e-qualidade-de-dados)
4. [Modelagem no SQL Server](#4-modelagem-no-sql-server)
   - [4.1 As tabelas e a escolha de cada tipo de dado](#41-as-tabelas-e-a-escolha-de-cada-tipo-de-dado)
   - [4.2 As Views — regras de negócio no banco](#42-as-views--regras-de-negócio-no-banco)
5. [Do Dado à Decisão — o Dashboard](#5-do-dado-à-decisão--o-dashboard)
   - [5.1 Como os indicadores são construídos](#51-como-os-indicadores-são-construídos)
   - [5.2 Página 1 — Painel Executivo](#52-página-1--painel-executivo)
   - [5.3 Página 2 — Curva ABC](#53-página-2--curva-abc)
   - [5.4 Página 3 — Análise de Produtos](#54-página-3--análise-de-produtos)
6. [Principais Insights e Achados de Governança](#6-principais-insights-e-achados-de-governança)
   - [6.1 O que os dados revelaram sobre o negócio](#61-o-que-os-dados-revelaram-sobre-o-negócio)
   - [6.2 Achados de governança e qualidade de dados](#62-achados-de-governança-e-qualidade-de-dados)
   - [6.3 Conclusão](#63-conclusão)

---

## 1. Contexto do Projeto

A **Domy Doces** é uma loja de doces, balas, chocolates e itens de confeitaria localizada no **interior de São Paulo**. Como muitos negócios do varejo brasileiro, a loja opera com um sistema ERP que registra cada venda, mas **não transforma esse registro em informação acionável**. Os dados ficam "presos" em relatórios operacionais, sem visão analítica do desempenho.

Este projeto nasceu para responder, com dados reais, perguntas que todo dono de comércio faz mas raramente consegue medir:

- Quanto a loja fatura, e qual a **rentabilidade real** por trás desse faturamento?
- Quais produtos **sustentam o negócio** e quais apenas ocupam prateleira?
- O produto que mais **fatura** é também o que mais **sai**? (spoiler: nem sempre)
- Existe **padrão de comportamento** de compra ao longo da semana e do ano?
- Qual a **dependência** da loja em relação a poucos produtos?
- A **precificação atual** está coerente com o que de fato é praticado no caixa?
- Quais produtos **giram rápido** e quais estão **encalhados**?
- Onde estão as **oportunidades de margem** sem perder competitividade?

E essa lista é só o ponto de partida. Uma vez que a estrutura está montada, dado limpo, modelo relacionado e medidas confiáveis, o mesmo dashboard passa a responder dezenas de outras perguntas de negócio conforme elas surgem, sem precisar refazer nada. É essa a diferença entre montar um relatório pontual e construir uma **base analítica reutilizável**.

### Objetivo

Construir um **dashboard para tomada de decisão**, da extração do dado bruto até a visualização interativa, demonstrando domínio técnico sobre todo o ciclo de vida do dado: engenharia (ETL), modelagem dimensional, governança e visualização.

### Escopo dos dados

| Item | Detalhe |
|------|---------|
| **Período analisado** | Setembro/2025 a Maio/2026 |
| **Volume de vendas** | ~97 mil linhas de itens vendidos |
| **Cadastro de produtos** | ~5,2 mil SKUs |
| **Faturamento total** | R$ 1.363.262,02 |
| **Granularidade** | Item por item, por cupom de venda |

---

## 2. Arquitetura da Solução

O projeto segue uma arquitetura **ETL clássica em três camadas**, com separação clara de responsabilidades, cada ferramenta faz aquilo que faz de melhor.

```
┌─────────────────────┐
│   FONTES DE DADOS    │   • Vendas detalhadas (.xlsx, 9 abas mensais)
│   (sistema ERP)      │   • Cadastro de custo/estoque (.xls binário)
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   PYTHON  (ETL)      │   Extract → Transform
│   pandas, xlrd,      │   • Empilha, limpa e padroniza
│   SQLAlchemy         │   • Converte tipos e formatos (BR → ISO)
│                      │   • Separa em Fato e Dimensão
└──────────┬──────────┘
           │  Load
           ▼
┌─────────────────────┐
│   SQL SERVER         │   Camada de persistência e modelagem
│   fato_vendas        │   • Tabelas tipadas (Star Schema)
│   dim_produtos       │   • Views com regras de negócio
│   + Views            │   • JOIN fato × dimensão
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   POWER BI           │   Camada de apresentação
│   Modelo + DAX       │   • Medidas, calendário, parâmetros
│   3 páginas + capa   │   • Dashboard interativo
└─────────────────────┘
```

### A decisão estrutural: ETL, não ELT

O dado bruto **nunca** entra no banco "sujo". Toda a limpeza e padronização acontece no **Python**, e o SQL Server recebe apenas dado já tratado e confiável. Essa escolha reflete um princípio de governança:

> **O banco de dados é uma fonte da verdade, não um depósito de dado cru.** Ele serve para consultar, agregar e relacionar, não para corrigir.

Na prática, isso significa que qualquer pessoa (ou ferramenta) que consultar o SQL Server pode confiar que os tipos estão corretos, os valores estão normalizados e não há lixo de teste no meio dos dados de produção.

---

## 3. Camada de Extração e Transformação (Python)

Optei por concentrar toda a transformação no Python por uma razão prática: é onde eu tenho controle fino sobre cada linha do dado antes que ele vire "verdade" no banco. O Python lê os arquivos do ERP, corrige tipos, padroniza formatos e separa os dados em duas naturezas distintas, e só então entrega ao SQL Server.

Essa separação em duas naturezas foi uma decisão de modelagem tomada logo no início:

- **Fato** (`fato_vendas`), os eventos que acontecem e se acumulam no tempo: cada item vendido, em cada cupom, em cada dia.
- **Dimensão** (`dim_produtos`), o contexto descritivo e relativamente estável: o cadastro de cada produto, com custo e preço.

Essa é a base de um **Star Schema** (esquema estrela), o padrão de modelagem para análise. Ela evita repetir dados e permite que o Power BI relacione as tabelas com performance. Para refletir isso, cada fonte ganhou seu próprio script:

| Script | Responsabilidade | Saída |
|--------|------------------|-------|
| `etl_vendas.py` | Trata o histórico de vendas (o fato) | `vendas_consolidado.csv` |
| `etl_custos.py` | Trata o cadastro de produtos (a dimensão) | `custo_consolidado.csv` |
| `carga_sql.py` | Carrega os dois CSVs no SQL Server | Tabelas populadas |

---

### 3.1 ETL de Vendas — o fato

A primeira coisa que fiz foi abrir o arquivo de vendas e entender com o que estava lidando. Era um Excel com **9 abas**, uma para cada mês (Setembro/2025 a Maio/2026), e todas tinham exatamente a mesma estrutura de colunas. Isso definiu a estratégia: em vez de tratar aba por aba, fazia muito mais sentido **empilhar todas em uma única tabela** e tratar o conjunto de uma vez.

```python
"""
ETL - Vendas Detalhadas Domy Doces
Etapa 1: Empilhar abas e limpar colunas. Fato fica enxuto.
"""

import pandas as pd

# 1) caminho do arquivo
arquivo = r"C:\...\Vendas Detalhadas Produtos Domy Docesxlsx.xlsx"

# 2) abrir o Excel
xl = pd.ExcelFile(arquivo)
print("Abas:", xl.sheet_names)

# 3) ler cada aba e empilhar
frames = []
for aba in xl.sheet_names:
    df = pd.read_excel(xl, sheet_name=aba, dtype={'cod_ean': str})
    frames.append(df)
    print(aba, len(df), "linhas")

vendas = pd.concat(frames, ignore_index=True)
print("Total empilhado:", len(vendas))

# 4) limpezas basicas
vendas['des_produto'] = vendas['des_produto'].astype(str).str.strip()
vendas['cod_ean'] = vendas['cod_ean'].astype(str).str.strip()
vendas['dta_venda'] = pd.to_datetime(vendas['dta_venda'])

# 5) remover produto teste
antes = len(vendas)
vendas = vendas[~vendas['des_produto'].str.contains('TESTE', case=False, na=False)]
print("Removidas:", antes - len(vendas), "linhas de teste")

# 6) descartar colunas redundantes, zeradas, calculadas ou de dimensao
colunas_remover = [
    'val_acrescimo', 'val_desconto', 'valor_bruto',
    'val_troca', 'subtotal', 'des_produto'
]
vendas = vendas.drop(columns=colunas_remover)

# 7) conferencia
print("Total final:", len(vendas))
print("Periodo:", vendas['dta_venda'].min().date(), "a", vendas['dta_venda'].max().date())

# 8) salvar
vendas.to_csv('vendas_consolidado.csv', index=False, encoding='utf-8-sig')
```

#### O raciocínio por trás de cada tratamento

**Por que ler o código de barras como texto (`dtype={'cod_ean': str}`).**
Esse foi o primeiro problema que percebi ao inspecionar os dados. O EAN (`7891118027255`) *parece* um número, e se eu deixasse o pandas adivinhar o tipo, ele leria como número, e aí dois desastres silenciosos aconteceriam: o valor viraria notação científica (`7.891118e+12`), e qualquer código com **zero à esquerda** perderia esse dígito (`007891...` viraria `7891...`). Como o EAN é o que vai ligar a venda ao cadastro de produto mais à frente, perder um dígito significaria nunca mais conseguir casar os dois. A conclusão foi direta: **EAN é um identificador, não uma quantidade, não se faz conta com ele.** Por isso forço como texto já na leitura.

**Por que empilhar com `pd.concat`.**
Como as 9 abas eram idênticas na estrutura, unificá-las em um único DataFrame me deu uma base contínua para o período inteiro. Isso é o que permite, lá no final, perguntas como "faturamento de toda a temporada" funcionarem sem precisar juntar planilha na mão. E como cada linha já carrega sua própria `dta_venda`, o nome da aba (o mês) virou informação redundante, a data já diz tudo.

**Por que remover o `PRODUTO TESTE`.**
Ao olhar os nomes de produto, encontrei lançamentos de teste, aqueles que o operador faz no caixa só para conferir se está funcionando. Não são vendas reais, e se ficassem, contaminariam qualquer número de faturamento ou volume. Removi filtrando pela palavra `TESTE` no nome, antes que esse ruído chegasse ao banco.

**Por que enxugar o fato, a análise coluna por coluna.**
Aqui foi onde mais parei para pensar. Uma tabela fato precisa ser **estreita e somável**, só o que é evento puro. Então fui olhando coluna por coluna e perguntando "isso é mesmo um fato, ou é outra coisa?":

- `val_acrescimo` e `val_troca`, **estavam zeradas** em todas as linhas. Não acrescentavam informação nenhuma, saíram.
- `subtotal` e `valor_bruto`, eram apenas `quantidade × valor_unitário`, ou seja, **calculáveis** a partir do que eu já tinha. A regra que segui foi: não se armazena o que se pode calcular. Guardar resultado pronto só ocupa espaço e abre brecha pra inconsistência.
- `val_desconto`, quando olhei os valores, **estava tudo zerado** também. Como não havia desconto nenhum registrado no período, não havia informação a preservar, então removi junto.
- `des_produto`, essa foi decisão de modelagem, não de limpeza. O nome do produto é **atributo da dimensão**, não do fato. Repeti-lo em 93 mil linhas seria desnormalização à toa. O nome vai vir do `dim_produtos` na hora do JOIN, sem precisar carregar essa coluna gorda no fato.

O resultado é um fato limpo com só o essencial: **data, código do produto, código da venda, quantidade e valor unitário.**

---

### 3.2 ETL de Custos — a dimensão

O cadastro de produtos (com custo e preço) me deu mais trabalho que o de vendas, e por motivos que só apareceram quando coloquei a mão.

**O primeiro problema: o arquivo nem abria.** Tentei ler como qualquer Excel e não funcionou. Investigando, descobri que era um **`.xls` binário antigo** (formato pré-2007), não o `.xlsx` moderno. O pandas, por padrão, usa o motor `openpyxl`, que só entende `.xlsx`. Para esse formato legado, é preciso pedir explicitamente o motor `xlrd`. (Para piorar, cheguei a desconfiar que fosse um HTML disfarçado de planilha antes de entender que era binário mesmo, testar a leitura via `xlrd` foi o que resolveu de vez.)

**O segundo problema apareceu nos valores.** Quando finalmente abri e olhei a coluna de custo, os números não vinham como números, vinham como **texto no formato brasileiro**: `"11,65"`. Não dava para somar nem multiplicar nada daquilo. Precisei entender o padrão exato antes de tratar: havia ponto separando milhar e vírgula separando o decimal. A conversão tinha que respeitar essa ordem.

```python
"""
ETL — Relatório de Custo/Estoque
Etapa 2: Limpar a planilha de custo e preparar pra fazer o JOIN com vendas.
"""

import pandas as pd

arquivo = r"C:\...\Relatorio_Custo_Estoque.xls"

# 2) ler o arquivo (.xls binário antigo) — exige o engine 'xlrd'
custo = pd.read_excel(arquivo, engine='xlrd')

# 3) garantir que COD_EAN seja TEXTO (proteção contra notação científica)
custo['COD_EAN'] = custo['COD_EAN'].astype(str).str.strip()
custo['DES_PRODUTO'] = custo['DES_PRODUTO'].astype(str).str.strip()

# 4) descartar colunas que decidimos remover
colunas_remover = [
    'COD_INTERNO', 'DES_MARCA', 'NCM', 'COR', 'DEPARTAMENTO',
    'OBSERVACAO', 'OBSERVACAO_INTERNA', 'VAL_CUSTO_TOTAL', 'VAL_VENDA_TOTAL'
]
custo = custo.drop(columns=colunas_remover)

# 5) renomear pra letras minúsculas (padrão profissional)
custo = custo.rename(columns={
    'COD_EAN': 'cod_ean',
    'DES_PRODUTO': 'des_produto',
    'VAL_CUSTO': 'val_custo',
    'VAL_VENDA': 'val_venda',
    'QTD_ESTOQUE_ATUAL': 'qtd_estoque_atual'
})

# 6) converter valores numéricos (vêm como texto brasileiro: "11,65")
for col in ['val_custo', 'val_venda', 'qtd_estoque_atual']:
    custo[col] = (
        custo[col].astype(str)
        .str.replace('.', '', regex=False)   # remove ponto de milhar
        .str.replace(',', '.', regex=False)  # vírgula vira ponto decimal
    )
    custo[col] = pd.to_numeric(custo[col], errors='coerce')

# 9) salvar
custo.to_csv('custo_consolidado.csv', index=False, encoding='utf-8-sig')
```

#### O raciocínio por trás de cada tratamento

**A conversão dos valores em duas etapas, na ordem certa.**
O texto `"1.234,56"` só vira número se eu fizer duas trocas na sequência correta: primeiro **removo o ponto de milhar** (`1.234,56` → `1234,56`), depois **troco a vírgula decimal por ponto** (`1234,56` → `1234.56`). Se eu invertesse, trocando a vírgula antes de tirar o ponto, o resultado seria um número quebrado e errado. Só depois dessas duas etapas o `pd.to_numeric` consegue interpretar como valor de verdade.

**A rede de segurança do `errors='coerce'`.**
Coloquei esse parâmetro de propósito: se alguma célula vier corrompida e não converter, ela vira nulo em vez de derrubar o script inteiro. Prefiro um valor ausente que eu consigo rastrear depois do que um pipeline que quebra no meio da carga.

**O mesmo cuidado com o EAN.**
Repeti aqui a leitura do `cod_ean` como texto, pela mesma razão da planilha de vendas, esse código é a chave que vai ligar as duas tabelas, e ele precisa estar idêntico nos dois lados para o JOIN funcionar. Um deslize de tipo aqui quebraria a ligação inteira.

**A padronização dos nomes.**
As colunas do ERP vinham em maiúsculas (`VAL_CUSTO`). Renomeei tudo para minúsculo no padrão `snake_case` (`val_custo`), mantendo a mesma convenção do resto do projeto e evitando dor de cabeça com maiúscula/minúscula no SQL depois.

> **Sobre o estoque:** a coluna `qtd_estoque_atual` é tratada e carregada, mas decidi **deixá-la de fora das análises**. Conversando com o dono da loja, confirmei que o saldo de estoque do ERP não era confiável. Achei importante manter o dado no banco mas registrar claramente essa limitação, usar um número que eu sei estar errado seria pior do que não usá-lo.

---

### 3.3 Carga no SQL Server

Com os dois CSVs limpos e validados, o último script faz a ponte com o SQL Server e popula as tabelas. A lógica é direta: conecta no banco, esvazia a tabela de destino e sobe o CSV tratado. Esvaziar antes de inserir é o que me permite **rodar o script de novo quantas vezes eu quiser sem duplicar dado**, importante porque, a cada novo mês que a loja gerar, o pipeline vai ser reexecutado.

```python
"""
Carga SQL — Sobe os CSVs limpos pras tabelas do SQL Server.
"""

import pandas as pd
from sqlalchemy import create_engine, text
from urllib.parse import quote_plus

# 1) String de conexão
servidor = "VINICIUS"
banco = "projeto_dommy"
driver = "ODBC Driver 17 for SQL Server"

conn_str = (
    f"mssql+pyodbc://@{servidor}/{banco}"
    f"?driver={quote_plus(driver)}"
    f"&trusted_connection=yes"
)

engine = create_engine(conn_str, fast_executemany=True)

# --- Carregar dim_produtos ---
custo = pd.read_csv('custo_consolidado.csv', dtype={'cod_ean': str})
custo = custo.drop_duplicates(subset='cod_ean', keep='first')

with engine.begin() as conn:
    conn.execute(text("DELETE FROM dim_produtos"))

custo.to_sql('dim_produtos', engine, if_exists='append', index=False, chunksize=1000)

# --- Carregar fato_vendas ---
vendas = pd.read_csv('vendas_consolidado.csv', dtype={'cod_ean': str})
vendas['dta_venda'] = pd.to_datetime(vendas['dta_venda']).dt.date

with engine.begin() as conn:
    conn.execute(text("DELETE FROM fato_vendas"))

vendas.to_sql('fato_vendas', engine, if_exists='append', index=False, chunksize=2000)
```

Um cuidado que vale destacar: antes de subir a dimensão, removo duplicatas de `cod_ean` (`drop_duplicates`). Isso porque o `cod_ean` vai ser a **chave primária** de `dim_produtos` no banco, e chave primária não admite repetição, um produto, um registro. Garantir isso aqui evita que a carga falhe lá na frente.

---

### 3.4 Validação e qualidade de dados

Antes de confiar que esses dados estavam prontos para virar análise, eu quis testar uma coisa que me incomodava: será que o mesmo código de barras sempre se refere ao mesmo produto ao longo dos 9 meses? E será que a descrição que está no cadastro bate com o que de fato foi vendido?

Isso importava porque, mais à frente, eu ia ligar vendas e custo justamente pelo EAN. Se o mesmo código aparecesse com nomes diferentes ao longo do tempo, ou se o cadastro estivesse defasado em relação à realidade, o cruzamento poderia juntar informação errada. Então, em vez de assumir que estava tudo certo, escrevi um script só para checar:

```python
import pandas as pd

v = pd.read_csv('vendas_consolidado.csv', dtype={'cod_ean': str})
c = pd.read_csv('custo_consolidado.csv', dtype={'cod_ean': str})

# Quantas descrições DIFERENTES existem por EAN nas vendas?
descs_por_ean = v.groupby('cod_ean')['des_produto'].nunique()
print(f"Produtos com 1 só descrição:    {(descs_por_ean==1).sum()}")
print(f"Produtos com 2 descrições:      {(descs_por_ean==2).sum()}")
print(f"Produtos com 3+ descrições:     {(descs_por_ean>=3).sum()}")

# A última descrição (mais recente) bate com a do cadastro?
v['dta_venda'] = pd.to_datetime(v['dta_venda'])
ultima_desc = v.sort_values('dta_venda').groupby('cod_ean')['des_produto'].last().reset_index()
cmp = ultima_desc.merge(c[['cod_ean','des_produto']], on='cod_ean', how='inner')
```

A ideia foi medir, primeiro, quantos produtos mantinham uma descrição estável e quantos variavam, e depois comparar a descrição mais recente de cada produto nas vendas com a que estava registrada no cadastro. Esse tipo de checagem é o que separa um número confiável de um número bonito mas furado, e foi exatamente essa postura de **verificar antes de confiar** que acabou revelando os principais achados do projeto, que eu detalho no bloco de Insights.

---

## 4. Modelagem no SQL Server

Com os dados limpos saindo do Python, o SQL Server entra como a **camada onde o dado vira estrutura**. Aqui eu desenho duas tabelas que conversam entre si, o fato e a dimensão, e crio as views que aplicam as regras de negócio. É neste ponto que o "esquema estrela" sai do conceito e vira algo concreto: uma tabela central de eventos (`fato_vendas`) cercada pela tabela de contexto (`dim_produtos`), ligadas pelo código do produto.

A grande diferença em relação ao Python é que, aqui, **eu declaro o tipo de cada coluna**, e essa declaração é um contrato. Quando digo que uma coluna é `DATE`, o banco passa a recusar qualquer coisa que não seja uma data. Esse rigor é o que garante que o dado não se degrade com o tempo.

### 4.1 As tabelas e a escolha de cada tipo de dado

```sql
USE projeto_dommy;
GO

-- Tabela DIMENSAO: cadastro de produtos (snapshot de 31/05/2026)
CREATE TABLE dim_produtos (
    cod_ean             VARCHAR(20)     PRIMARY KEY,
    des_produto         VARCHAR(200)    NOT NULL,
    val_custo           DECIMAL(10,2),
    val_venda           DECIMAL(10,2),
    qtd_estoque_atual   DECIMAL(12,2)
);

-- Tabela FATO: vendas detalhadas (historico empilhado)
CREATE TABLE fato_vendas (
    id_venda_linha  INT IDENTITY(1,1)   PRIMARY KEY,
    dta_venda       DATE                NOT NULL,
    cod_ean         VARCHAR(20)         NOT NULL,
    cod_venda       BIGINT              NOT NULL,
    qtd_item        DECIMAL(10,3)       NOT NULL,
    val_unitario    DECIMAL(10,2)       NOT NULL
);

-- Indice no cod_ean da fato pra acelerar o JOIN com a dimensao
CREATE INDEX idx_fato_vendas_cod_ean ON fato_vendas(cod_ean);
```

Cada tipo acima foi uma escolha consciente. Explicando **por grupo de decisão**, já que colunas parecidas seguem a mesma lógica:

**Os identificadores, texto e chave artificial.** O `cod_ean` é `VARCHAR(20)` (texto) pela mesma razão do Python: código de barras é identificador, não número, texto preserva zeros à esquerda e evita notação científica. Como cada produto tem EAN único, ele é a **`PRIMARY KEY`** da dimensão. Na tabela fato, porém, não existe coluna naturalmente única (o mesmo produto se repete em dias diferentes), então criei uma **chave artificial**: o `id_venda_linha` é um `INT IDENTITY(1,1)`, um número que o próprio SQL Server gera e incrementa sozinho a cada linha. Já o `cod_venda` é `BIGINT` (e não `INT`) porque o número do cupom é grande e estouraria um `INT` comum.

**O dinheiro, `DECIMAL`, nunca `FLOAT`.** Essa é a decisão mais importante da modelagem. Todos os valores monetários são `DECIMAL(10,2)`, e isso é deliberado: o `FLOAT` guarda número de forma **aproximada**, gerando erros de arredondamento que, somados em milhares de linhas, fazem o faturamento **não fechar**. O `DECIMAL` é **exato**. O `(10,2)` cobre valores até R$ 99.999.999,99 com precisão de centavo.

**As quantidades, uma casa decimal a mais.** O `qtd_item` é `DECIMAL(10,3)`, com **3 casas** em vez das 2 do dinheiro, porque venda a peso ou granel aparece com fração (`1,250`). As 3 casas evitam perder essa precisão.

**As datas, `DATE`, não `DATETIME`.** A `dta_venda` guarda só a data, sem hora. As vendas são analisadas por dia, e a hora exata não agregava nada, quando o tipo mais simples resolve, fico com ele.

**As restrições, `NOT NULL` como contrato.** Uma venda *tem* que ter data, produto, quantidade e valor; com `NOT NULL`, o banco recusa qualquer registro incompleto. Já `val_custo` e `val_venda` na dimensão **não** têm `NOT NULL` de propósito, pode existir produto sem custo, e prefiro permitir isso para tratar a ausência de forma controlada na view.

**O índice, acelerando o cruzamento.** A fato tem ~94 mil linhas e é cruzada com a dimensão pelo `cod_ean` o tempo todo. O índice funciona como o índice remissivo de um livro: em vez de varrer tudo, o banco vai direto onde o código está, a diferença entre um dashboard ágil e um travado.

---

### 4.2 As Views — regras de negócio no banco

Aqui entra um conceito que considero o coração da modelagem: a **view**. Uma view é uma "tabela virtual", ela **não armazena dado nenhum**, apenas guarda uma consulta pronta que é executada toda vez que alguém a acessa. Pense nela como uma "consulta salva com nome".

Por que isso importa? Porque eu poderia ter deixado o Power BI fazer todos os cálculos (cruzar tabelas, calcular lucro, classificar margem). Mas escolhi **centralizar a regra de negócio no banco**, por três motivos:

1. **Uma fonte única da verdade.** A regra de "como se calcula o lucro" mora em um lugar só. Se amanhã eu conectar outra ferramenta no banco, ela enxerga o mesmo cálculo — não preciso reescrever a lógica em cada lugar.
2. **O Power BI recebe o dado pronto.** Em vez de o relatório ter que cruzar e calcular tudo, ele já consome um dado limpo, relacionado e com as métricas prontas. A camada de visualização fica mais leve.
3. **Manutenção num ponto só.** Se a regra mudar, eu altero a view — e todo mundo que a consome passa a usar a nova regra automaticamente.

Criei duas views, cada uma com um papel.

#### `vw_vendas_completa` — juntando o fato com a dimensão

Esta é a view principal: ela faz o **JOIN** entre as vendas e o cadastro de produtos, e já calcula as métricas de cada linha.

```sql
ALTER VIEW [dbo].[vw_vendas_completa] AS
SELECT
    v.id_venda_linha,
    v.dta_venda,
    v.cod_venda,
    v.cod_ean,
    COALESCE(p.des_produto, 'Não Cadastrado')           AS descricao_produto,
    v.qtd_item,
    v.val_unitario,
    p.val_custo                                         AS custo_unitario,
    -- Métricas calculadas por linha (somáveis)
    v.qtd_item * v.val_unitario                         AS faturamento,
    v.qtd_item * p.val_custo                            AS custo_total,
    v.qtd_item * (v.val_unitario - p.val_custo)         AS lucro_bruto,
    -- Flag para identificar produtos sem cadastro de custo
    CASE 
        WHEN p.val_custo IS NULL THEN 0 
        ELSE 1 
    END                                                 AS tem_custo
FROM fato_vendas v
LEFT JOIN dim_produtos p
    ON v.cod_ean = p.cod_ean;
GO
```

**Por que `LEFT JOIN` e não `INNER JOIN`**, essa foi uma decisão importante. O `INNER JOIN` só traria as vendas cujo produto existe no cadastro; qualquer venda de um produto não cadastrado **sumiria** do resultado. Eu não quero isso: faturamento é faturamento, mesmo que o produto não tenha custo registrado. O `LEFT JOIN` garante que **todas as vendas permanecem**, quando não há produto correspondente no cadastro, os campos de custo simplesmente ficam nulos, mas a venda continua lá, contando no faturamento. Perder venda em análise financeira seria um erro grave; o `LEFT JOIN` evita isso.

**As métricas calculadas na própria view.** Em vez de guardar `faturamento` como coluna no banco (lembra da regra: não armazenar o que se pode calcular), eu calculo na hora: `faturamento = qtd_item × val_unitario`, `lucro_bruto = qtd_item × (val_unitario − val_custo)`. Essas métricas são **somáveis**, dá para empilhá-las por mês, por produto, por dia da semana, e elas sempre somam certo. Entregar isso pronto facilita muito o trabalho no Power BI.

**O `COALESCE` para o nome.** Como o `LEFT JOIN` deixa o nome nulo quando o produto não está cadastrado, uso `COALESCE(p.des_produto, 'Não Cadastrado')` para trocar esse nulo por um texto legível. Assim, no dashboard, em vez de um campo em branco, aparece "Não Cadastrado", o usuário entende que existe a venda, só falta o cadastro.

**A flag `tem_custo`.** Criei uma marcação simples (1 ou 0) que sinaliza se aquela linha tem custo cadastrado. Isso me permite, na análise, separar facilmente o que tem informação de margem confiável do que não tem, uma forma de manter transparência sobre a qualidade do dado.

#### `vw_dim_produtos` — enriquecendo o cadastro com margem

A segunda view pega o cadastro de produtos e adiciona o cálculo de **margem** e a classificação em **faixas**, prontas para a análise de rentabilidade.

```sql
ALTER VIEW [dbo].[vw_dim_produtos] AS
SELECT
    cod_ean,
    des_produto,
    val_custo,
    val_venda,
    qtd_estoque_atual,
    -- margem calculada (fração: 0.42 = 42%)
    CASE 
        WHEN val_venda > 0 
        THEN (val_venda - val_custo) / val_venda 
        ELSE NULL 
    END AS margem_pct,
    -- faixa de margem (Prejuízo + 0-10% juntos em "Abaixo de 10%")
    CASE
        WHEN val_venda <= 0                             THEN NULL
        WHEN (val_venda - val_custo) / val_venda < 0.10 THEN 'Abaixo de 10%'
        WHEN (val_venda - val_custo) / val_venda < 0.20 THEN '10-20%'
        WHEN (val_venda - val_custo) / val_venda < 0.30 THEN '20-30%'
        WHEN (val_venda - val_custo) / val_venda < 0.40 THEN '30-40%'
        ELSE '40%+'
    END AS faixa_margem,
    -- ordem da faixa
    CASE
        WHEN val_venda <= 0                             THEN 99
        WHEN (val_venda - val_custo) / val_venda < 0.10 THEN 1
        WHEN (val_venda - val_custo) / val_venda < 0.20 THEN 2
        WHEN (val_venda - val_custo) / val_venda < 0.30 THEN 3
        WHEN (val_venda - val_custo) / val_venda < 0.40 THEN 4
        ELSE 5
    END AS ordem_faixa
FROM dim_produtos;
GO
```

**A proteção contra divisão por zero.** O cálculo da margem é `(val_venda − val_custo) / val_venda`. Se algum produto tiver `val_venda` igual a zero (ou negativo, por erro de cadastro), essa divisão quebraria. Por isso cada `CASE` começa testando `WHEN val_venda <= 0 THEN NULL`, se o preço de venda não for válido, a margem vira nula em vez de causar erro. É o mesmo princípio do `errors='coerce'` lá no Python: falhar com elegância, nunca quebrar.

**Por que calcular a margem na view, e não no Python.** A margem e suas faixas são uma **regra de análise**, e regras de análise mudam. As faixas (Abaixo de 10%, 10-20%, etc.) foram definidas para esse estudo de rentabilidade e podem ser ajustadas a qualquer momento. Mantendo esse cálculo na view, eu mudo a regra em um lugar só, sem precisar reprocessar todo o ETL do zero. Deixo o Python para o que é tratamento bruto e estável, e a view para o que é regra de negócio flexível.

**A coluna de ordenação (`ordem_faixa`).** Repare que, além do texto da faixa, criei um número de ordem (1 a 5, e 99 para os inválidos). Isso existe por um detalhe prático de visualização: se eu deixasse o Power BI ordenar as faixas pelo texto, elas sairiam em ordem alfabética ("10-20%" viria antes de "Abaixo de 10%"), o que não faz sentido. Com a coluna numérica, eu garanto que as faixas apareçam na ordem lógica de rentabilidade, da pior para a melhor.

---

## 5. Do Dado à Decisão — o Dashboard

Toda a engenharia das camadas anteriores existe para servir a esta: o dashboard é onde o dado vira **decisão**. As tabelas tratadas e relacionadas no Power BI alimentam três páginas, cada uma respondendo a uma pergunta de negócio diferente.

Antes de passar por elas, vale entender **como os principais números são construídos**, porque saber o que cada indicador realmente mede é o que separa "olhar um gráfico bonito" de "tomar uma decisão informada".

### 5.1 Como os indicadores são construídos

**Faturamento.** É a soma, item a item, de *quantidade × preço efetivamente praticado na venda*. Ponto importante: o faturamento usa o **preço real de cada venda histórica**, não o preço de tabela atual. É o dinheiro que de fato entrou no caixa, no momento em que entrou.

**Custo Total.** É a soma de *quantidade × custo do produto*. Aqui há uma premissa que vale registrar com transparência: o custo vem do **cadastro atual** (uma "foto" tirada em 31/05/2026), e não do custo histórico de cada data, porque o sistema só guarda o custo vigente, não o da época de cada venda. Na prática, é uma aproximação muito boa, mas é honesto dizer que o custo é o atual aplicado sobre as vendas passadas.

**Lucro Bruto.** É simplesmente *faturamento − custo*. O termo "bruto" é proposital e importante: este é o lucro **sobre a mercadoria**, antes de descontar despesas operacionais (aluguel, salários, energia, impostos). Ele responde "quanto sobra da diferença entre o que vendi e o que paguei pela mercadoria", não é o lucro líquido final da loja. Chamá-lo de "bruto" evita prometer um número que ele não representa.

**O detalhe do `tem_custo`, e por que ele protege a margem.** Nem todo item vendido tem custo cadastrado: cerca de 0,4% das vendas são de produtos sem custo conhecido (entradas antigas, cadastros incompletos). Isso cria um problema sutil no cálculo da **margem**: se eu dividisse o lucro pelo faturamento *total*, estaria incluindo no denominador vendas cujo lucro é impossível calcular, e a margem apareceria menor do que realmente é, artificialmente puxada para baixo.

A solução foi marcar cada venda com uma sinalização (`tem_custo`): vale 1 quando o produto tem custo cadastrado, 0 quando não tem. No cálculo da margem, considero **apenas as vendas com custo conhecido**. Assim, a margem reflete a rentabilidade real dos produtos rastreáveis, sem distorção. O **faturamento total**, por outro lado, continua somando tudo, porque dinheiro que entrou é dinheiro que entrou, com ou sem custo cadastrado. Em resumo: faturamento conta tudo; margem só conta o que dá para calcular com honestidade.

**Transação ≠ unidade.** Por fim, uma distinção que aparece em vários pontos do dashboard. O **número de vendas** conta *cupons* (transações), quantas compras aconteceram, e não a quantidade de itens. Uma compra com cinco produtos é uma venda, não cinco. Essa diferença importa: para medir o movimento da loja (quantos clientes, qual o ritmo), o número de transações é o certo; para medir giro de produto (quanto saiu de cada item), aí sim se conta unidades. Cada análise usa a métrica adequada à pergunta.

### 5.2 Página 1 — Painel Executivo

![Página 1 — Painel Executivo](Dashboards/Dashboard_Visao_Geral.png)

**Objetivo.** É a página de abertura, pensada para responder uma única pergunta em cinco segundos: *como o negócio está indo?* É o que a diretoria olha primeiro, antes de qualquer detalhe.

**O que ela mostra.** Cinco indicadores-chave no topo, numa sequência que conta uma história, Faturamento (quanto entrou), Lucro Bruto (quanto sobrou na mercadoria), Margem (a eficiência por real vendido), Ticket Médio (quanto cada cliente gasta por compra) e Número de Vendas (quantas transações). Abaixo, a evolução mês a mês, os produtos que mais faturam e mais vendem, e o comportamento por dia da semana. Cada KPI traz a comparação com o mês anterior, sinalizando em verde ou vermelho se melhorou ou piorou.

**O que conseguimos traduzir dos dados:**

- **O tamanho do negócio.** Ao longo dos 9 meses analisados (setembro/2025 a maio/2026), a loja faturou **R$ 1,36 milhão**, o que dá uma média em torno de **R$ 151 mil por mês**. Esse é o ponto de partida: saber a ordem de grandeza do negócio e ter uma régua para comparar cada mês.

- **Tendência temporal, março é o ponto forte da loja.** O faturamento não é estável ao longo dos meses, e **março é, com folga, o mês que mais vende**, efeito direto da **Páscoa**, a data mais importante para uma loja de doces. Isso fica evidente no gráfico e é uma informação estratégica de primeira ordem: o negócio tem um pico sazonal claro, e prepará-lo bem (estoque, variedade, fluxo de caixa) é o que garante aproveitar o melhor momento do ano. Vale, inclusive, olhar com lupa **o que mais vende nesse período** para dar foco. A mesma lógica se aplica às outras datas que sempre movimentam o setor de doces, **Dia das Crianças, Natal, Dia dos Namorados**, que merecem planejamento antecipado e uma análise específica do que gira em cada uma.

- **O que mais fatura não é o que mais sai.** Esse é um dos cruzamentos mais reveladores da página. O **Chantilly Norcau** é o **maior produto em faturamento** (cerca de R$ 30,8 mil no período), mas quando olhamos o ranking de **volume** (unidades vendidas), quem lidera é o **"Produtos Diversos"**, com mais de 1.600 unidades. Ou seja: o produto que traz mais dinheiro não é o que mais sai da prateleira. Isso acontece porque produtos de **maior valor unitário** faturam alto mesmo vendendo menos, enquanto itens **baratos e populares** vendem muito mas pesam pouco na receita. Ter os dois rankings lado a lado deixa essa distinção explícita, e evita a armadilha de gerir a loja olhando só um dos ângulos.

- **O padrão da semana, e a explicação do sábado.** Olhando o faturamento por dia da semana, **sexta-feira é o dia mais forte** (cerca de 19% do faturamento da semana) e **sábado é o mais fraco** (cerca de 14%). À primeira vista parece contraintuitivo para uma loja de doces, mas a explicação é simples e operacional: **a loja fecha às 15h no sábado**, enquanto de segunda a sexta funciona até as 18h. Menos horas de operação, menos vendas, o número faz total sentido quando se conhece a rotina da loja. A leitura prática: o movimento se concentra de segunda a sexta, com pico na sexta, e é nesses dias que estoque e equipe precisam estar mais reforçados.

- **A leitura cruzada de ticket e volume.** Comparar o ticket médio com o número de vendas conta histórias que nenhum dos dois sozinho conta. Se num mês o faturamento cai mas a margem melhora e o ticket sobe, a leitura é "vendi para menos clientes, mas cada um comprou mais e melhor", um cenário muito diferente de "perdi clientes e tive que dar desconto". É essa leitura combinada que transforma números em narrativa de negócio.

### 5.3 Página 2 — Curva ABC

![Página 2 — Curva ABC](Dashboards/Dashboard_Visao_Produtos.png)

**Objetivo.** Esta página responde a uma pergunta que todo gestor de varejo precisa fazer mas raramente mede: *quais produtos realmente sustentam o faturamento?* Ela aplica o **princípio de Pareto** (a regra do 80/20) para separar o essencial do acessório.

**O que ela mostra.** Os produtos são ordenados do que mais fatura para o que menos fatura, e classificados em três grupos: **A** (os que concentram a maior parte do faturamento), **B** (os intermediários) e **C** (a cauda de produtos que pesam menos). A Curva ABC aqui tem um objetivo prático: focar nos produtos que realmente movem o faturamento, em vez de diluir a análise entre os milhares de itens do catálogo. Por isso o recorte são os **30 maiores produtos**, e os percentuais desta página são calculados **dentro desse grupo**, não sobre o faturamento total da loja. É uma lupa sobre os campeões.

**O que conseguimos traduzir dos dados:**

- **Mesmo entre os campeões, ninguém domina.** Dentro do Top 30, o **Grupo A** reúne os **20 produtos** que concentram cerca de 80% do faturamento desse recorte (R$ 240,9 mil), seguido do **Grupo B** com 7 produtos (R$ 43,7 mil, 14,52%) e do **Grupo C** com 3 (R$ 16,6 mil, 5,50%). O maior de todos, o Chantilly Norcau, pesa cerca de **10% dentro do Top 30**, o que, olhando a loja **inteira**, equivale a apenas **2,26% do faturamento total**. Nem mesmo o líder absoluto chega perto de "carregar" o negócio sozinho.

- **A força está no catálogo amplo (cauda longa).** Se o produto nº 1 vale só 2,26% do total da loja, fica claro que o faturamento da Domy não vem de dois ou três campeões, e sim de **muitos produtos contribuindo um pouco cada**, um catálogo extenso e bem distribuído. Isso tem duas leituras: de um lado é resiliência (a loja não quebra se um item sumir); de outro, é um **desafio de gestão de mix**, porque administrar bem centenas de itens exige controle de estoque e curadoria constante. A Curva ABC serve justamente para **priorizar os campeões** (nunca deixar faltar um produto do Grupo A) e, na cauda, decidir o que vale manter por conveniência de sortimento e o que pode ser enxugado para liberar capital e espaço.

### 5.4 Página 3 — Análise de Produtos

![Página 3 — Análise de Produtos](Dashboards/Dashboard_Visao_Margem.png)

**Objetivo.** Se a página 2 olha *quanto* cada produto fatura, esta olha *quão rentável* ele é. O objetivo é entender a **saúde da margem** do portfólio, onde está o lucro e onde estão os riscos.

**O que ela mostra.** Os produtos são distribuídos em faixas de margem (de "abaixo de 10%" até "40% ou mais"), com a contagem de produtos e o peso de cada faixa no faturamento. Há ainda um gráfico que cruza **margem versus giro** (rentabilidade contra velocidade de venda), criando uma matriz de decisão.

**O que conseguimos traduzir dos dados:**

- **A rentabilidade é saudável.** A maior parte do faturamento, cerca de **51%**, vem de produtos com margem entre **20% e 30%**, e a margem mediana do portfólio gira em torno de **31%**, um patamar saudável para o varejo de doces. A leitura tranquilizadora: o negócio não depende de produtos de margem apertada; a base é sólida.

- **Os pontos de atenção estão isolados e visíveis.** Pouquíssimos produtos caem na faixa crítica ("abaixo de 10%" de margem), destacada em vermelho justamente para saltar aos olhos. São poucos, mas é onde a gestão deve olhar: produto vendido com margem muito baixa ou negativa geralmente indica **custo desatualizado no cadastro** ou **preço mal calibrado**. Identificá-los é o primeiro passo para corrigir precificação.

- **A matriz margem × giro é a ferramenta de decisão mais rica.** Cruzar rentabilidade com velocidade de venda separa os produtos em quatro perfis acionáveis. Analisando os 60 maiores produtos, eles se distribuíram assim: os **Ideais** (boa margem + giro rápido, proteger e priorizar), os de **Volume** (giram muito mas com margem apertada, candidatos a reajuste de preço), as **Joias** (boa margem mas giro lento, merecem mais exposição e divulgação) e os **Problemas** (margem baixa e giro lento, candidatos a renegociação com fornecedor ou descontinuação). É a página que transforma "temos 5 mil produtos" em "aqui está o que fazer com cada grupo".

- **O insight mais valioso: o campeão de vendas tem margem apertada.** Cruzando os dados, aparece um achado de negócio forte, o **Chantilly Norcau**, maior produto em faturamento da loja, gira rapidíssimo (vende a cada ~1,4 dias) mas opera com margem realizada em torno de **22%**, abaixo do ideal. A implicação é direta e poderosa: como ele vende **muito**, um pequeno ajuste de margem (subir de 22% para, digamos, 26%) teria um **impacto desproporcional no lucro total**, justamente pelo volume. Esse é o tipo de recomendação que sai do "achei um número interessante" e vira "olha exatamente onde mexer para ganhar dinheiro", o produto certo, na direção certa, com o maior efeito alavancado.

---

## 6. Principais Insights e Achados de Governança

Um projeto de dados tem dois tipos de entrega. O primeiro é o óbvio: os **insights de negócio** que o dashboard revela. O segundo é menos visível mas igualmente valioso: os **achados de qualidade e governança** que aparecem no caminho, os problemas nos dados que, se não fossem percebidos, levariam a conclusões erradas. Esta seção cobre os dois.

### 6.1 O que os dados revelaram sobre o negócio

Consolidando as leituras das três páginas, os principais insights de negócio foram:

- **A loja tem um pico sazonal claro em março**, puxado pela Páscoa, a data mais importante para o setor. Saber disso permite planejar estoque e caixa para o melhor momento do ano, e olhar com atenção as outras datas que movimentam doces (Dia das Crianças, Natal, Dia dos Namorados).

- **O movimento se concentra de segunda a sexta, com pico na sexta**, e cai no sábado, explicado pelo horário reduzido (fecha às 15h, contra 18h nos dias úteis). Isso orienta a alocação de estoque e equipe.

- **O produto que mais fatura não é o que mais vende em volume.** O Chantilly Norcau lidera em receita, mas itens populares e baratos lideram em unidades. Gerir a loja exige olhar os dois ângulos.

- **A loja não depende de nenhum produto isolado**, o maior deles representa apenas ~2,26% do faturamento. É um negócio de catálogo amplo, com baixo risco de concentração, mas que exige boa gestão de mix.

- **A rentabilidade geral é saudável** (margem mediana ~31%), mas os **campeões de venda operam com margem apertada**, e é justamente neles, pelo volume, que pequenos ajustes de margem teriam o maior impacto no lucro.

### 6.2 Achados de governança e qualidade de dados

Aqui estão os achados que considero os mais importantes do projeto, não porque apareçam no dashboard, mas porque mostram o trabalho de **questionar o dado antes de confiar nele**. Cada um deles, se ignorado, teria comprometido a análise inteira.

#### O erro do código de barras truncado — o achado mais crítico

Este é o achado de que mais me orgulho, porque exigiu desconfiar de um resultado que "funcionava". Na primeira tentativa de cruzar as vendas com o cadastro de produtos, usei uma planilha de cadastro cujo código de barras (EAN) vinha **truncado em 10 dígitos**, em vez dos 13 completos. O cruzamento até rodava, mas os números de margem saíam estranhos, com **centenas de produtos aparentemente no prejuízo**.

Em vez de aceitar, fui investigar. E descobri o problema: ao cortar o EAN em 10 dígitos, **dois produtos diferentes podiam acabar com o mesmo código truncado**. O cruzamento, então, casava a venda de um produto com o custo de **outro produto**, comparando o preço de venda de um item com o custo de um item completamente diferente. O resultado eram margens absurdas, negativas, que não refletiam a realidade. Pior: esse cruzamento só casava corretamente uma fração pequena das vendas, e silenciosamente errava o resto.

A solução foi trocar a fonte: passei a usar uma planilha de cadastro com o **EAN completo de 13 dígitos**, que permite um cruzamento exato. A cobertura subiu para mais de **99% das vendas**, e o quadro de rentabilidade se corrigiu: em vez de centenas de produtos no prejuízo, revelou-se que **pouquíssimos** realmente operavam com margem negativa, e que a margem mediana da loja era saudável (~31%).

A lição que esse achado carrega é a essência da governança de dados: **um cruzamento que "funciona" não é necessariamente um cruzamento correto.** Um identificador precisa ser íntegro e único, ou ele conecta as coisas erradas, e o erro se esconde atrás de números que parecem plausíveis. Foi por desconfiar dos números que o problema apareceu.

#### O custo desatualizado e as duas margens

Um segundo achado importante diz respeito à natureza do custo. O custo de cada produto vem do **cadastro atual** do sistema (uma foto de 31/05/2026), não do custo histórico vigente na época de cada venda, porque o ERP simplesmente não guarda o custo de cada data passada. Isso cria uma consequência que precisei entender e documentar: existem, na prática, **duas margens diferentes**, e elas não batem de propósito.

- A **margem de cadastro** usa o preço e o custo atuais. Responde "como esse produto está precificado hoje?".
- A **margem realizada** usa o preço pelo qual cada venda *de fato* aconteceu. Responde "quanto eu realmente lucrei vendendo isso?".

Elas divergem porque **o preço de venda variou ao longo do tempo**. Investigando o Chantilly, por exemplo, encontrei que ele foi vendido a quatro preços diferentes ao longo dos meses, então a margem realizada (que inclui as vendas mais baratas do passado) fica naturalmente abaixo da margem de cadastro (que só enxerga o preço atual). Nenhuma das duas está "errada"; elas respondem a perguntas diferentes, e a governança está em saber **qual usar em cada contexto** (a de cadastro para classificar a política de preço atual; a realizada para medir o lucro efetivo). Documentar essa distinção evita que alguém olhe os dois números e ache que há um erro de cálculo.

Como sinal de alerta prático, isso também significa que **produtos com margem muito baixa ou negativa no cadastro frequentemente apontam para um custo desatualizado** no ERP, o que transforma um problema de dado em uma ação de negócio concreta: revisar o cadastro desses itens.

#### Os produtos vendidos abaixo do custo

A análise de margem revelou **9 produtos com margem negativa**, ou seja, vendidos por um valor menor do que custaram. Mas margem negativa não é automaticamente um erro: ela pode significar três coisas diferentes, e cada uma pede uma ação distinta.

Pode ser uma **decisão consciente**, com a loja vendendo abaixo do custo de propósito para dar vazão a um produto encalhado ou perto do vencimento, recuperando parte do dinheiro em vez de perder o item inteiro. Pode ser um **alerta real**, um produto sendo vendido no prejuízo sem que ninguém tenha percebido, que precisa de reajuste de preço urgente. Ou pode ser um **erro de cadastro**, com o custo digitado errado no ERP, fazendo um produto saudável apenas *parecer* deficitário.

O valor da análise não é apontar "está errado", e sim **sinalizar esses casos** para que a loja investigue um a um e descubra em qual das três situações cada produto se encaixa. É um exemplo claro de como o dado, bem tratado, vira uma lista de ações concretas em vez de um número solto.

#### O estoque que ficou de fora

Nem todo dado disponível merece entrar na análise. A planilha de cadastro trazia uma coluna de **saldo de estoque**, mas, ao conversar com o dono da loja, confirmei que esse número **não era confiável** no sistema. A decisão foi deliberada: tratei e carreguei o dado (ele existe no banco), mas o **excluí conscientemente das análises**, registrando essa limitação. Usar um número que se sabe estar errado é pior do que não usá-lo, e parte da maturidade em governança é ter a disciplina de deixar dados ruins de fora, com transparência sobre o porquê.

#### As vendas sem cadastro

Por fim, uma pequena fração das vendas (cerca de 0,4%) era de produtos que **não tinham correspondência no cadastro de custo**, entradas antigas ou cadastros incompletos. Em vez de descartar essas vendas (o que distorceria o faturamento) ou de fingir que elas tinham custo (o que distorceria a margem), tratei o caso explicitamente: essas vendas **permanecem no faturamento** (via o `LEFT JOIN` da view), aparecem identificadas como "Não Cadastrado", e ficam **fora do cálculo de margem** (via a flag `tem_custo`). Cada venda contabilizada onde deve, sem inventar dado que não existe.

### 6.3 Conclusão

Este projeto percorreu o ciclo completo de um dado: da planilha bruta exportada de um ERP até um dashboard que responde perguntas de negócio. No caminho, passou por **extração e transformação em Python** (com o cuidado de tratar cada tipo de dado corretamente), **modelagem dimensional em SQL Server** (com tabelas tipadas e views que centralizam a regra de negócio) e **visualização em Power BI** (com métricas confiáveis e leitura orientada à decisão).

Mas talvez o que melhor resuma o trabalho não seja a stack técnica, e sim a **postura**: a de questionar o dado antes de confiar nele. Foi essa postura que revelou o erro do EAN truncado, que distinguiu as duas margens, que deixou o estoque de fora e que tratou as vendas sem cadastro com honestidade. Ferramentas se aprendem; o que distingue uma análise confiável de uma análise apenas bonita é o rigor em garantir que cada número signifique exatamente o que diz significar.

No fim, o objetivo nunca foi "fazer gráficos". Foi transformar dados que estavam presos e silenciosos em **informação capaz de orientar decisões reais** de um negócio real, e fazê-lo de um jeito que qualquer pessoa, técnica ou não, possa confiar no resultado.

---

<p align="center">
  <strong>Domy Doces, Plataforma de Análise de Vendas</strong><br>
  <em>Python · SQL Server · Power BI</em><br>
  Desenvolvido por Vinícius Braga Bruno
</p>
