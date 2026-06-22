# COFIEX BI — Análise de Financiamentos Externos do Brasil

Um projeto de análise exploratória e processamento de dados sobre financiamentos externos gerenciados pela COFIEX (Comissão para Assuntos Financeiros Externos). Com um ETL robusto em Python/Jupyter, este repositório preserva a integridade dos 3.548 registros originais enquanto fornece indicadores derivados, flags de qualidade e um dashboard interativo para responder dez questões estratégicas sobre a carteira de financiamentos brasileira.

## 📊 Dashboard Interativo

O projeto inclui um dashboard HTML profissional com gráficos dinâmicos, filtros e análise de padrões:

- **Evolução temporal** dos financiamentos de 2000 a 2024
- **Distribuição geográfica** por região e UF
- **Composição da carteira** por fonte financiadora e fase
- **Análise financeira** com métricas de contrapartida
- **Dez questões-chave** respondidas por dados

[Abrir Dashboard →](dashboard_cofiex.html)

## 🎯 Objetivo e Contexto

A COFIEX gerencia financiamentos externos que apoiam projetos brasileiros em infraestrutura, educação, saúde e outros setores. Este projeto responde a pergunta central:

> _Como a carteira de financiamentos externos da COFIEX revela prioridades, concentração territorial e dependência de fontes financiadoras internacionais ao longo do tempo?_

## 📈 Perguntas Principais

1. **Como evoluiu** o volume financeiro e quantidade de pleitos ao longo do tempo?
2. **Quais regiões** e UFs concentram mais pleitos e maior valor financiado?
3. **Qual é a distribuição** entre esferas federal, estadual e municipal?
4. **Quais fontes** têm maior participação na carteira da COFIEX?
5. **Quais tipos** de operação predominam no portfólio?
6. **Em quais fases** os pleitos se concentram?
7. **Qual é a relação** entre financiamento e contrapartida?
8. **Há concentração** em poucas fontes, regiões ou proponentes?
9. **Quais mudanças** de perfil aparecem em períodos distintos?
10. **Quais limitações** e inconsistências afetam a leitura dos dados?

## 📁 Estrutura do Projeto

```
CofiexBI/
├── README.md                                # Este arquivo
├── logo_cofiex.svg                          # Logo do projeto
├── dashboard_cofiex.html                    # Dashboard interativo
├── AGENTS.md                                # Documentação técnica completa
├── dados_brutos/
│   └── dados_2026-04-14.xlsx               # Arquivo original COFIEX (3.548 linhas)
├── dados_tratados/
│   ├── cofiex_tratado.csv                  # Base processada com flags de qualidade
│   └── relatorio_qualidade_colunas.csv     # Relatório de integridade por coluna
└── notebooks/
    ├── extracao_diagnostico.ipynb          # Diagnóstico inicial da base
    ├── limpeza_transformacao.ipynb         # ETL: limpeza e transformação
    └── validacao_carga.ipynb               # Validação final e flags de qualidade
```

## 🔧 Como Rodar o Projeto

### Pré-requisitos

- Python 3.8+
- Jupyter Notebook
- Bibliotecas: `pandas`, `numpy`, `openpyxl`

### Instalação

```bash
# Clone o repositório
git clone https://github.com/devheron/CofiexBI.git
cd CofiexBI

# Crie um ambiente virtual (recomendado)
python -m venv venv
source venv/bin/activate  # Linux/Mac
# ou
venv\Scripts\activate     # Windows

# Instale as dependências
pip install pandas numpy openpyxl jupyter matplotlib seaborn
```

### Executar os Notebooks

```bash
# Inicie o Jupyter
jupyter notebook

# Abra e execute os notebooks nesta ordem:
# 1. notebooks/extracao_diagnostico.ipynb
# 2. notebooks/limpeza_transformacao.ipynb
# 3. notebooks/validacao_carga.ipynb
```

### Visualizar o Dashboard

Abra `dashboard_cofiex.html` em seu navegador para explorar os gráficos interativos.

## 📊 Resultados Principais

### Métricas Gerais

| Métrica               | Valor                    |
| --------------------- | ------------------------ |
| Registros Totais      | 3.548                    |
| Pleitos Únicos        | 3.206                    |
| Pleitos com Repetição | 158 códigos (342 linhas) |
| Financiamento Total   | ~USD 238 bilhões         |
| Fontes Financiadoras  | 28 instituições ativas   |
| Período Coberto       | 2000-2024                |

### Distribuição Geográfica

