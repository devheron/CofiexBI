# 🚀 Quick Start - COFIEX BI

## Coloque Tudo Funcionando em 5 Minutos

### Opção 1: Apenas Explorar (Recomendado para começo)

```
1. Abra: dashboard_cofiex.html no seu navegador
2. Explore: clique nos filtros, veja os 6 gráficos interativos
3. Leia: a seção "Sobre este projeto" (dropdown azul)
4. Pronto: você tem uma visão completa dos dados
```

**Tempo**: 5 minutos  
**Requer**: Navegador moderno (Chrome, Firefox, Safari, Edge)  
**Resultado**: Entendimento dos padrões principais de financiamento

---

### Opção 2: Usar os Dados em Suas Ferramentas

```bash
# 1. Download de dois arquivos
dados_tratados/cofiex_tratado.csv      # Base processada pronta
dados_tratados/relatorio_qualidade_colunas.csv  # Documentação de nulos/completude

# 2. Abra em Excel, Power BI, Tableau, Python/R, ou qualquer BI
#    - 3.548 linhas com 3.548 registros intactos
#    - 25 colunas originais + 14 derivadas + 9 flags de qualidade

# 3. Agregue, visualize, analise conforme necessário

# 4. Se tiver dúvidas sobre um campo específico:
#    - Consulte relatorio_qualidade_colunas.csv (qual % está vazio?)
#    - Consulte README.md seção "Dicionário de Colunas"
#    - Abra AGENTS.md para regra exata de cálculo
```

**Tempo**: 10 minutos  
**Requer**: Excel, Power BI, Tableau, ou programação (Python/R)  
**Resultado**: Análises customizadas seus seus dados/perguntas

---

### Opção 3: Rodar o Processo Completo (Entender Como Funciona)

```bash
# 1. Clone
git clone https://github.com/devheron/CofiexBI.git
cd CofiexBI

# 2. Prepare Python
python -m venv venv
source venv/bin/activate
pip install pandas numpy openpyxl jupyter

# 3. Rode os 3 notebooks (20 min total)
jupyter notebook
# Abra e rode em sequência:
# - notebooks/extracao_diagnostico.ipynb
# - notebooks/limpeza_transformacao.ipynb
# - notebooks/validacao_carga.ipynb

# 4. Veja os resultados
# - dados_tratados/cofiex_tratado.csv (base processada)
# - dados_tratados/relatorio_qualidade_colunas.csv (análise de qualidade)
```

**Tempo**: 20 minutos (notebooks) + 10 minutos leitura  
**Requer**: Python 3.8+, Jupyter, 30 minutos de paciência  
**Resultado**: Compreensão completa do ETL, capacidade de estender/adaptar

---

## 📚 Documentação (Por Nível de Detalhe)

| Você quer... | Leia isto... | Tempo | Ideal para |
|---|---|---|---|
| Visão de 30 segundos | README.md primeira página | 2 min | Gerentes, PMs |
| Responder as 10 perguntas-chave | README.md "Perguntas Principais" + "Resultados" | 10 min | Analistas, pesquisadores |
| Explorar os dados interativamente | dashboard_cofiex.html | 5 min | Todos |
| Entender cada coluna | README.md "Dicionário de Colunas" | 15 min | Analistas de dados |
| Saber como rodar | ESTRUTURA_E_USO.md "Quick Start" | 5 min | Devs |
| Entender cada decisão no ETL | AGENTS.md (completo) | 60 min | Data engineers |
| Reportar problemas | CONTRIBUTING.md | 5 min | Quem encontrar bugs |

---

## 🎯 Seus Próximos Passos

### Se Você Quer Usar os Dados em Excel/BI

```
1. Baixe: cofiex_tratado.csv
2. Abra em: Excel, Power BI, Tableau
3. Importe: com encoding UTF-8
4. Analise: agora é com você
5. Dúvidas: consulte relatorio_qualidade_colunas.csv para ver nulos/cobertura
```

### Se Você Quer Aprender a Fazer ETL

```
1. Clone o repo
2. Instale Python/Jupyter
3. Abra: notebooks/extracao_diagnostico.ipynb
4. Leia + rode a primeira célula
5. Entenda o que ela faz
6. Próxima célula...repeat
7. Quando tiver dúvidas: consulte AGENTS.md (seção relevante)
```

### Se Você Quer Estender Este Projeto

```
1. Entenda o que está aqui (rode os 3 notebooks)
2. Leia AGENTS.md completamente
3. Identifique o que você quer adicionar
4. Faça uma branch: git checkout -b feature/sua-ideia
5. Implemente (siga os padrões)
6. Teste
7. Abra PR com descrição clara
```

### Se Você Achou um Problema nos Dados

```
1. Note o número do pleito (cd_pleito)
2. Especifique: qual campo? qual linha?
3. Abra issue no GitHub com seus detalhes
4. Referencie AGENTS.md se relevante
```

---

## 📊 Estrutura de Arquivos (Resumida)

```
CofiexBI/
├── dashboard_cofiex.html          👈 Abra AGORA no navegador
├── logo_cofiex.svg                📌 Logo do projeto (use no README)
├── README.md                       📖 Documentação completa
├── ESTRUTURA_E_USO.md            🗂️  Como navegar o repo
├── CONTRIBUTING.md                💬 Como reportar/contribuir
│
├── dados_brutos/
│   └── dados_2026-04-14.xlsx     📄 Original (não toque)
│
├── dados_tratados/
│   ├── cofiex_tratado.csv         ✅ Base processada (use isto)
│   └── relatorio_qualidade_colunas.csv  📊 Análise de nulos
│
└── notebooks/
    ├── extracao_diagnostico.ipynb      (1º: diagnóstico)
    ├── limpeza_transformacao.ipynb     (2º: ETL)
    └── validacao_carga.ipynb          (3º: validação)
```

---

## 🔑 Fatos-Chave

- **Quantos dados?** 3.548 registros de financiamento
- **Qualidade?** Nenhuma linha removida, tudo flagueado e auditado
- **Período?** 2000-2024 (com concentração 2010+)
- **Valor total?** ~USD 238 bilhões
- **Fontes?** 28 instituições financiadoras
- **Regiões?** Sudeste concentra 41%, Nordeste 28%, Sul 18%

---

## ❓ Perguntas Rápidas

**P: Posso remover linhas duplicadas?**  
R: Não automaticamente. Leia AGENTS.md sobre a regra de pleitos repetidos. 158 códigos repetem-se em 342 linhas, mas não são erros.

**P: Por que o arquivo é tão grande?**  
R: Preservação completa de dados + 23 colunas derivadas + 9 flags de qualidade. Auditar é melhor que descartar.

**P: Posso usar em Power BI/Tableau?**  
R: Sim. Importe `cofiex_tratado.csv` direto, use `flag_*` para filtros inteligentes.

**P: Tem versões em outras línguas?**  
R: Não. Dados e código em português (como da fonte). Documente em português para coerência.

**P: Posso comercializar análises usando esses dados?**  
R: Sim, respeite a licença dos dados originais (dados.gov.br).

---

## 📞 Próximo Passo Sugerido

1. **Agora**: Abra `dashboard_cofiex.html` no navegador (30 segundos)
2. **Explore**: Clique nos filtros, veja as tendências (3 minutos)
3. **Leia**: Resumo executivo no README.md (5 minutos)
4. **Decida**: O que fazer (usar dados / aprender ETL / estender projeto)

---

**Bem-vindo ao COFIEX BI!** 🎉
