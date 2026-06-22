# Guia de Uso do Repositório

## 🚀 Quick Start

### Para Explorar os Dados Rapidamente

1. **Abra o Dashboard**
   ```
   Clique em dashboard_cofiex.html no seu navegador
   ```
   Você verá gráficos interativos sobre financiamentos, distribuição geográfica e tendências temporais sem precisar instalar nada.

2. **Leia o Resumo Executivo**
   ```
   Veja as seções "Resultados Principais" e "Achados Principais" no README.md
   ```
   Responde às dez questões-chave em 5 minutos de leitura.

### Para Rodar o ETL Completo

1. **Clone o repositório**
   ```bash
   git clone https://github.com/devheron/CofiexBI.git
   cd CofiexBI
   ```

2. **Configure o ambiente**
   ```bash
   python -m venv venv
   source venv/bin/activate  # Linux/Mac
   pip install pandas numpy openpyxl jupyter matplotlib seaborn
   ```

3. **Execute os notebooks em sequência**
   ```bash
   jupyter notebook
   ```
   Abra na ordem:
   - `notebooks/extracao_diagnostico.ipynb` (20 min) — Conheça os dados brutos
   - `notebooks/limpeza_transformacao.ipynb` (15 min) — Veja o ETL em ação
   - `notebooks/validacao_carga.ipynb` (10 min) — Valide a integridade final

4. **Acesse os resultados**
   - Base tratada: `dados_tratados/cofiex_tratado.csv`
   - Relatório de qualidade: `dados_tratados/relatorio_qualidade_colunas.csv`

---

## 📂 O Que Está Onde

### `dashboard_cofiex.html`
Dashboard profissional interativo. Abra no navegador, explore filtros, veja gráficos em tempo real. Nenhuma instalação necessária.

**O que traz:**
- 6 gráficos: Timeline, tipos de operação, distribuição regional, esfera pública, fontes e fases
- 4 cards de resumo: Total de registros, pleitos únicos, financiamento agregado, fontes ativas
- Seção "Bio" com dropdown explicando o projeto
- Lista das 10 questões respondidas com formulação original

### `dados_brutos/dados_2026-04-14.xlsx`
Arquivo original da COFIEX extraído em 14/04/2026. 3.548 linhas, 25 colunas, sem modificações. Abas: `Dados` e `Dicionário`.

Quando você quiser:
- Comparar com a base antes e depois
- Verificar valores originais
- Entender o estado inicial dos dados

### `dados_tratados/cofiex_tratado.csv`
A base "pronta para análise". Mesmas 3.548 linhas, 25 colunas originais + 14 derivadas (anos, prazos, totais) + 9 flags de qualidade. Sem linhas removidas.

Ideal para:
- Importar em ferramentas BI (Power BI, Tableau, Qlik)
- Rodas análises secundárias em R, SAS ou ferramentas de dados
- Validar contra a bruta linha por linha

### `dados_tratados/relatorio_qualidade_colunas.csv`
Tabla com tipo, nulos, percentual de nulos e únicos para cada coluna. 25 linhas, uma por coluna original.

Para:
- Entender completude dos dados
- Decidir quais colunas usar em cada análise
- Documentar limitações de cobertura

### `notebooks/extracao_diagnostico.ipynb`
Notebook que lê a bruta e faz diagnóstico inicial.

**Output:**
- Quantas linhas? Quantas colunas? Que tipos?
- Quantos nulos em cada coluna?
- Quantos valores únicos?
- Há linhas exatas duplicadas? (Resposta: 0)
- Há códigos de pleito repetidos? (Resposta: 158 códigos em 342 linhas)

**Tempo**: 20 minutos rodando

**Próximo passo**: Leia o output, considere as implicações (não remova linhas, flagueias), depois passe para limpeza.

### `notebooks/limpeza_transformacao.ipynb`
Notebook que executa todo o ETL.

**Fases:**
1. Reconversão de tipos (datas de serial numbers, monetários de texto)
2. Tratamento de placeholders e sentinelas em datas
3. Padronização de texto (strip, case, espaços)
4. Cálculo de colunas derivadas (anos, prazos, totais, percentuais)
5. Criação de flags (duplicação, nulos, outliers, cronologia)
6. Export para CSV

**Output:**
- `dados_tratados/cofiex_tratado.csv` — base processada
- Mensagens de resumo no notebook

**Tempo**: 15 minutos rodando

**Próximo passo**: A base está pronta. Vá para validação antes de usar em análises críticas.

### `notebooks/validacao_carga.ipynb`
Notebook que valida a integridade pós-ETL.

**Verificações:**
- Valores financeiros negativos? (0 encontrados)
- Valores zerados? (Sim, flagueados)
- Cronologia válida? (88 inconsistências encontradas, flagueadas)
- Repetições confirmadas? (Sim, 342 linhas flagueadas)
- Releitura do CSV tratado funciona? (Sim ✓)

**Output:**
- Flags finais adicionadas ao CSV
- `dados_tratados/relatorio_qualidade_colunas.csv` — análise por coluna

**Tempo**: 10 minutos rodando

**Próximo passo**: Confira os números, leia AGENTS.md para entender cada flag. A base está pronta para BI.

### `AGENTS.md`
Documentação técnica de 1.500+ linhas. Leia se:
- Precisar entender por que uma decisão foi tomada
- Estiver implementando análises que usem flags
- Quiser saber a exata definição de cada coluna derivada
- For integrar esses dados em um data warehouse

Seções:
- Contexto e pergunta central
- Regras de contagem de pleitos repetidos (crucial)
- Especificação de cada flag
- Estado final do ETL com números
- Decisões de design explicadas