Os financiamentos concentram-se principalmente no **Sudeste** (41% dos registros), seguido por **Nordeste** (28%) e **Sul** (18%). A esfera **federal** representa 60% da carteira, refletindo a natureza dos projetos estruturantes financiados externamente.

### Composição da Carteira

- **Por Tipo de Operação**: Financiamento (52%), Garantia (26%), Cofinanciamento (13%), Linha de Crédito (9%)
- **Por Fase**: Assinado (48%), Em Desembolso (32%), Encerrado (13%), Em Negociação (7%)
- **Por Moeda Principal**: Dólar (73%), Euro (15%), Real (8%), Outras (4%)

### Análise de Qualidade

A base foi submetida a validação em sete dimensões:

1. **Duplicação**: 158 cd_pleito repetidos identificados e flagados
2. **Datas**: 7 registros com datas extremas, 6 com sentinelas, 382 com placeholders
3. **Valores Financeiros**: 312 registros sem financiamento, 338 com contrapartida > financiamento
4. **Cronologia**: 88 registros com inconsistência na sequência temporal
5. **Outliers**: 47 registros com valores acima do percentil 99
6. **Nulos Auditáveis**: 283-289 registros com categorias críticas não preenchidas
7. **Integridade**: 0 linhas removidas, 3.548 preservadas com flags documentadas

## 🧹 Processo ETL

### Etapa 1: Extração e Diagnóstico

Leitura do arquivo `dados_2026-04-14.xlsx`, inventário de 25 colunas e identificação de:

- Padrões de nulos por coluna
- Distribuição de valores categóricos
- Primeira visão de duplicações e inconsistências
- Tipos de dados e conversões necessárias

### Etapa 2: Limpeza e Transformação

**Padronização de Datas:**

- Conversão de Excel serial numbers para datetime
- Tratamento de 382 placeholders (1940-01-01) como NaT
- Sinalização de 6 datas sentinelas (9999-12-31, etc.)

**Colunas Derivadas:**

- `ano_recebimento`, `ano_primeira_cofiex`, `ano_assinatura`
- `valor_total_dolar = financiamento + contrapartida`
- `percentual_contrapartida` (fração 0-1)
- Prazos em dias: `prazo_recebimento_cofiex`, `prazo_cofiex_assinatura`, `prazo_assinatura_desembolso`

**Padronização Categórica:**

- Remoção de espaços, normalização de case
- Categorias não informadas preenchidas como "Não informado" (auditável)
- 7 categorias de moeda identificadas e padronizadas

**Flags de Qualidade:**

- `flag_cd_pleito_repetido` (342 registros)
- `flag_financiamento_nulo_zero` (312 registros)
- `flag_contrapartida_maior_financiamento` (338 registros)
- `flag_data_extrema`, `flag_data_sentinela`, `flag_recebimento_placeholder`
- `flag_cronologia_invalida`, `flag_outlier_financeiro`

### Etapa 3: Validação e Carga

Verificação de:

- Consistência cronológica (datas em ordem lógica)
- Valores negativos ou zerados
- Contagem de registros antes e depois (3.548 = 3.548 ✓)
- Releitura e reteste do CSV tratado

**Saídas Geradas:**

- `cofiex_tratado.csv` — base completa, 25 colunas originais + 14 derivadas + 9 flags
- `relatorio_qualidade_colunas.csv` — tipo, nulos, percentual de nulos e unicos por coluna

## 📚 Dicionário de Colunas Principais

### Identificadores

- `cd_pleito` — Código único do pleito (158 códigos repetidos em 342 linhas)
- `nu_operacao` — Número da operação financeira (16 códigos repetidos)

### Geográficos

- `nm_regiao` — Nome da região (Norte, Nordeste, Sudeste, Sul, Centro-Oeste)
- `sg_uf` — Sigla da UF (27 valores)
- `nm_uf` — Nome completo da UF
- `de_esfera` — Esfera de governo (Federal, Estadual, Municipal)

### Financeiros

- `vl_financiamento_dolar` — Valor em dólar ou equivalente (USD)
- `vl_contrapartida_dolar` — Valor de contrapartida em dólar
- `nm_moeda` — Moeda original (Dólar, Euro, Real, SDR, Iene, Libra)
- `valor_total_dolar` — Soma de financiamento + contrapartida (derivada)
- `percentual_contrapartida` — Razão contrapartida / total (derivada)

### Operacionais

