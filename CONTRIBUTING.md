# Contribuindo para o COFIEX BI

## Reportando Problemas nos Dados

Encontrou um valor que não faz sentido? Uma data fora de ordem? Valores financeiros inconsistentes?

**Abra uma issue no GitHub com:**

1. **Título claro**
   ```
   Erro de cronologia: cd_pleito XXX tem assinatura posterior ao desembolso
   ```

2. **Contexto**
   - Qual campo está afetado?
   - Qual linha específica? (forneça o cd_pleito se possível)
   - O problema está na bruta ou na tratada?

3. **Detalhes**
   - Valor encontrado
   - O que você esperava
   - Por que isso é um problema

4. **Exemplo mínimo**
   ```
   cd_pleito: 1234567
   dt_assinatura: 2020-05-15
   dt_ultimo_desembolso_vigente: 2020-03-10
   
   Resultado: Assinatura 66 dias após desembolso (cronologia inválida)
   flag_cronologia_invalida: 1 (marcada corretamente)
   
   Questão: Esse padrão é administração normal ou erro de registro?
   ```

---

## Sugerindo Análises Novas

Tem uma pergunta que gostaria que o dashboard ou notebooks respondessem?

**Crie uma issue com label `enhancement`:**

```
Título: Análise de Concentração por Proponente

Descrição:
Gostaria de ver os top 20 proponentes por valor de financiamento.
Seria útil para entender se a carteira está muito concentrada em poucas 
organizações ou distribuída.

Metodologia Sugerida:
- Agregar vl_financiamento_dolar + vl_contrapartida_dolar por nm_proponente
- Ranking descendente
- Gráfico de barras horizontal com percentual acumulado
```

---

## Corrigindo Erros ou Melhorando Código

Se você:
- Encontrou um bug no notebook
- Quer melhorar a documentação
- Quer adicionar validações novas

**Faça um Pull Request:**

1. **Fork o repositório**
   ```bash
   git clone https://github.com/seu-usuario/CofiexBI.git
   cd CofiexBI
   git checkout -b melhoria/descricao-curta
   ```

2. **Faça suas alterações**
   - Edite o arquivo (notebook, código, documentação)
   - Teste localmente
   - Certifique-se de que não quebrma nada

3. **Commit com mensagem clara**
   ```bash
   git add .
   git commit -m "Corrige flag_cronologia_invalida para casos de recebimento posterior"
   ```

4. **Push e abra PR**
   ```bash
   git push origin melhoria/descricao-curta
   ```

5. **Descreva o PR**
   - O que mudou?
   - Por que é necessário?
   - Como foi testado?
   - Referência a issues relacionadas

---

## Padrões de Código

### Notebooks

- Use seções com markdown `# Titulo`, `## Subtítulo`
- Comente decisões não óbvias
- Sempre execute de cima para baixo sem erros
- Reinicie o kernel antes de fazer commit

### Documentação

- Use markdown simples, sem HTML embutido
- Links relativos para arquivos do repo: `[AGENTS.md](AGENTS.md)`
- Links externos: formato padrão `[texto](url)`
- Nenhuma jargão sem explicação (ou expanda na próxima frase)

### CSV e Dados

- UTF-8 com BOM (`utf-8-sig`) para compatibilidade Excel
- Sem quebras de linha dentro de células
- Sem caracteres de controle
- Headers em português (conforme original)

---

## Processo de Revisão

Um PR será revisor se:

✅ Sem conflitos com `main`  
✅ Mudanças bem descritas e justificadas  
✅ Testes passam (notebooks rodando)  
✅ Documentação atualizada se necessário  

⚠️ Não serão aprovados:

❌ PRs que removem dados sem justificativa documentada  
❌ Mudanças que quebram reprodutibilidade  
❌ Alterações em cofiex_tratado.csv sem recalcular tudo  

---

## Dúvidas?

Consulte:
- [README.md](README.md) — visão geral
- [ESTRUTURA_E_USO.md](ESTRUTURA_E_USO.md) — como rodar e usar
- [AGENTS.md](AGENTS.md) — especificações técnicas completas

Se ainda tiver dúvidas, abra uma issue com label `question`.

---

## Código de Conduta

Somos uma comunidade de analistas de dados. Todos ganham com:
- Respeito mútuo
- Feedback construtivo
- Documentação clara
- Compartilhamento de conhecimento

Qualquer comportamento abusivo, discriminatório ou desrespeitoso resultará em banimento sem aviso.

---

**Obrigado por contribuir para melhores análises de dados públicos!** 🙌
