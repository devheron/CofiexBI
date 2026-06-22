# AGENTS.md

## Papel deste documento

Este arquivo orienta o trabalho no projeto **CofiexBI**. Ele deve ser usado por Codex e pelo autor do projeto como guia de contexto, padroes de ETL, regras de qualidade e ordem de execucao das tarefas planejadas no Trello.

O objetivo e construir, em **Jupyter + Python**, um ETL confiavel para os dados de financiamentos externos analisados pela COFIEX, preservando a base bruta e gerando uma base tratada pronta para analises e dashboard.

## Contexto do projeto

Fonte bruta atual:

- Arquivo: `dados_brutos/dados_2026-04-14.xlsx`
- Abas: `Dados` e `Dicionário`
- Volume inicial: 3.548 linhas e 25 colunas
- Data de acesso inicial neste projeto: 2026-05-23
- Data de extracao inferida pelo nome do arquivo: 2026-04-14
- Notebook atual: `notebooks/Untitled.ipynb`, ainda vazio
- Pastas existentes: `dados_brutos/`, `dados_tratados/`, `notebooks/`

A aba `Dados` contem pleitos, operacoes, fontes de financiamento, valores, localizacao do proponente e datas do ciclo COFIEX. A aba `Dicionário` descreve as 25 variaveis.

## Pergunta central oficial

Como a carteira de financiamentos externos da COFIEX revela prioridades, concentracao territorial e dependencia de fontes financiadoras internacionais ao longo do tempo?

Essa pergunta trata a base como uma carteira publica de projetos financiados externamente. O dashboard deve combinar tempo, territorio, valores, fontes financiadoras, tipos de operacao e fases para revelar padroes de concentracao, mudancas historicas e composicao da carteira.

## Perguntas secundarias oficiais

1. Como o volume financeiro e a quantidade de pleitos evoluiram ao longo do tempo?
2. Quais regioes e UFs concentram mais pleitos e maior valor financiado?
3. Quais esferas de governo concentram a carteira: federal, estadual ou municipal?
4. Quais fontes financiadoras tem maior participacao na carteira da COFIEX?
5. Quais tipos de operacao predominam e qual o peso financeiro de cada tipo?
6. Em quais fases os pleitos se concentram e quanto valor esta associado a cada fase?
7. Qual e a relacao entre financiamento e contrapartida nos diferentes grupos da carteira?
8. Ha concentracao da carteira em poucas fontes, regioes, UFs ou proponentes?
9. Quais mudancas de perfil aparecem quando comparamos periodos diferentes?
10. Quais limitacoes e inconsistencias dos dados afetam a leitura da carteira?

## Granularidade oficial da analise

A granularidade primaria da analise sera:

> 1 linha da base = 1 registro de pleito/operacao conforme aparece no arquivo bruto da COFIEX.

A base tratada deve preservar a mesma granularidade do arquivo original. Cada linha representa um registro individual da carteira COFIEX e deve ser mantida no `df_tratado`, salvo exclusao explicitamente justificada e documentada.

A coluna `cd_pleito` nao deve ser assumida como chave unica, porque o diagnostico inicial encontrou:

- 0 linhas duplicadas exatas
- 158 valores de `cd_pleito` repetidos
- 342 linhas envolvidas em repeticoes de `cd_pleito`
- 16 valores nao nulos de `nu_operacao` repetidos

Resultado confirmado no notebook de extracao:

- `qtd_duplicatas_exatas`: 0
- `qtd_cd_pleito_repetidos`: 342 linhas
- `qtd_codigos_cd_pleito_repetidos`: 158 codigos distintos
- `qtd_nu_operacao_repetidos`: 82 linhas
- `qtd_codigos_nu_operacao_repetidos`: 16 numeros de operacao distintos

Regra oficial:

- Manter todas as linhas no `df_tratado`.
- Criar flags auxiliares para repeticao de `cd_pleito` e `nu_operacao`.
- Diferenciar sempre contagem de registros e contagem de pleitos unicos.
- Fazer agregacoes por pleito apenas em tabelas derivadas, com regra explicita de contagem.
- Nao deduplicar automaticamente com base em `cd_pleito`.

Metricas de contagem:

- `qtd_registros`: contagem simples de linhas.
- `qtd_pleitos_unicos`: contagem distinta de `cd_pleito`.

Granularidades analiticas derivadas:

- Ano: `ano_recebimento`, `ano_primeira_cofiex`, `ano_assinatura`.
- Territorio: `nm_regiao`, `sg_uf`, `nm_uf`.
- Esfera: `de_esfera`.
- Fonte financiadora: `sg_fonte`, `de_fonte`.
- Tipo de operacao: `de_tipo_operacao`.
- Fase: `de_fase`.
- Proponente: `nm_proponente`.
- Pleito: `cd_pleito`, sempre com cuidado para repeticoes.

Regra financeira:

- Na visao por linha, manter o valor original.
- Em agregacoes por ano, territorio, fonte, fase, esfera ou tipo de operacao, somar os valores das linhas.
- Em agregacoes por `cd_pleito`, verificar antes se linhas repetidas representam operacoes diferentes, fontes diferentes ou possivel duplicidade administrativa.

## Regras oficiais para contar pleitos repetidos

### Objetivo

Definir como o ETL e o dashboard devem tratar registros com `cd_pleito` repetido, evitando contagens infladas sem perder informacoes relevantes da base original.

### Contexto

A coluna `cd_pleito` identifica o pleito, mas nao deve ser tratada automaticamente como chave unica da base.

No diagnostico inicial da base foram encontrados `cd_pleito` repetidos, mas nao foram encontradas duplicatas exatas de linha.

Resultado confirmado no notebook de extracao:

- `qtd_duplicatas_exatas`: 0
- `qtd_cd_pleito_repetidos`: 342 linhas
- `qtd_codigos_cd_pleito_repetidos`: 158 codigos distintos
- `qtd_nu_operacao_repetidos`: 82 linhas
- `qtd_codigos_nu_operacao_repetidos`: 16 numeros de operacao distintos

Isso indica que as repeticoes podem representar registros administrativos distintos, operacoes diferentes, fases diferentes, fontes diferentes, atualizacoes de informacao ou registros complementares.

Por isso, a base tratada deve preservar todas as linhas originais e criar regras especificas para contagem, auditoria e agregacao.

### Regra principal

A base tratada deve manter todos os registros originais.

Nao remover linhas apenas porque o `cd_pleito` aparece mais de uma vez.

Nao usar deduplicacao por `cd_pleito` sem analise previa.

Criar uma flag para identificar pleitos repetidos.

Separar sempre a contagem de registros da contagem de pleitos unicos.

### Metricas oficiais de contagem

Quantidade de registros

Nome sugerido: `qtd_registros`

Regra: contar todas as linhas da base tratada.

Uso: medir o volume total de ocorrencias registradas na base COFIEX.

Exemplo de calculo: `len(df_tratado)`

Quantidade de pleitos unicos

Nome sugerido: `qtd_pleitos_unicos`

Regra: contar valores distintos de `cd_pleito`.

Uso: medir quantos pleitos diferentes existem na base, sem duplicar pleitos repetidos.

Exemplo de calculo: `df_tratado["cd_pleito"].nunique()`

Flag de pleito repetido

Nome sugerido: `flag_cd_pleito_repetido`

Regra: marcar como verdadeiro quando o mesmo `cd_pleito` aparecer em mais de uma linha.

Uso: permitir filtrar, auditar e analisar pleitos com multiplas ocorrencias.

Exemplo de calculo: `df_tratado["cd_pleito"].duplicated(keep=False)`

Quantidade de ocorrencias por pleito

Nome sugerido: `qtd_ocorrencias_cd_pleito`

Regra: contar quantas linhas existem para cada `cd_pleito`.

Uso: identificar pleitos com maior numero de repeticoes e apoiar auditoria da base.

Exemplo de calculo: `df_tratado.groupby("cd_pleito")["cd_pleito"].transform("size")`

### Regras para o dashboard

Cartoes de resumo devem mostrar separadamente:

- Quantidade de registros
- Quantidade de pleitos unicos
- Percentual de pleitos repetidos

Graficos financeiros por regiao, UF, fonte, fase, esfera ou ano devem usar soma dos valores por linha.

Nao deduplicar automaticamente valores financeiros por `cd_pleito`.

Graficos de quantidade devem usar:

- `qtd_pleitos_unicos` quando o objetivo for contar pleitos distintos.
- `qtd_registros` quando o objetivo for contar ocorrencias da base.

Titulos, legendas ou tooltips devem deixar claro se a metrica exibida representa registros ou pleitos unicos.

### Regras para valores financeiros

Valores financeiros devem ser somados por linha nas analises gerais.

Antes de consolidar valores por `cd_pleito`, verificar se as linhas repetidas possuem diferencas em:

