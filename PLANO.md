# Plano — Nova Planilha Financeira Nuvio Tech

**Objetivo:** Criar `Nuvio_Tech_Corrigida.xlsx` com dados limpos e relatórios derivados confiáveis.
**Executor:** Claude Code (uma aba por vez, em ordem)
**Referência de problemas:** `PROBLEMAS.md`

---

## Princípios

1. **Não inventar dados.** Se a informação correta não existe, sinalizar com flag — não preencher com chute.
2. **Derivar, não digitar.** Toda célula que puder ser fórmula deve ser fórmula. Zero input manual em abas derivadas.
3. **Cross-sheet obrigatório.** Abas derivadas referenciam abas raw. Nada de copiar/colar valores.
4. **Ordem de construção = ordem de dependência.** Raw primeiro, derivadas depois.

---

### Aba 2: `ERP_Limpo`

**Fonte:** Lancamentos_ERP original (200 linhas)

**O que fazer:**
- Copiar todas as colunas originais
- Adicionar coluna `Categoria_Corrigida`: reclassificar com base no Fornecedor/Cliente (não na categoria original). Regras:
  - AWS → Infraestrutura Cloud (sempre)
  - Google Cloud → Infraestrutura Cloud
  - GitHub → SaaS/Ferramentas
  - Slack → SaaS/Ferramentas
  - HubSpot → SaaS/Ferramentas
  - Notion → SaaS/Ferramentas
  - Datadog → SaaS/Ferramentas
  - ContaAzul → SaaS/Ferramentas
  - WeWork → Aluguel e Facilities
  - Porto Seguro → Seguros
  - Silva & Advogados → Jurídico e Contabilidade
  - Unimed → Benefícios
  - Freelancer A, Freelancer B → Freelancers/PJ
  - Para receitas: manter a Categoria original (MRR, Consultoria, Setup, Outras) — não temos como saber qual é correta, mas o fato de existirem é legítimo
- Adicionar coluna `Flag_Competencia`: "DIVERGENTE" se Data Competência difere do mês de Data Lançamento, vazio caso contrário
- Adicionar coluna `Flag_NF_Duplicada`: se o NF/Doc aparece em mais de um lançamento com cliente diferente, marcar "DUPLICADA"
- Adicionar coluna `Flag_Sem_Doc`: "SEM DOCUMENTO" se NF/Doc é vazio em lançamento de despesa
- Adicionar coluna `Mês_Competencia` (formato Out/Nov/Dez baseado em Data Competência)

**O que NÃO fazer:**
- Não corrigir NFs duplicadas (não sabemos a NF correta)
- Não corrigir datas de competência (não sabemos a data correta)
- Não excluir lançamentos — tudo fica, com flags

**Verificação:**
- Problema 3 (categorias absurdas): ✅ Categoria_Corrigida baseada em fornecedor
- Problema 4 (NFs duplicadas): ✅ Flaggadas, não corrigidas
- Problema 5 (despesas sem doc): ✅ Flaggadas
- Problema 14 (competência divergente): ✅ Flaggadas
- Problema 10 (52% não conciliados): ⚠️ Preservado, sem alteração (correção requer conciliação manual real)

---

## Problemas que NÃO serão resolvidos na planilha

Estes problemas são operacionais/de processo e não se resolvem corrigindo dados:

- **Problema 8 (freelancers sem contrato):** Flaggado, mas resolver requer ação jurídica real
- **Problema 9 (13o com atraso):** Fato consumado, não há correção retroativa
- **Problema 10 (52% não conciliados):** Conciliação real requer match manual entre extrato e ERP, item a item — pode ser um próximo passo, mas não é escopo desta planilha
- **Problema 7 (NFs pendentes/atraso):** Problema de processo de faturamento, não de dados

Esses problemas serão endereçados no material de apresentação como recomendações operacionais.