**Não é necessário ler para usar o dashboard ou a base tratada.** É referência para analistas e engenheiros de dados.

---

## 🎯 Fluxos de Uso Recomendados

### Cenário 1: Exploração Rápida (5 minutos)
```
dashboard_cofiex.html → leia resumo no README → visto!
```

### Cenário 2: Análise com Seus Dados (30 minutos)
```
Download de cofiex_tratado.csv
→ Importe em Excel, Power BI ou Python/R
→ Crie visualizações suas
→ Consulte relatorio_qualidade_colunas.csv se tiver dúvidas sobre nulos
```

### Cenário 3: Entender o Processo (1-2 horas)
```
Leia README.md completamente
→ Abra extracao_diagnostico.ipynb e rode tudo
→ Leia AGENTS.md seções relevantes
→ Abra limpeza_transformacao.ipynb e acompanhe cada decisão
```

### Cenário 4: Validação Científica (2-3 horas)
```
Todos os notebooks em sequência
→ Compare bruta com tratada lado a lado
→ Valide cada flag manualmente em amostras
→ Leia AGENTS.md completo
→ Documente qualquer discrepância encontrada
```

---

## ❓ Perguntas Frequentes

### "Posso remover linhas duplicadas?"

**Não recomendado sem análise prévia.** 158 códigos de pleito aparecem em 342 linhas, mas não são duplicações exatas. Podem ser operações diferentes, fases diferentes, ou atualizações administrativas. Use `flag_cd_pleito_repetido` para filtrar se necessário, mas documente sua decisão.

A base foi mantida integralmente (nenhuma linha removida) justamente para permitir essa flexibilidade.

### "Quanto está faltando nos dados?"

Veja `relatorio_qualidade_colunas.csv`. Alguns destaques:
- 283 registros (8%) sem informação de fonte
- 289 registros (8%) sem informação de esfera
- 26 registros (<1%) sem UF
- 312 registros (9%) sem valor de financiamento

Essas lacunas são normais em dados públicos e foram preservadas (não zeradas) para auditoria.

### "Posso usar essa base em um data warehouse?"

Sim. Importe `cofiex_tratado.csv` diretamente. Cada linha é um registro administrivo único. As flags permitem filtros inteligentes e auditoria. Considere criar uma tabela dimensional de pleitos de forma denormalizada se precisar agregar por cd_pleito único.

### "Como reportar problemas nos dados?"

Abra uma issue no GitHub com:
- Linha específica (forneça cd_pleito)
- Campo afetado
- Valor encontrado vs. esperado
- Justificativa

O AGENTS.md explica cada decisão, então referenças a essa seção ajudam.

### "Posso estender esse ETL para dados novos?"

Sim. A lógica é modular. Copie a estrutura do notebook `limpeza_transformacao.ipynb` para:
1. Reconverter tipos
2. Aplicar as mesmas regras de data e moeda
3. Gerar as mesmas colunas derivadas
4. Replicar as flags

Documente qualquer desvio em um novo AGENTS_extensao.md.

---

## 📊 Estrutura de Dados

### Granularidade

1 linha = 1 registro administrativo da COFIEX, conforme aparece no arquivo original.

Isso significa que um mesmo pleito (cd_pleito) pode aparecer várias vezes se tiver sido registrado em múltiplas operações, fases ou fontes. **Isso é proposital e auditado.**

### Agregações Recomendadas

- **Por ano**: Use colunas `ano_*` derivadas
- **Por região**: Use `nm_regiao_padronizado`
- **Por fonte**: Use `sg_fonte_padronizado` (código) ou `de_fonte_padronizado` (descrição)
- **Por pleito único**: Use `cd_pleito` com `flag_cd_pleito_repetido` para contextualizar
- **Por valor**: Sempre some `vl_financiamento_dolar + vl_contrapartida_dolar`

Evite agregar por cd_pleito sem considerar:
- Quantas linhas ele tem (`qtd_ocorrencias_cd_pleito`)
- Se as linhas representam operações diferentes ou duplicação

### Moedas e Conversão

Todas as colunas financeiras no CSV estão em dólar ou equivalente USD (`vl_financiamento_dolar`, `vl_contrapartida_dolar`).

Se precisar da moeda original, use `nm_moeda_padronizado`. A base preserva: Dólar, Euro, Real, SDR, Iene, Libra, Não informado.

Taxas de conversão **não estão** nesta base (seria suposição). Se precisar, aplique suas próprias taxas históricas.

---

## 🔐 Integridade de Dados

Nenhuma linha foi removida em nenhum estágio do ETL. 3.548 entrada → 3.548 saída.

Anomalias foram flagueadas, não descartadas:
- Valores nulos e zero → `flag_*_nulo_zero`
- Datas extremas → `flag_data_extrema`
- Cronologia inválida → `flag_cronologia_invalida`
- Outliers → `flag_outlier_financeiro`
- Repetições → `flag_cd_pleito_repetido`

Isso permite:
- Auditoria linha por linha
- Decisões downstream informadas (você escolhe o que fazer)
- Rastreabilidade completa

Consulte AGENTS.md seção "Validações Realizadas" para números exatos de cada flag.

---

## 📞 Suporte

Dúvidas sobre:
- **Dados**: Veja relatorio_qualidade_colunas.csv
- **Metodologia**: Veja AGENTS.md
- **Dashboard**: Abra no navegador, clique em filtros
- **Reprodução**: Rode os notebooks em sequência

Para problemas não documentados, abra uma issue no GitHub com detalhes e um mínimo reproduzível.