- `nu_operacao`
- `de_fonte`
- `sg_fonte`
- `de_fase`
- datas
- `vl_financiamento_dolar`
- `vl_contrapartida_dolar`

Se houver diferenca relevante entre as linhas do mesmo `cd_pleito`, nao consolidar automaticamente.

Caso seja criada uma tabela agregada por `cd_pleito`, a regra deve ser explicita.

Sugestao de regra para tabela agregada por `cd_pleito`:

- Somar valores financeiros das linhas do mesmo pleito.
- Usar a menor data valida de `dt_ultimo_recebimento`.
- Usar a primeira data valida de `dt_primeira_cofiex`.
- Usar a maior data valida de `dt_assinatura`.
- Usar a maior data valida de `dt_ultimo_desembolso_vigente`.
- Manter lista de fontes associadas.
- Manter lista de fases associadas.
- Registrar `qtd_ocorrencias_cd_pleito`.

### Regras de auditoria

Criar uma visao auxiliar com os `cd_pleito` repetidos.

A visao de auditoria deve conter:

- `cd_pleito`
- `qtd_ocorrencias_cd_pleito`
- `nu_operacao`
- `de_fonte`
- `sg_fonte`
- `de_fase`
- `vl_financiamento_dolar`
- `vl_contrapartida_dolar`
- `dt_ultimo_recebimento`
- `dt_primeira_cofiex`
- `dt_assinatura`
- `dt_ultimo_desembolso_vigente`

Essa auditoria ajudara a entender se os repetidos representam duplicidade, historico, contratos diferentes ou registros complementares.

### Exemplos inspecionados de `cd_pleito` repetidos

Foi criada uma visao auxiliar com registros em que `cd_pleito` aparece mais de uma vez, contendo colunas de auditoria como operacao, pleito, proponente, tipo de operacao, fase, fonte, valores e datas.

A inspecao dos exemplos indica que os `cd_pleito` repetidos nao devem ser tratados automaticamente como duplicatas.

Foram observados casos em que o mesmo `cd_pleito` aparece com:

- fontes financiadoras diferentes;
- siglas de fonte diferentes;
- valores de financiamento diferentes;
- numeros de operacao diferentes;
- fases diferentes;
- registros com datas iguais, mas informacoes financeiras ou operacionais distintas.

Exemplo observado:

- O `cd_pleito` 120 aparece para o mesmo programa e proponente, mas com fontes diferentes: BID, com financiamento de 20.000.000; e BIRD, com financiamento de 28.000.000.

Tambem foram observados pleitos recentes repetidos com fontes como Fundo OPEP, FONPLATA, BIRD e BID, indicando que um mesmo pleito pode estar associado a mais de uma fonte financiadora.

Tambem foram observados casos antigos em que o mesmo pleito aparece com fases diferentes, como `Finalizada` e `Repagamento`, ou com numeros de operacao distintos.

Conclusao:

- A repeticao de `cd_pleito` representa, em muitos casos, multiplas ocorrencias relevantes do mesmo pleito, e nao duplicatas exatas.
- A base tratada deve preservar todas as linhas.
- A deduplicacao por `cd_pleito` nao deve ser aplicada automaticamente.
- O dashboard deve diferenciar quantidade de registros, quantidade de pleitos unicos e registros com `cd_pleito` repetido.
- Na etapa de transformacao, deve ser criada a flag `flag_cd_pleito_repetido`.
- Tambem deve ser criada uma visao auxiliar de auditoria contendo os pleitos repetidos para analise posterior.

### Definicao oficial

A analise deve preservar todas as linhas da base COFIEX.

O campo `cd_pleito` sera usado para identificar pleitos unicos, mas nao sera usado para remover registros automaticamente.

O dashboard devera diferenciar `qtd_registros` de `qtd_pleitos_unicos`.

Pleitos repetidos serao marcados por `flag_cd_pleito_repetido` e auditados antes de qualquer consolidacao financeira ou deduplicacao.

## Padrao de pastas

Usar a seguinte organizacao:

```text
CofiexBI/
  AGENTS.md
  dados_brutos/
    dados_2026-04-14.xlsx
  dados_tratados/
    cofiex_tratado.csv
    relatorio_qualidade_colunas.csv
  notebooks/
    01_extracao_diagnostico.ipynb
    02_limpeza_transformacao.ipynb
    03_validacao_carga.ipynb
```

Se o projeto crescer, criar tambem:

```text
  src/
    config.py
    extract.py
    transform.py
    validate.py
  reports/
    qualidade/
```

## Regras gerais de trabalho

- Preservar sempre os arquivos em `dados_brutos/`.
- Nunca sobrescrever a base bruta.
- Ler a aba `Dados` como `df_raw`.
- Criar `df_tratado = df_raw.copy()` antes de qualquer limpeza.
- Usar caminhos com `pathlib.Path`.
- Manter celulas de notebook em ordem reprodutivel: imports, caminhos, extracao, diagnostico, limpeza, transformacao, validacao e carga.
- Toda transformacao relevante deve ser documentada em Markdown no notebook.
- Campos ausentes textuais devem virar nulos reais (`pd.NA`), nao strings vazias.
- Datas devem ser convertidas com `pd.to_datetime(..., errors="coerce")`.
- Valores monetarios devem permanecer numericos.
- Exportar CSV tratado com encoding `utf-8-sig`.
- Ao final, testar a leitura do CSV exportado.

## Colunas necessarias por indicador

Esta secao conecta a pergunta central e as perguntas secundarias aos indicadores que o dashboard deve calcular. O ETL deve garantir que as colunas abaixo estejam limpas, tipadas e documentadas antes da carga.

| Indicador | Objetivo analitico | Colunas brutas necessarias | Colunas derivadas / auxiliares |
|---|---|---|---|
| Total financiado | Medir o volume financeiro da carteira | `vl_financiamento_dolar` | - |
| Total de contrapartida | Medir o esforco financeiro complementar | `vl_contrapartida_dolar` | - |
| Valor total da carteira | Combinar financiamento e contrapartida | `vl_financiamento_dolar`, `vl_contrapartida_dolar` | `valor_total_dolar` |
| Percentual de contrapartida | Avaliar composicao financeira dos projetos | `vl_financiamento_dolar`, `vl_contrapartida_dolar` | `valor_total_dolar`, `percentual_contrapartida` |
| Quantidade de registros | Medir volume total de linhas da base | todas as linhas preservadas | `qtd_registros` em agregacoes |
| Quantidade de pleitos unicos | Evitar confundir linhas com pleitos distintos | `cd_pleito` | `qtd_pleitos_unicos`, `flag_cd_pleito_repetido` |
| Ticket medio por registro | Medir valor medio por ocorrencia da base | `vl_financiamento_dolar`, `vl_contrapartida_dolar` | `valor_total_dolar` |
| Evolucao anual por recebimento | Entender entrada dos pleitos ao longo do tempo | `dt_ultimo_recebimento` | `ano_recebimento` |
| Evolucao anual por primeira COFIEX | Entender aprovacao/preparacao ao longo do tempo | `dt_primeira_cofiex` | `ano_primeira_cofiex` |
| Evolucao anual por assinatura | Entender contratos assinados ao longo do tempo | `dt_assinatura` | `ano_assinatura` |
| Distribuicao por regiao | Medir concentracao territorial regional | `nm_regiao` | - |
| Distribuicao por UF | Medir concentracao territorial por unidade federativa | `nm_uf`, `sg_uf` | - |
| Distribuicao por esfera | Comparar federal, estadual e municipal | `de_esfera` | - |
| Ranking de fontes financiadoras | Medir dependencia/concentracao por organismo | `de_fonte`, `sg_fonte` | `flag_sem_fonte` |
| Participacao das fontes no tempo | Ver mudancas historicas das fontes financiadoras | `de_fonte`, `sg_fonte`, datas selecionadas | `ano_recebimento`, `ano_primeira_cofiex` ou `ano_assinatura` |
| Distribuicao por tipo de operacao | Identificar modalidades predominantes | `de_tipo_operacao` | - |
| Distribuicao por fase | Mostrar situacao atual da carteira | `de_fase` | - |
| Concentracao por proponente | Identificar entes ou instituicoes com maior peso | `nm_proponente` | - |
| Prazo recebimento ate primeira COFIEX | Medir tempo inicial de tramitacao | `dt_ultimo_recebimento`, `dt_primeira_cofiex` | `prazo_recebimento_cofiex` |
| Prazo primeira COFIEX ate assinatura | Medir tempo ate formalizacao do contrato | `dt_primeira_cofiex`, `dt_assinatura` | `prazo_cofiex_assinatura` |
| Prazo assinatura ate desembolso vigente | Medir horizonte posterior a assinatura | `dt_assinatura`, `dt_ultimo_desembolso_vigente` | `prazo_assinatura_desembolso` |
| Validade da recomendacao | Avaliar vencimento ou prazo de recomendacao | `dt_primeira_cofiex`, `dt_validade_recomendacao` | prazo derivado opcional |
| Publicacao no DOU | Acompanhar etapa formal de publicacao | `dt_publicacao_dou` | ano derivado opcional |
| Operacoes/contratos identificados | Analisar registros com numero de contrato/operacao | `nu_operacao` | `flag_nu_operacao_repetida` |
| Qualidade de preenchimento por coluna | Medir completude da base | todas as colunas | `relatorio_qualidade_colunas.csv` |
| Campos financeiros ausentes | Identificar registros sem valores essenciais | `vl_financiamento_dolar`, `vl_contrapartida_dolar` | `flag_sem_financiamento`, `flag_sem_contrapartida` |
| Campos de ciclo ausentes | Identificar registros sem marcos temporais importantes | `dt_primeira_cofiex`, `dt_assinatura` | `flag_sem_primeira_cofiex`, `flag_sem_assinatura` |
| Cronologia invalida | Controlar inconsistencias temporais | todas as colunas `dt_*` relevantes | `flag_cronologia_invalida` |