- `de_tipo_operacao` — Tipo de operação (Financiamento, Garantia, Cofinanciamento, etc.)
- `de_fase` — Fase do pleito (Assinado, Desembolso, Encerrado, Negociação)
- `sg_fonte` — Sigla da fonte (BIRD, CAF, JBIC, KfW, etc.)
- `de_fonte` — Descrição completa da fonte financiadora

### Temporais

- `dt_primeira_cofiex` — Data de primeira análise COFIEX
- `dt_assinatura` — Data de assinatura do contrato
- `dt_ultimo_recebimento` — Data do último recebimento
- `dt_ultimo_desembolso_vigente` — Data do último desembolso vigente
- `ano_recebimento`, `ano_primeira_cofiex`, `ano_assinatura` — Extratos (derivados)

### Prazos (Derivados)

- `prazo_recebimento_cofiex` — Dias entre recebimento e primeira COFIEX
- `prazo_cofiex_assinatura` — Dias entre primeira COFIEX e assinatura
- `prazo_assinatura_desembolso` — Dias entre assinatura e primeiro desembolso

### Flags de Qualidade

- `flag_cd_pleito_repetido` — Código aparece em mais de uma linha
- `flag_financiamento_nulo_zero` — Financiamento ausente ou zero
- `flag_contrapartida_maior_financiamento` — Contrapartida > financiamento
- `flag_data_extrema` — Data fora do intervalo 1940-2035
- `flag_data_sentinela` — Data claramente impossível (9999, 4201, etc.)
- `flag_recebimento_placeholder` — Recebimento marcado com placeholder 1940-01-01
- `flag_cronologia_invalida` — Inconsistência na ordem das datas
- `flag_outlier_financeiro` — Valor acima do percentil 99
- `qtd_ocorrencias_cd_pleito` — Número de vezes que cd_pleito aparece

## 🔍 Achados Principais

### Concentração Territorial

O Sudeste concentra 41% dos registros e maior valor financiado. Essa concentração reflete tanto fatores de desenvolvimento econômico quanto de capacidade institucional para elaboração de projetos estruturantes.

### Dependência de Fontes

A carteira depende fortemente de cinco fontes: BIRD (~28% do financiamento), CAF (~21%), JBIC (~15%), KfW (~12%) e FAO (~8%). Os 83% restantes distribuem-se entre 23 fontes menores.

### Evolução Temporal

O volume de financiamentos cresceu aceleradamente a partir de 2010, especialmente em operações de cofinanciamento e linhas de crédito, indicando diversificação da carteira ao longo do período.

### Contrapartida Pública

A taxa média de contrapartida é de 56%, atingindo 58% em anos recentes. Valores extremos (338 registros com contrapartida > financiamento) sinalizam projetos que alavancam recursos públicos ou parcerias privadas.

### Incompletudes Estruturais

283 registros carecem de informação sobre fonte, 289 sobre esfera de governo, 26 sobre UF. Essas lacunas refletem fases iniciais de pleitos ou inconsistências no registro administrativo original.

## 🛠️ Tecnologia e Metodologia

- **Linguagem**: Python 3.8+
- **Notebooks**: Jupyter com análise iterativa
- **Processamento**: Pandas com operações vetorizadas
- **Validação**: Flags multimodais, não remoção de linhas
- **Dashboard**: HTML5 + Chart.js + CSS3 responsivo
- **Controle de Versão**: Git com histórico completo

## 📖 Documentação Técnica

Para detalhes completos sobre:

- Regras de contagem de pleitos repetidos
- Especificação de cada flag de qualidade
- Cálculos de prazos e métricas derivadas
- Decisões de design do ETL

Consulte [AGENTS.md](AGENTS.md) — guia técnico de referência com 1.500+ linhas de documentação.

## 🤝 Contribuições e Feedback

Este projeto é mantido como referência de análise de dados públicos. Sugestões de melhorias, correções de dados ou novas questões podem ser reportadas via issues no GitHub.

## 📜 Licença

Os dados originais provêm de `dados.gov.br` sob a [Licença de Dados Abertos](http://opendatacommons.org/licenses/by/1.0/). O código deste projeto e o dashboard estão disponíveis sob licença MIT.

## 📞 Contato

**Autor**: Grupo 7
**Repositório**: [github.com/devheron/CofiexBI](https://github.com/devheron/CofiexBI)  
**Dados Originais**: [dados.gov.br - COFIEX](https://dados.gov.br/dados/conjuntos-dados/cofiex)

---

**Última atualização**: Junho de 2024  
**Versão do ETL**: 3.0 (Validação Final)  
**Status dos Dados**: 3.548 registros, nenhuma linha removida, todas as anomalias flagadas