Eixo temporal oficial para o dashboard:

- Usar `ano_recebimento` para analisar entrada de pleitos na carteira.
- Usar `ano_primeira_cofiex` para analisar o momento de aprovacao/preparacao pela COFIEX.
- Usar `ano_assinatura` para analisar contratos efetivamente assinados.
- Quando houver apenas um eixo temporal em visual executivo, priorizar `ano_primeira_cofiex`, pois ele conversa melhor com a atuacao da COFIEX. Se a analise for sobre entrada da demanda, usar `ano_recebimento`; se for sobre carteira contratada, usar `ano_assinatura`.

Medidas minimas do dashboard:

- `qtd_registros`
- `qtd_pleitos_unicos`
- `valor_financiamento_total`
- `valor_contrapartida_total`
- `valor_total_dolar`
- `ticket_medio`
- `percentual_contrapartida_medio`
- participacao percentual por fonte, regiao, UF, esfera, fase e tipo de operacao

## Dicionario de dados bruto

| Coluna | Descricao |
|---|---|
| `cd_pleito` | Numero do pleito |
| `nu_processo_sei` | Numero do processo SEI |
| `nm_pleito` | Nome do programa/projeto |
| `sg_pleito` | Sigla do programa/projeto |
| `nm_proponente` | Nome do proponente |
| `de_tipo_operacao` | Modalidade da operacao |
| `de_fase` | Fase do programa/projeto no momento da extracao |
| `de_fonte` | Fonte de financiamento / instituicao financeira |
| `sg_fonte` | Sigla da fonte de financiamento |
| `nm_moeda` | Moeda de financiamento presente no contrato |
| `vl_financiamento_dolar` | Valor do financiamento em dolares americanos |
| `vl_contrapartida_dolar` | Valor da contrapartida financeira em dolares americanos |
| `de_esfera` | Esfera do programa/projeto |
| `nm_regiao` | Regiao geografica do proponente |
| `nm_uf` | Unidade federativa do proponente |
| `sg_uf` | Sigla da unidade federativa do proponente |
| `dt_ultimo_recebimento` | Data de recebimento da ultima versao do pleito |
| `dt_primeira_cofiex` | Data da reuniao COFIEX de aprovacao da preparacao |
| `dt_validade_recomendacao` | Data de validade da resolucao COFIEX |
| `dt_reuniao_negociacao` | Data da reuniao de negociacao do contrato |
| `dt_publicacao_dou` | Data de publicacao no DOU |
| `dt_assinatura` | Data de assinatura do contrato de emprestimo apos aprovacao do Senado |
| `dt_ultimo_desembolso_original` | Data/previsao do ultimo desembolso no projeto original |
| `dt_ultimo_desembolso_vigente` | Data/previsao vigente do ultimo desembolso no momento da extracao |
| `nu_operacao` | Numero da operacao ou contrato |

## Diagnostico inicial conhecido

Categorias principais:

- `de_tipo_operacao`: 7 categorias; maior grupo e operacao de credito externo.
- `de_fase`: 8 categorias; maiores grupos sao arquivado, repagamento e finalizada.
- `de_fonte`: 115 categorias; principais fontes incluem BID, BIRD, CAF, GEF e FONPLATA.
- `sg_fonte`: 112 categorias.
- `de_esfera`: estadual, federal, municipal e ausente.
- `nm_regiao`: Nacional, Sudeste, Nordeste, Sul, Norte e Centro-Oeste.
- `sg_uf`: 28 categorias, incluindo DF e UFs.
- `nm_moeda`: Dolar, Euro, Real, SDR, Iene, Libra e ausente.

Distribuicao de `de_tipo_operacao`:

| Categoria | Registros | Percentual |
|---|---:|---:|
| Operacao de credito externo | 2.643 | 74,49% |
| Doacoes | 532 | 14,99% |
| Contribuicao Financeira nao Reembolsavel GEF | 273 | 7,69% |
| Outros | 64 | 1,80% |
| Contribuicao financeira nao reembolsavel | 15 | 0,42% |
| Operacao Comercial | 12 | 0,34% |
| Cooperacao Tecnica GEF | 9 | 0,25% |

Interpretacao de `de_tipo_operacao`:

- A base e fortemente concentrada em operacao de credito externo, com 74,49% dos registros.
- A segunda maior categoria e doacoes, com 14,99% dos registros.
- `de_tipo_operacao` deve ser usada como dimensao principal para comparar quantidade de pleitos, valor financiado e fases por modalidade.
- Na etapa de limpeza, avaliar se `Contribuicao Financeira nao Reembolsavel GEF` e `Contribuicao financeira nao reembolsavel` devem permanecer separadas ou se representam variacoes de uma mesma modalidade.

Distribuicao de `de_fase`:

| Categoria | Registros | Percentual |
|---|---:|---:|
| Arquivado | 1.587 | 44,73% |
| Repagamento | 706 | 19,90% |
| Finalizada | 598 | 16,85% |
| Aprovado | 197 | 5,55% |
| Reprovada | 190 | 5,36% |
| Em execucao | 168 | 4,74% |
| Negociacao concluida | 95 | 2,68% |
| Em negociacao | 7 | 0,20% |

Interpretacao de `de_fase`:

- A maior parte dos registros esta em `Arquivado`, representando 44,73% da base.
- As fases `Repagamento` e `Finalizada` tambem possuem peso relevante, com 19,90% e 16,85% dos registros.
- A coluna `de_fase` nao possui nulos e deve ser usada como dimensao central para analisar a situacao da carteira.
- Para o dashboard, e importante separar a leitura de fases encerradas, fases ativas e fases de rejeicao/arquivamento.

Distribuicao de `sg_fonte`:

`sg_fonte` possui 112 categorias, contando valores nulos como categoria.

Principais categorias observadas:

| Categoria | Registros | Percentual |
|---|---:|---:|
| BID | 898 | 25,31% |
| BIRD | 812 | 22,89% |
| Nulo | 284 | 8,00% |
| CAF | 237 | 6,68% |
| GEF | 161 | 4,54% |

Interpretacao de `sg_fonte`:

- A carteira possui forte concentracao inicial em BID e BIRD, que juntos representam 48,20% dos registros.
- Existem 284 registros sem `sg_fonte`, equivalentes a 8,00% da base.
- A alta quantidade de categorias indica que o dashboard deve priorizar rankings, Top N e agrupamentos, em vez de exibir todas as fontes ao mesmo tempo.
- `sg_fonte` deve ser usada como dimensao central para analisar dependencia de fontes financiadoras internacionais.
- Na limpeza, sera necessario avaliar padronizacoes de siglas e possiveis variacoes de escrita.

Distribuicao de `nm_regiao`:

`nm_regiao` possui 6 categorias e nao possui valores nulos.

| Categoria | Registros | Percentual |
|---|---:|---:|
| Nacional | 1.087 | 30,64% |
| Sudeste | 844 | 23,79% |
| Nordeste | 618 | 17,42% |
| Sul | 472 | 13,30% |
| Norte | 270 | 7,61% |
| Centro-Oeste | 257 | 7,24% |

Distribuicao de `sg_uf`:

`sg_uf` possui 28 categorias, contando valores nulos como categoria.

| Categoria | Registros | Percentual |
|---|---:|---:|
| DF | 1.134 | 31,96% |
| SP | 369 | 10,40% |
| RJ | 230 | 6,48% |
| RS | 211 | 5,95% |
| MG | 176 | 4,96% |
| CE | 160 | 4,51% |
| SC | 143 | 4,03% |
| BA | 120 | 3,38% |
| PR | 118 | 3,33% |
| PE | 96 | 2,71% |
| PA | 81 | 2,28% |
| AM | 76 | 2,14% |
| ES | 69 | 1,94% |
| GO | 63 | 1,78% |
| PI | 61 | 1,72% |
| MS | 59 | 1,66% |
| MA | 51 | 1,44% |
| TO | 50 | 1,41% |
| PB | 45 | 1,27% |
| SE | 40 | 1,13% |
| RN | 33 | 0,93% |
| MT | 33 | 0,93% |
| AL | 30 | 0,85% |
| AC | 26 | 0,73% |
| Nulo | 26 | 0,73% |
| RO | 22 | 0,62% |
| AP | 18 | 0,51% |
| RR | 8 | 0,23% |

Interpretacao territorial:

- A categoria `Nacional` representa 30,64% dos registros em `nm_regiao`, indicando que parte relevante da carteira nao deve ser lida como operacao territorializada por regiao comum.
- `DF` concentra 31,96% dos registros em `sg_uf`; essa concentracao deve ser interpretada com cautela, pois pode refletir operacoes nacionais/federais e nao apenas projetos territoriais do Distrito Federal.
- `nm_regiao` e `sg_uf` devem ser usadas como dimensoes centrais para analisar concentracao territorial, mas o dashboard deve separar ou sinalizar registros nacionais quando a leitura territorial puder ser distorcida.
- Existem 26 registros sem `sg_uf`, equivalentes a 0,73% da base.

Distribuicao de `de_esfera`:

`de_esfera` possui 4 categorias, contando valores nulos como categoria.

| Categoria | Registros | Percentual |
|---|---:|---:|
| Estadual | 1.330 | 37,49% |
| Federal | 1.218 | 34,33% |
| Municipal | 711 | 20,04% |
| Nulo | 289 | 8,15% |

Interpretacao de `de_esfera`:

- A carteira possui maior presenca de registros estaduais, com 37,49% da base.
- A esfera federal tambem tem peso elevado, com 34,33% dos registros.
- A esfera municipal representa 20,04% dos registros.
- Existem 289 registros sem `de_esfera`, equivalentes a 8,15% da base, que devem ser tratados ou sinalizados antes de analises por esfera.
- `de_esfera` deve ser usada como dimensao central para comparar perfil institucional, mas os nulos precisam aparecer como categoria auditavel ou receber tratamento documentado.

Distribuicao de `nm_moeda`:

`nm_moeda` possui 7 categorias, contando valores nulos como categoria.

| Categoria | Registros | Percentual |
|---|---:|---:|
| Dolar | 2.983 | 84,08% |
| Nulo | 283 | 7,98% |
| Euro | 166 | 4,68% |
| Real | 86 | 2,42% |
| SDR | 17 | 0,48% |
| Iene | 12 | 0,34% |
| Libra | 1 | 0,03% |

Interpretacao de `nm_moeda`:

- A maior parte da carteira esta associada a `Dolar`, com 84,08% dos registros.
- Existem 283 registros sem moeda, equivalentes a 7,98% da base.
- Embora os valores financeiros estejam em colunas denominadas em dolar, `nm_moeda` deve ser preservada para auditoria e contextualizacao do contrato.
- A coluna pode ser usada como filtro secundario, mas nao deve ser uma dimensao principal do dashboard executivo.

Quantidade de valores unicos por coluna, contando nulos como categoria:

```text
cd_pleito: 3.364
nm_pleito: 3.248
sg_pleito: 2.925
dt_ultimo_recebimento: 1.717
vl_financiamento_dolar: 1.515
nu_operacao: 1.338
vl_contrapartida_dolar: 1.262
dt_assinatura: 1.106
nu_processo_sei: 1.027
dt_ultimo_desembolso_vigente: 895
dt_reuniao_negociacao: 830
dt_ultimo_desembolso_original: 804
nm_proponente: 488
dt_primeira_cofiex: 394
dt_validade_recomendacao: 290
dt_publicacao_dou: 219
de_fonte: 115
sg_fonte: 112
sg_uf: 28
nm_uf: 28
de_fase: 8
de_tipo_operacao: 7
nm_moeda: 7
nm_regiao: 6
de_esfera: 4
```

Interpretacao:

- `cd_pleito`, `nm_pleito`, `sg_pleito`, `nu_operacao` e `nu_processo_sei` possuem alta cardinalidade e devem ser tratados como identificadores ou campos descritivos, nao como categorias principais de filtro amplo.
- `de_fonte` e `sg_fonte` possuem cardinalidade intermediaria e podem exigir agrupamento ou ranking no dashboard.
- `de_fase`, `de_tipo_operacao`, `nm_moeda`, `nm_regiao` e `de_esfera` possuem baixa cardinalidade e sao boas dimensoes para filtros e segmentacoes.
- Colunas de data e valores financeiros possuem muitos valores distintos, o que reforca a necessidade de criar colunas derivadas como anos, prazos e faixas analiticas.

Tipos de dados inferidos pelo pandas:

```text
cd_pleito: object
nu_processo_sei: object
nm_pleito: object
sg_pleito: object
nm_proponente: object
de_tipo_operacao: object
de_fase: object
de_fonte: object
sg_fonte: object
nm_moeda: object
vl_financiamento_dolar: float64
vl_contrapartida_dolar: float64
de_esfera: object
nm_regiao: object
nm_uf: object
sg_uf: object
dt_ultimo_recebimento: object
dt_primeira_cofiex: object
dt_validade_recomendacao: object
dt_reuniao_negociacao: object
dt_publicacao_dou: object
dt_assinatura: object
dt_ultimo_desembolso_original: object
dt_ultimo_desembolso_vigente: object
nu_operacao: object
```

Interpretacao dos tipos:

- A maioria das colunas foi inferida como `object`, incluindo identificadores, textos categoricos e datas.
- As colunas financeiras `vl_financiamento_dolar` e `vl_contrapartida_dolar` foram inferidas corretamente como `float64`.
- Todas as colunas `dt_*` foram inferidas como `object`, portanto devem ser convertidas explicitamente com `pd.to_datetime(..., errors="coerce")` na etapa de limpeza.
- Identificadores como `cd_pleito`, `nu_processo_sei` e `nu_operacao` devem permanecer como texto para evitar perda de zeros, caracteres especiais ou formatacao administrativa.

Relatorio inicial de qualidade por coluna:

Foi gerado um relatorio inicial de qualidade por coluna da base `df_raw`, contendo:

- nome da coluna;
- tipo inferido pelo pandas;
- quantidade de nulos;
- percentual de nulos;
- quantidade de valores unicos;
- percentual de valores unicos em relacao ao total de registros.

Principais observacoes:

1. Colunas com maior ausencia

As maiores ausencias estao concentradas em campos de datas do ciclo COFIEX e identificacao administrativa:

- `dt_reuniao_negociacao`: 74,13% de nulos;
- `dt_validade_recomendacao`: 71,98% de nulos;
- `dt_publicacao_dou`: 69,22% de nulos;
- `nu_processo_sei`: 69,17% de nulos;
- `dt_ultimo_desembolso_original`: 65,11% de nulos;
- `nu_operacao`: 60,46% de nulos;
- `dt_ultimo_desembolso_vigente`: 60,20% de nulos;
- `dt_assinatura`: 60,09% de nulos.

2. Colunas financeiras

As colunas financeiras foram inferidas como `float64`:

- `vl_financiamento_dolar`;
- `vl_contrapartida_dolar`.

Porem, possuem ausencias relevantes:

- `vl_financiamento_dolar`: 7,98% de nulos;
- `vl_contrapartida_dolar`: 29,85% de nulos.

3. Colunas de alta cardinalidade

Algumas colunas possuem percentual elevado de valores unicos:

- `cd_pleito`: 94,81%;
- `nm_pleito`: 91,54%;
- `sg_pleito`: 82,44%.

Essas colunas devem ser tratadas como identificadores ou campos descritivos, e nao como dimensoes categoricas principais.

4. Colunas boas para segmentacao

Algumas colunas possuem baixa cardinalidade e boa completude, sendo adequadas para filtros e segmentacoes do dashboard:

- `de_tipo_operacao`;
- `de_fase`;
- `nm_regiao`;
- `de_esfera`;
- `nm_moeda`;
- `sg_uf`;
- `sg_fonte`.

5. Conclusao inicial

A base tem boa estrutura para analise por fase, tipo de operacao, regiao, UF, esfera e fonte financiadora.

As maiores limitacoes estao nas datas do ciclo COFIEX e em alguns campos financeiros. Por isso, o ETL deve criar flags de qualidade, tratar datas com cuidado e preservar registros problematicos para auditoria, sem remove-los automaticamente.

Ausencias relevantes:

- `nu_processo_sei`: 2.454 nulos
- `de_fonte`: 283 nulos
- `sg_fonte`: 284 nulos
- `vl_financiamento_dolar`: 283 nulos
- `vl_contrapartida_dolar`: 1.059 nulos
- `dt_primeira_cofiex`: 1.630 nulos
- `dt_reuniao_negociacao`: 2.630 nulos
- `dt_assinatura`: 2.132 nulos
- `dt_ultimo_desembolso_vigente`: 2.136 nulos

Alertas iniciais:

- Existem datas extremas que devem ser validadas, como `2228-03-29` e `2225-01-01`.
- Ha 75 casos em que `dt_ultimo_recebimento` e posterior a `dt_primeira_cofiex`.
- Ha 4 casos em que `dt_reuniao_negociacao` e posterior a `dt_assinatura`.
- Ha 1 caso em que `dt_assinatura` e posterior a `dt_ultimo_desembolso_vigente`.
- Ha 10 casos em que `dt_ultimo_desembolso_vigente` e anterior a `dt_ultimo_desembolso_original`.
- Ha 312 registros com financiamento nulo ou zero.
- Ha 338 registros em que a contrapartida e maior que o financiamento.

Esses alertas nao devem ser automaticamente apagados. Eles devem virar flags, relatorio de qualidade e, quando necessario, decisao documentada.

## Limitacoes iniciais da base

### Objetivo

Documentar as principais limitacoes encontradas na base bruta da COFIEX antes do inicio do tratamento, para orientar decisoes do ETL, evitar interpretacoes erradas no dashboard e dar transparencia a analise.

### Arquivo analisado

```text
dados_brutos/dados_2026-04-14.xlsx
```

### Abas identificadas

```text
Dados
Dicionário
```

### Dimensao inicial da base

```text
3.548 linhas
25 colunas
```

### Limitacoes de identificacao

A coluna `cd_pleito` nao e unica.

Diagnostico inicial:

- 0 duplicatas exatas de linha
- 158 valores de `cd_pleito` repetidos
- 342 linhas envolvidas em repeticoes de `cd_pleito`
- 16 valores nao nulos de `nu_operacao` repetidos

Resultado confirmado no notebook de extracao:

- `qtd_duplicatas_exatas`: 0
- `qtd_cd_pleito_repetidos`: 342 linhas
- `qtd_codigos_cd_pleito_repetidos`: 158 codigos distintos
- `qtd_nu_operacao_repetidos`: 82 linhas
- `qtd_codigos_nu_operacao_repetidos`: 16 numeros de operacao distintos

Impacto:

- Nao e seguro remover duplicatas usando apenas `cd_pleito`.
- O dashboard deve diferenciar registros totais de pleitos unicos.
- Pleitos repetidos precisam ser auditados antes de qualquer consolidacao.

### Limitacoes de completude

Algumas colunas possuem grande quantidade de valores ausentes.

Principais ausencias identificadas:

```text
dt_reuniao_negociacao: 2.630 nulos, 74,13%
dt_validade_recomendacao: 2.554 nulos, 71,98%
dt_publicacao_dou: 2.456 nulos, 69,22%
nu_processo_sei: 2.454 nulos, 69,17%
dt_ultimo_desembolso_original: 2.310 nulos, 65,11%
nu_operacao: 2.145 nulos, 60,46%
dt_ultimo_desembolso_vigente: 2.136 nulos, 60,20%
dt_assinatura: 2.132 nulos, 60,09%
dt_primeira_cofiex: 1.630 nulos, 45,94%
vl_contrapartida_dolar: 1.059 nulos, 29,85%
dt_ultimo_recebimento: 406 nulos, 11,44%
sg_pleito: 393 nulos, 11,08%
de_esfera: 289 nulos, 8,15%
sg_fonte: 284 nulos, 8,00%
vl_financiamento_dolar: 283 nulos, 7,98%
nm_moeda: 283 nulos, 7,98%
de_fonte: 283 nulos, 7,98%
nm_uf: 26 nulos, 0,73%
sg_uf: 26 nulos, 0,73%
nm_pleito: 1 nulo, 0,03%
```

Impacto:

- Indicadores por fonte podem ter registros sem identificacao da instituicao financiadora.
- Indicadores financeiros podem ter valores incompletos.
- Analises de prazo e cronologia serao afetadas por muitos nulos em datas.
- Algumas visualizacoes temporais devem informar a quantidade de registros sem data valida.
- As colunas `cd_pleito`, `nm_proponente`, `de_tipo_operacao`, `de_fase` e `nm_regiao` nao possuem valores nulos, favorecendo segmentacoes principais do dashboard.

### Limitacoes nas datas

Foram identificadas 8 colunas de data na base:

```text
dt_ultimo_recebimento
dt_primeira_cofiex
dt_validade_recomendacao
dt_reuniao_negociacao
dt_publicacao_dou
dt_assinatura
dt_ultimo_desembolso_original
dt_ultimo_desembolso_vigente
```

As colunas de data foram lidas inicialmente como texto.

Percentual de datas ausentes por coluna:

| Coluna | Nulos | Percentual |
|---|---:|---:|
| `dt_reuniao_negociacao` | 2.630 | 74,13% |
| `dt_validade_recomendacao` | 2.554 | 71,98% |
| `dt_publicacao_dou` | 2.456 | 69,22% |
| `dt_ultimo_desembolso_original` | 2.310 | 65,11% |
| `dt_ultimo_desembolso_vigente` | 2.136 | 60,20% |
| `dt_assinatura` | 2.132 | 60,09% |
| `dt_primeira_cofiex` | 1.630 | 45,94% |
| `dt_ultimo_recebimento` | 406 | 11,44% |

Datas minimas e maximas por coluna, apos conversao temporaria com `pd.to_datetime(..., errors="coerce")`:

| Coluna | Data minima | Data maxima | Datas invalidas ou nulas |
|---|---:|---:|---:|
| `dt_ultimo_recebimento` | 1940-01-01 | 2026-03-23 | 406 |
| `dt_primeira_cofiex` | 1990-05-30 | 2026-04-01 | 1.630 |
| `dt_validade_recomendacao` | 2000-09-11 | 2027-12-22 | 2.555 |
| `dt_reuniao_negociacao` | 1989-11-07 | 2026-04-01 | 2.630 |
| `dt_publicacao_dou` | 1998-06-24 | 2025-11-21 | 2.456 |
| `dt_assinatura` | 1949-01-27 | 2025-12-30 | 2.132 |
| `dt_ultimo_desembolso_original` | 1986-12-31 | 2228-03-29 | 2.311 |
| `dt_ultimo_desembolso_vigente` | 1953-12-31 | 2225-01-01 | 2.136 |

Alem disso, existem datas extremas ou possivelmente sentinela, como:

```text
2228-03-29
2225-01-01
```

Impacto:

- Todas as colunas `dt_*` precisam ser convertidas com cuidado.
- Datas fora de faixa plausivel devem ser marcadas por flag.
- Datas extremas nao devem ser usadas diretamente em graficos temporais sem validacao.
- Indicadores de prazo podem ser distorcidos se essas datas nao forem tratadas.
- Analises temporais devem informar ou filtrar registros sem data valida, principalmente em marcos posteriores do ciclo COFIEX.
- Datas maximas muito futuras em campos de desembolso podem representar erro, data sentinela ou previsao extrema e devem ser auditadas antes de calculos de prazo.

### Limitacoes de cronologia

Foram identificadas possiveis inconsistencias temporais:

```text
75 casos em que dt_ultimo_recebimento e posterior a dt_primeira_cofiex
4 casos em que dt_reuniao_negociacao e posterior a dt_assinatura
1 caso em que dt_assinatura e posterior a dt_ultimo_desembolso_vigente
10 casos em que dt_ultimo_desembolso_vigente e anterior a dt_ultimo_desembolso_original
```

Impacto:

- Nem toda sequencia de datas representa uma cronologia valida.
- O ETL deve criar `flag_cronologia_invalida`.
- Registros inconsistentes devem ser auditados, nao removidos automaticamente.

### Limitacoes financeiras

Foram encontrados registros com valores financeiros ausentes ou potencialmente problematicos.

Diagnostico inicial:

```text
283 registros com financiamento nulo, 7,98%
29 registros com financiamento zero, 0,82%
312 registros com financiamento nulo ou zero, 8,79%
1.059 registros com contrapartida nula, 29,85%
338 registros em que a contrapartida e maior que o financiamento, 9,53%
```

Impacto:

- O calculo de `valor_total_dolar` precisa tratar nulos com regra explicita.
- O calculo de `percentual_contrapartida` deve proteger contra divisao por zero.
- Registros com `vl_financiamento_dolar` nulo ou zero devem ser marcados por `flag_sem_financiamento` antes de qualquer analise financeira.
- Registros com `vl_contrapartida_dolar` nula devem ser marcados por `flag_sem_contrapartida`.
- Contrapartida maior que financiamento nao deve ser considerada erro automaticamente, mas deve ser sinalizada para analise e auditoria.

### Outliers financeiros confirmados no arquivo bruto

Foi investigada a possibilidade de que valores financeiros muito altos fossem resultado de formatacao do pandas ou diferenca entre separador de milhar brasileiro e americano.

Conclusao da verificacao:

- A formatacao do pandas nao alterou os valores.
- Os valores extremos existem no arquivo Excel bruto.
- O Excel apresenta os valores com separador de milhar no padrao brasileiro.
- O pandas pode exibir os mesmos valores sem separador, com ponto decimal ou em notacao cientifica, mas isso e apenas formato de exibicao.

Exemplos confirmados diretamente no Excel bruto:

| `cd_pleito` | Descricao observada | `vl_financiamento_dolar` | `vl_contrapartida_dolar` | Observacao |
|---|---|---:|---:|---|
| 59637 | Consolidacao do Programa Bolsa Familia e Apoio... | 200.000.000 | 10.015.000.000 | Maior contrapartida identificada; valor confirmado no Excel bruto. |
| 59454 | Programa Integrado de Melhoria Ambiental na Area... | nulo | 6.208.000.000 | Contrapartida bilionaria com financiamento nulo; valor confirmado no Excel bruto. |

Diagnostico:

- A base contem outliers financeiros reais em sua origem.
- Esses valores podem representar programas muito grandes, contrapartidas acumuladas, erro de escala, erro de digitacao ou regra de negocio especifica da fonte original.
- Nao ha evidencia suficiente, nesta etapa, para corrigir ou remover esses registros.
- Os outliers podem distorcer totais, medias, rankings e comparacoes territoriais ou por fonte.

Regra de tratamento:

- Preservar os valores originais no `df_raw`.
- Manter os valores no `df_tratado`, salvo decisao posterior documentada.
- Criar uma flag de outlier financeiro na etapa de transformacao/validacao.
- Nao arredondar valores financeiros automaticamente, pois existem valores com casas decimais reais.
- Aplicar formatacao de milhar apenas na visualizacao, relatorios ou dashboard.
- Avaliar indicadores financeiros com e sem outliers quando houver distorcao relevante.
- Registrar no dashboard ou documentacao quando uma visualizacao for sensivel a outliers.

Flag recomendada:

- `flag_outlier_financeiro`

Possiveis criterios para a flag, a definir na etapa de validacao:

- valores acima de percentil 99;
- valores acima de limite absoluto definido por regra de negocio;
- contrapartida muito superior ao financiamento;
- financiamento nulo com contrapartida muito alta.

### Limitacoes categoricas

Algumas colunas categoricas possuem muitas categorias ou variacoes.

Exemplos:

```text
de_fonte: 115 categorias
sg_fonte: 112 categorias
de_tipo_operacao: 7 categorias
de_fase: 8 categorias
sg_uf: 28 categorias
```

Impacto:

- `de_fonte` e `sg_fonte` podem exigir padronizacao.
- Rankings de fonte devem considerar possiveis variacoes de escrita.
- Categorias devem ser revisadas antes de fechar o dashboard.

### Limitacoes territoriais

A base contem registros com regiao `Nacional` e muitos registros associados ao `DF`.

Impacto:

- `DF` pode representar tanto registros territoriais quanto operacoes de abrangencia nacional/federal.
- Analises por UF devem ser interpretadas com cuidado.
- Pode ser necessario separar operacoes nacionais de operacoes territorializadas.

### Limitacoes do eixo temporal

Existem tres possiveis marcos temporais principais:

```text
dt_ultimo_recebimento
dt_primeira_cofiex
dt_assinatura
```

Impacto:

- A escolha do ano altera a interpretacao do dashboard.
- `ano_recebimento` representa entrada do pleito.
- `ano_primeira_cofiex` representa atuacao/aprovacao da COFIEX.
- `ano_assinatura` representa carteira contratada.
- Para visao executiva, a definicao inicial e priorizar `ano_primeira_cofiex`.

### Definicao oficial

A base bruta possui limitacoes relevantes de identificacao, completude, datas, cronologia, valores financeiros e padronizacao categorica.

Essas limitacoes nao impedem o projeto, mas exigem um ETL cuidadoso, com flags de qualidade, relatorios auxiliares e documentacao das decisoes de tratamento.

Nenhum registro deve ser removido automaticamente por repeticao, ausencia ou inconsistencia sem regra explicita e justificativa documentada.

## Checklist Trello consolidado

### Planejamento

- [ ] Definir pergunta central do dashboard.
- [ ] Definir perguntas secundarias.
- [ ] Definir granularidade da analise.
- [ ] Mapear colunas necessarias por indicador.
- [ ] Definir regras para contar pleitos repetidos.
- [ ] Definir padrao de pastas do projeto.
- [ ] Registrar limitacoes iniciais da base.

### Extracao

- [ ] Confirmar nome do arquivo bruto correto.
- [ ] Ler aba `Dados` no notebook.
- [ ] Ler aba `Dicionário` no notebook.
- [ ] Registrar fonte, data de extracao e data de acesso.
- [ ] Conferir quantidade inicial de linhas e colunas.
- [ ] Criar copia `df_tratado` a partir do `df_raw`.
- [ ] Preservar arquivo bruto em `dados_brutos/`.

### Diagnostico

- [ ] Gerar quantidade de valores unicos por coluna.
- [ ] Verificar duplicatas exatas.
- [ ] Verificar repeticoes de `cd_pleito`.
- [ ] Verificar repeticoes de `nu_operacao`.
- [ ] Listar categorias de `de_tipo_operacao`.
- [ ] Listar categorias de `de_fase`.
- [ ] Listar categorias de `sg_fonte`.
- [ ] Listar categorias de `nm_regiao` e `sg_uf`.
- [ ] Gerar mapa de nulos por coluna.
- [ ] Conferir tipos de dados inferidos pelo pandas.

### Limpeza

- [x] Padronizar categorias de `de_tipo_operacao`.
- [x] Padronizar categorias de `de_fase`.
- [x] Padronizar categorias de `de_esfera`.
- [x] Padronizar categorias de `nm_regiao`.
- [x] Padronizar siglas em `sg_uf` e `sg_fonte`.
- [x] Tratar campos textuais vazios como nulos.
- [x] Criar flags para campos ausentes importantes.
- [x] Converter colunas de data.
- [x] Garantir valores monetarios numericos.

### Transformacao

- [x] Criar `ano_recebimento`.
- [x] Criar `ano_primeira_cofiex`.
- [x] Criar `ano_assinatura`.
- [x] Criar `valor_total_dolar`.
- [x] Criar `percentual_contrapartida`.
- [x] Criar `prazo_recebimento_cofiex`.
- [x] Criar `prazo_cofiex_assinatura`.
- [x] Criar `prazo_assinatura_desembolso`.
- [x] Criar flags de repeticao de `cd_pleito` e `nu_operacao`.
- [x] Criar flags de ausencia para campos-chave.

### Validacao

- [x] Validar recebimento posterior a primeira COFIEX.
- [x] Validar negociacao posterior a assinatura.
- [x] Validar assinatura posterior ao desembolso.
- [x] Validar desembolso vigente anterior ao original.
- [x] Criar flag `cronologia_invalida`.
- [x] Criar relatorio de qualidade por coluna.
- [x] Comparar linhas antes e depois do tratamento.
- [ ] Validar ranges plausiveis de datas.
- [x] Validar valores negativos, nulos e zeros em campos financeiros.

### Carga

- [x] Exportar base tratada para `dados_tratados/`.
- [x] Salvar CSV com encoding `utf-8-sig`.
- [x] Testar leitura do CSV tratado.
- [ ] Conferir tipos apos exportacao.

## Colunas derivadas recomendadas

Criar no `df_tratado`:

- `ano_recebimento`
- `ano_primeira_cofiex`
- `ano_assinatura`
- `valor_total_dolar`
- `percentual_contrapartida`
- `prazo_recebimento_cofiex`
- `prazo_cofiex_assinatura`
- `prazo_assinatura_desembolso`
- `flag_cd_pleito_repetido`
- `flag_nu_operacao_repetida`
- `flag_sem_fonte`
- `flag_sem_financiamento`
- `flag_sem_contrapartida`
- `flag_sem_primeira_cofiex`
- `flag_sem_assinatura`
- `flag_cronologia_invalida`

## Regras iniciais de transformacao

- `valor_total_dolar = vl_financiamento_dolar + vl_contrapartida_dolar`, tratando nulo financeiro conforme decisao documentada.
- `percentual_contrapartida = vl_contrapartida_dolar / valor_total_dolar`, com protecao para divisao por zero.
- Prazos devem ser calculados em dias.
- Anos devem ser extraidos apenas de datas validas.
- Datas sentinela ou fora de faixa plausivel devem ser marcadas em flag antes de qualquer exclusao.
- Categorias devem ser padronizadas com `str.strip()`, normalizacao de espacos e, quando necessario, dicionarios de substituicao.

## Saidas esperadas

Saidas minimas:

- `dados_tratados/cofiex_tratado.csv`
- `dados_tratados/relatorio_qualidade_colunas.csv`

Saidas desejaveis:

- Tabela agregada por ano
- Tabela agregada por regiao/UF
- Tabela agregada por fonte financiadora
- Tabela agregada por fase
- Tabela de registros com flags de qualidade

## Padrao dos notebooks

Cada notebook deve ter:

- Titulo e objetivo.
- Celula de imports.
- Celula de configuracao de caminhos.
- Celulas pequenas e nomeadas por etapa.
- Comentarios apenas quando ajudarem a entender uma decisao.
- Conferencias impressas depois de cada bloco relevante.
- Markdown registrando decisoes de tratamento.

Sugestao de ordem:

1. `01_extracao_diagnostico.ipynb`: leitura, dicionario, shape, tipos, nulos, unicos, duplicatas e categorias.
2. `02_limpeza_transformacao.ipynb`: padronizacao, datas, numericos, flags e colunas derivadas.
3. `03_validacao_carga.ipynb`: regras de qualidade, relatorios, exportacao e leitura de teste.

## Como Codex deve auxiliar

Ao ajudar neste projeto, Codex deve:

- Responder em portugues do Brasil.
- Ler o contexto local antes de propor mudancas.
- Preservar `dados_brutos/`.
- Preferir implementacoes simples, reprodutiveis e auditaveis.
- Criar codigo Python claro para Jupyter, usando `pandas`, `numpy` e `pathlib`.
- Explicar decisoes de tratamento de dados quando houver ambiguidade.
- Sugerir validacoes antes de remover ou alterar registros suspeitos.
- Evitar criar abstracoes prematuras.
- Manter o foco em um ETL de alta qualidade, com rastreabilidade entre bruto, tratado e relatorio de qualidade.

## Definicao de pronto para o ETL

O ETL so deve ser considerado pronto quando:

- A base bruta continua intacta.
- A leitura da aba `Dados` e da aba `Dicionário` esta documentada.
- O `df_tratado` possui os mesmos registros do `df_raw`, salvo exclusoes justificadas.
- Todas as colunas de data foram convertidas ou marcadas como invalidas.
- Campos textuais vazios foram padronizados como nulos.
- Categorias principais foram revisadas.
- Colunas derivadas foram criadas e validadas.
- Flags de qualidade foram geradas.
- O relatorio de qualidade por coluna foi exportado.
- O CSV tratado foi exportado em `utf-8-sig`.
- A leitura do CSV tratado foi testada com sucesso.

## Limpezas realizadas

Notebook: `notebooks/limpeza_transformacao.ipynb`

Data de execucao: 2026-05-23

Base de entrada: `df_raw` com 3.548 linhas e 25 colunas, copiado para `df_tratado`.

### 1. Limpar espacos extras em campos textuais

Aplicado a todas as colunas de tipo `object` e `string`.

Operacoes: `.astype("string")`, `.str.strip()`, `.str.replace(r"\s+", " ", regex=True)`.

Resultado: espacos iniciais, finais e multiplos internos removidos de todos os campos textuais.

### 2. Padronizar categorias de `de_tipo_operacao`

Criada coluna `de_tipo_operacao_padronizado`.

Operacoes: strip, normalizacao de espacos e mapa de substituicao.

Mapa aplicado:

- `Contribuição Financeira não Reembolsável GEF` → `Contribuição financeira não reembolsável`
- `Contribuição financeira não reembolsável` → `Contribuição financeira não reembolsável`

Resultado: de 7 categorias para 6 categorias.

Categorias finais:

| Categoria | Registros |
|---|---:|
| Operação de crédito externo | 2.643 |
| Doações | 532 |
| Contribuição financeira não reembolsável | 288 |
| Outros | 64 |
| Operação Comercial | 12 |
| Cooperação Técnica GEF | 9 |

### 3. Padronizar categorias de `de_fase`

Criada coluna `de_fase_padronizado`.

Operacoes: strip, normalizacao de espacos e mapa de substituicao (mapa vazio, nenhuma correcao necessaria).

Resultado: 8 categorias mantidas sem alteracao. Nenhuma inconsistencia encontrada.

### 4. Padronizar categorias de `de_esfera`

Criada coluna `de_esfera_padronizado`.

Operacoes: strip, normalizacao de espacos e preenchimento de nulos com `"Não informado"`.

Resultado: 4 categorias (Estadual, Federal, Municipal, Não informado). Os 289 registros sem esfera foram categorizados explicitamente.

### 5. Padronizar categorias de `nm_regiao`

Criada coluna `nm_regiao_padronizado`.

Operacoes: strip, normalizacao de espacos e mapa de substituicao (mapa vazio, nenhuma correcao necessaria).

Resultado: 6 categorias mantidas sem alteracao.

### 6. Padronizar siglas em `sg_uf`

Criada coluna `sg_uf_padronizado`.

Operacoes: `.astype("string")`, `.str.strip()`, `.str.upper()`, `.str.replace(r"\s+", "", regex=True)`, `.fillna("NAO_INFORMADO")`.

Resultado: 28 categorias (27 UFs + `NAO_INFORMADO`). Nenhuma inconsistencia de grafia encontrada.

### 7. Padronizar siglas em `sg_fonte`

Criada coluna `sg_fonte_padronizado`.

Operacoes: `.astype("string")`, `.str.strip()`, `.str.upper()`, `.str.replace(r"\s+", " ", regex=True)`, `.fillna("NAO_INFORMADO")`.

Correcoes pontuais aplicadas na coluna original `sg_fonte` antes da padronizacao:

- Caso SECI: 1 registro (`cd_pleito` 61185) tinha `sg_fonte` nula, mas `de_fonte` continha `"Secretaria de Estado de Cooperação Internacional da Espanha - SECI"`. A sigla `"SECI"` foi preenchida na coluna original.
- Caso BID/ICSF: 1 registro (`cd_pleito` 97) usava a sigla `"BID/ICSF"` e outro (`cd_pleito` 59821) usava `"BID/ICSTF"` para o mesmo fundo (Institutional Capacity Strengthening Thematic Fund). Unificado como `"BID/ICSTF"` na coluna original.

Mapa de substituicao aplicado na coluna padronizada:

- `BID/ICSF` → `BID/ICSTF`

Resultado: 112 categorias (111 fontes + `NAO_INFORMADO`).

### 8. Tratar campos textuais vazios como nulos

Aplicado a todas as colunas de tipo `object` e `string`.

Operacao: `.replace(r"^\s*$", pd.NA, regex=True)`.

Resultado: campos que continham apenas espacos em branco ou strings vazias foram convertidos para nulos reais (`pd.NA`). Foi encontrado 1 campo adicional em `nu_operacao` que era apenas espaco em branco (total de nulos subiu de 2.145 para 2.146).

### 9. Criar flags para campos ausentes importantes

Flags criadas no `df_tratado`:

| Flag | Coluna de origem | Nulos encontrados |
|---|---|---:|
| `flag_sem_fonte` | `sg_fonte` | 283 |
| `flag_sem_financiamento` | `vl_financiamento_dolar` | 283 |
| `flag_sem_contrapartida` | `vl_contrapartida_dolar` | 1.059 |
| `flag_sem_primeira_cofiex` | `dt_primeira_cofiex` | 1.630 |
| `flag_sem_assinatura` | `dt_assinatura` | 2.132 |

Todos os valores conferidos com o diagnostico inicial da base.

### 10. Converter colunas de data para datetime

Antes da conversao, cada coluna original foi preservada em uma copia `<coluna>_original`.

Colunas convertidas com `pd.to_datetime(..., errors="coerce")`:

```text
dt_ultimo_recebimento
dt_primeira_cofiex
dt_validade_recomendacao
dt_reuniao_negociacao
dt_publicacao_dou
dt_assinatura
dt_ultimo_desembolso_original
dt_ultimo_desembolso_vigente
```

Resultado: 8 colunas convertidas para `datetime64`. Datas invalidas viraram `NaT`.

### 11. Garantir valores monetarios numericos

Aplicado a `vl_financiamento_dolar` e `vl_contrapartida_dolar` com `pd.to_numeric(..., errors="coerce")`.

Resultado: ambas mantidas como `float64`. Valores nao arredondados (existem casas decimais reais).

### 12. Flag para datas fora de faixa plausivel

Criada `flag_data_extrema` para datas validas fora da faixa 1940-01-01 a 2035-12-31.

Resultado: 7 registros marcados.

### 13. Flag para valores financeiros nulos ou zero

Criadas no `df_tratado`:

| Flag | Regra | Registros |
|---|---|---:|
| `flag_financiamento_nulo_zero` | financiamento nulo ou igual a zero | 312 |
| `flag_contrapartida_maior_financiamento` | contrapartida > financiamento (ambos validos) | 338 |

### 14. Flag para outliers financeiros

Criada `flag_outlier_financeiro`.

Criterio adotado e documentado: valor acima do percentil 99 em financiamento (1.000.000.000) OU em contrapartida (805.451.880), OU financiamento nulo/zero com contrapartida acima do percentil 99.

Resultado: 47 registros marcados. Valores preservados, apenas sinalizados.

### 15. Padronizar nm_moeda

Criada `nm_moeda_padronizado`.

Operacoes: strip, normalizacao de espacos e `fillna("Não informado")`.

Resultado: 7 categorias (Dolar, Euro, Real, SDR, Iene, Libra, Não informado). 283 nulos categorizados como `Não informado`.

### 16. Padronizar valores nulos categoricos como categoria auditavel

Criada `de_fonte_padronizado` com nulos preenchidos como `Não informado` (283 registros).

Categorias de nulo auditavel confirmadas:

| Coluna padronizada | Rotulo de nulo | Registros |
|---|---|---:|
| `de_esfera_padronizado` | Não informado | 289 |
| `sg_uf_padronizado` | NAO_INFORMADO | 26 |
| `sg_fonte_padronizado` | NAO_INFORMADO | 283 |
| `de_fonte_padronizado` | Não informado | 283 |
| `nm_moeda_padronizado` | Não informado | 283 |

### Estado do `df_tratado` apos a limpeza

- 3.548 linhas (nenhuma linha removida).
- 25 colunas originais preservadas.
- 8 colunas `<coluna>_original` de datas preservadas antes da conversao.
- 6 colunas padronizadas: `de_tipo_operacao_padronizado`, `de_fase_padronizado`, `de_esfera_padronizado`, `nm_regiao_padronizado`, `sg_uf_padronizado`, `sg_fonte_padronizado` (alem de `nm_moeda_padronizado` e `de_fonte_padronizado` criadas na transformacao).
- Flags de limpeza: `flag_sem_fonte`, `flag_sem_financiamento`, `flag_sem_contrapartida`, `flag_sem_primeira_cofiex`, `flag_sem_assinatura`, `flag_data_extrema`, `flag_financiamento_nulo_zero`, `flag_contrapartida_maior_financiamento`, `flag_outlier_financeiro`.

## Transformacoes realizadas

Notebook: `notebooks/limpeza_transformacao.ipynb` (segunda parte, apos o separador "TRANSFORMACAO").

Data de execucao: 2026-05-23

A etapa de transformacao parte do `df_tratado` ja limpo e cria tratamento de datas placeholder/sentinela e colunas derivadas. Nenhuma linha removida.

### 1. Tratar dt_ultimo_recebimento igual a 1940-01-01 como placeholder

A data `1940-01-01` aparece 382 vezes apenas em `dt_ultimo_recebimento`. Sem o placeholder, a data minima real e 1989-04-03.

Tratamento: os 382 registros viraram `NaT` e foram marcados por `flag_recebimento_placeholder`. O valor original esta preservado em `dt_ultimo_recebimento_original`.

### 2. Tratar datas extremas como sentinela

Datas claramente impossiveis foram tratadas como sentinela/erro, viraram `NaT` e foram marcadas por `flag_data_sentinela`.

Datas tratadas: `9999-12-31`, `4201-01-01`, `2228-03-29`, `2225-01-01`, `2100-01-01`.

Resultado: 6 registros marcados. Datas futuras plausiveis (ate 2052) foram mantidas, pois sao previsoes de desembolso aceitaveis.

### 3. Criar colunas de ano

Extraidas apenas de datas validas (placeholder e sentinela ja tratados como `NaT`):

- `ano_recebimento` (de `dt_ultimo_recebimento`)
- `ano_primeira_cofiex` (de `dt_primeira_cofiex`)
- `ano_assinatura` (de `dt_assinatura`)

### 4. Criar valor_total_dolar

Regra: `valor_total_dolar = vl_financiamento_dolar + vl_contrapartida_dolar`, tratando ausente como 0.

Excecao documentada: quando ambos os valores sao nulos, o resultado fica `NaN` (245 registros), para nao gerar zero artificial.

### 5. Criar percentual_contrapartida

Regra: `percentual_contrapartida = vl_contrapartida_dolar / valor_total_dolar`, expresso em fracao (0 a 1).

Protecao: resultado fica `NaN` quando `valor_total_dolar` e zero ou nulo (protecao contra divisao por zero).

### 6. Criar prazos em dias

Calculados apenas entre datas validas. Valores negativos indicam inconsistencia cronologica (tratada na validacao).

| Coluna | Calculo | Validos | Negativos |
|---|---|---:|---:|
| `prazo_recebimento_cofiex` | `dt_primeira_cofiex - dt_ultimo_recebimento` | 1.805 | 75 |
| `prazo_cofiex_assinatura` | `dt_assinatura - dt_primeira_cofiex` | 1.048 | 37 |
| `prazo_assinatura_desembolso` | `dt_ultimo_desembolso_vigente - dt_assinatura` | 1.364 | 1 |

### 7. Carga da base tratada

Ao final do notebook, o `df_tratado` foi exportado para `dados_tratados/cofiex_tratado.csv` em `utf-8-sig` e a releitura foi testada com sucesso.

## Validacoes realizadas

Notebook: `notebooks/validacao_carga.ipynb`

Data de execucao: 2026-05-23

Fluxo: o notebook le `dados_tratados/cofiex_tratado.csv` (gerado pela limpeza/transformacao), reconverte as colunas de data, aplica as regras de qualidade, cria flags finais, gera o relatorio de qualidade e regrava o CSV com as flags. Nenhuma linha removida.

### 1. Valores financeiros negativos

Verificado: 0 em financiamento e 0 em contrapartida. Criada `flag_valor_negativo` (0 registros) para auditoria futura.

### 2. Valores financeiros zerados

Criadas `flag_financiamento_zero` (29 registros) e `flag_contrapartida_zero` (254 registros).

### 3. Datas fora de ordem e cronologia

Pares cronologicos verificados (anterior <= posterior), comparados apenas quando ambas as datas sao validas:

| Verificacao | Casos |
|---|---:|
| `dt_ultimo_recebimento` > `dt_primeira_cofiex` (recebimento posterior a primeira COFIEX) | 75 |
| `dt_primeira_cofiex` > `dt_assinatura` | 37 |
| `dt_reuniao_negociacao` > `dt_assinatura` (negociacao posterior a assinatura) | 4 |
| `dt_assinatura` > `dt_ultimo_desembolso_vigente` (assinatura posterior ao desembolso) | 1 |
| `dt_ultimo_desembolso_vigente` < `dt_ultimo_desembolso_original` (vigente anterior ao original) | 8 |

Observacao: o caso "vigente anterior ao original" caiu de 10 (diagnostico inicial) para 8 porque as datas sentinela ja foram convertidas em `NaT` na transformacao e deixaram de entrar na comparacao.

### 4. flag_cronologia_invalida

Consolida qualquer inconsistencia cronologica acima (recebimento posterior a COFIEX, negociacao posterior a assinatura, assinatura posterior ao desembolso, vigente anterior ao original).

Resultado: 88 registros marcados.

### 5. Flags de repeticao

| Flag | Regra | Registros |
|---|---|---:|
| `flag_cd_pleito_repetido` | `cd_pleito` aparece em mais de uma linha | 342 |
| `flag_nu_operacao_repetida` | `nu_operacao` nao nulo repetido | conforme base |

Criada tambem `qtd_ocorrencias_cd_pleito` (numero de linhas por `cd_pleito`).

### 6. Comparar linhas antes e depois do tratamento

Confirmado: 3.548 linhas na base bruta e 3.548 no `df_tratado`. Diferenca de 0. Nenhuma linha removida em todo o ETL.

### 7. Relatorio de qualidade por coluna

Gerado `dados_tratados/relatorio_qualidade_colunas.csv` com, para cada uma das 25 colunas originais: tipo, nulos, percentual de nulos, unicos e percentual de unicos.

### Saidas geradas

| Arquivo | Conteudo |
|---|---|
| `dados_tratados/cofiex_tratado.csv` | base tratada completa (3.548 linhas, todas as colunas derivadas e flags), `utf-8-sig` |
| `dados_tratados/relatorio_qualidade_colunas.csv` | qualidade por coluna, `utf-8-sig` |

### Estado final do ETL

- Base bruta intacta em `dados_brutos/`.
- `df_tratado` com 3.548 linhas (nenhuma removida).
- Todas as colunas de data convertidas; placeholder e sentinela tratados como `NaT` com flag.
- Campos textuais vazios padronizados como nulos reais.
- Categorias principais padronizadas e nulos categoricos auditaveis.
- Colunas derivadas criadas: anos, valor total, percentual de contrapartida e prazos.
- Flags de qualidade geradas (ausencia, financeiras, datas, cronologia, repeticao).
- Relatorio de qualidade por coluna exportado.
- CSV tratado exportado em `utf-8-sig` e releitura testada com sucesso.
