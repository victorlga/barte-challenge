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

## Ordem de implementação

### Aba 1: `Extrato_Limpo`

**Fonte:** Extrato_Bancario original (180 linhas)

**O que fazer:**
- Copiar todas as colunas originais (Data, Descrição, Tipo, Valor, Saldo, Ref. Interna, Banco)
- Adicionar coluna `Categoria` com classificação baseada na Descrição:
  - `CLIENTE *` → Receita de Cliente
  - `SALÁRIOS|FOLHA` → Pessoal
  - `AWS|GOOGLE CLOUD` → Infraestrutura Cloud
  - `SLACK|GITHUB|HUBSPOT|NOTION|DATADOG` → SaaS/Ferramentas
  - `PLANO SAÚDE` → Benefícios
  - `ALUGUEL` → Aluguel e Facilities
  - `IMPOSTOS|DAS` → Impostos
  - `ENERGIA` → Utilities
  - `SEGURO` → Seguros
  - `VIAGEM` → Viagens
  - `FREELANCER|FORNECEDOR` → Freelancers/Fornecedores
  - `TRANSFERÊNCIA.*RESERVA` → Transferência Interna
  - `RENDIMENTO` → Receita Financeira
  - `ESTORNO` → Estorno
  - `MATERIAL|ANTECIPAÇÃO` e outros → Outros
- Adicionar coluna `Operacional` (Sim/Não): marcar "Não" para Transferência Interna, Receita Financeira e Estorno
- Remover coluna de saldo consolidado (é fictício — mistura 3 bancos)
- Adicionar coluna `Mês` (Out/Nov/Dez) para facilitar agregações

**O que NÃO fazer:**
- Não tentar recalcular saldo por banco (não temos saldo inicial de cada banco)
- Não corrigir o campo "Tipo" original — manter como está e usar a nova coluna `Categoria` para análises

**Verificação (ler PROBLEMAS.md e checar):**
- Problema 12 (saldo consolidado): ✅ Coluna de saldo removida, dado bruto preservado
- Problema 15 (Tipo não confiável): ✅ Nova coluna Categoria baseada em Descrição, Tipo original mantido
- Problema sobre transferências internas: ✅ Flaggadas com Operacional = Não

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

### Aba 3: `Receita_Completa`

**Fonte:** Receita_Clientes original + ERP_Limpo

**O que fazer:**
- Seção 1 — MRR (copiar Receita_Clientes original integralmente, incluindo fórmulas de total)
- Seção 2 — Receita Total por Cliente: tabela nova abaixo com:
  - Coluna Cliente
  - Coluna MRR Q4: `=SUM` dos faturados Out+Nov+Dez da Seção 1
  - Coluna Receita ERP Q4: `=SUMIFS` do ERP_Limpo filtrando Tipo="Receita" por cliente
  - Coluna Receita Extrato Q4: `=SUMIFS` do Extrato_Limpo filtrando Categoria="Receita de Cliente" por cliente (match parcial na Descrição)
  - Coluna Delta ERP vs Extrato
- Linha de TOTAL com somas
- Seção 3 — Clientes sem recebimento: lista dos 6 clientes que têm MRR faturado mas R$0 no extrato

**O que NÃO fazer:**
- Não inventar breakdown de receita por tipo (consultoria vs setup) — a categorização do ERP não é confiável
- Não alterar a tabela MRR original

**Verificação:**
- Problema 1 (DRE ignora receitas não-MRR): ✅ Receita completa visível, pronta para alimentar DRE
- Problema sobre 6 clientes sem recebimento: ✅ Identificados explicitamente

---

### Aba 4: `DRE_Corrigida`

**Fonte:** ERP_Limpo (receitas e despesas por Categoria_Corrigida) + Receita_Completa

**O que fazer:**
- Estrutura mensal (Out/Nov/Dez + Q4) mantendo formato padrão da DRE original
- **Receita Bruta**: `=SUMIFS` do ERP_Limpo, Tipo="Receita", por Mês_Competencia. Não usar Receita_Clientes (que é só MRR)
- **Impostos sobre Receita**: `=SUMIFS` do ERP_Limpo, Categoria_Corrigida="Impostos", por mês. Se não houver dados granulares, usar a alíquota de 7.5% sobre receita bruta com nota de que é estimativa
- **COGS**: `=SUMIFS` do ERP_Limpo por categorias relevantes (Infraestrutura Cloud, SaaS/Ferramentas usados em operação)
- **Despesas Operacionais**: `=SUMIFS` por cada Categoria_Corrigida (Pessoal, Benefícios, Freelancers/PJ, SaaS/Ferramentas, Aluguel, Jurídico, Viagens, Seguros, Outros)
- **Subtotais e margens**: todas fórmulas (Receita Líquida, Lucro Bruto, EBITDA, EBIT, Resultado)
- **D&A e Resultado Financeiro**: manter valores da DRE original (R$3k D&A, receitas/despesas financeiras) — não temos fonte melhor
- Coluna Q4 = `=SUM(Out:Dez)`
- Coluna de comparação com DRE original para cada linha (DRE_Gerencial!Bx)

**O que NÃO fazer:**
- Não digitar nenhum valor manualmente (exceto D&A e financeiro, que não têm fonte raw)
- Não ignorar meses de competência fora do Q4 que existem no ERP — decidir: usar Data Lançamento ou Data Competência. **Usar Data Competência** quando dentro do Q4, e **ignorar** lançamentos com competência fora do Q4 (registrar separadamente como "competência retroativa")

**Verificação:**
- Problema 1 (DRE subestima receita em ~8x): ✅ Receita derivada do ERP completo
- Problema 11 (sem cross-sheet): ✅ Toda a DRE é fórmula referenciando ERP_Limpo
- Problema 13 (DRE manual): ✅ Totalmente automatizada
- Problema sobre alíquota fixa: ⚠️ Se não houver dados reais de impostos, documentar premissa

---

### Aba 5: `Caixa_Corrigido`

**Fonte:** Extrato_Limpo

**O que fazer:**
- **Saldo Inicial Out/25**: usar o saldo implícito do extrato (R$2,850,000) — derivar por fórmula: `=Extrato_Limpo primeira linha Saldo - primeira linha Valor`
- **Entradas Operacionais**: `=SUMIFS` do Extrato_Limpo, Valor > 0, Operacional = "Sim", por Mês
- **Saídas Operacionais**: `=SUMIFS` do Extrato_Limpo, Valor < 0, Operacional = "Sim", por Mês
- **Transferências Internas** (linha separada, informativa): `=SUMIFS` com Categoria = "Transferência Interna"
- **Receitas Financeiras**: `=SUMIFS` com Categoria = "Receita Financeira"
- **Estornos** (linha separada): `=SUMIFS` com Categoria = "Estorno"
- **Saldo Final**: `=Saldo Inicial + Entradas + Saídas + Financeiras + Estornos` (transferências internas excluídas)
- **Saldo Inicial meses seguintes**: `=Saldo Final do mês anterior`
- **Burn Rate**: `=-(Entradas Operacionais + Saídas Operacionais)` por mês
- **Runway**: `=IF(Burn<=0, "N/A", Saldo Final / Burn)` — manter lógica, mas agora com dados reais
- Coluna de comparação com Caixa_Runway original

**O que NÃO fazer:**
- Não incluir transferências internas no cálculo de entradas/saídas operacionais
- Não usar saldo da aba original (R$3.2M) — usar o derivado do extrato

**Verificação:**
- Problema 2 (Caixa descolado do extrato): ✅ Derivado 100% do extrato
- Problema 12 (saldo consolidado): ⚠️ Ainda consolidado (não temos saldo por banco), mas agora pelo menos é matematicamente correto
- Problema sobre runway enganoso: ✅ Runway calculado com dados reais

---

### Aba 6: `AP_AR_Anotado`

**Fonte:** AP_AR original (16 linhas)

**O que fazer:**
- Copiar tabela original integralmente
- Adicionar coluna `Ação Recomendada`:
  - Beta SaaS (AR, churned, R$38k): "Acionar jurídico para cobrança — cliente churnou"
  - Delta Cloud (AR, R$18k, NF errada): "Emitir NF corrigida (R$15k) e negociar diferença"
  - Mu Analytics (AR, R$19k, NF não enviada): "Enviar NF imediatamente"
  - Gamma Digital (AR, R$15k, setup): "Enviar cobrança de setup"
  - Pi Logistics (AR, R$7.5k, churn risk): "Contato comercial urgente + cobrança"
  - Freelancer A (AP, sem contrato): "Regularizar contrato antes de pagar"
  - Freelancer B (AP, sem NF): "Solicitar NF ao fornecedor"
  - Silva & Advogados (AP, contestada): "Revisar contrato e alinhar valor"
- Adicionar coluna `Prioridade` (Alta/Média/Baixa)
- Adicionar coluna `Recebido_no_Extrato` (para AR): cruzar com Extrato_Limpo para verificar se o cliente tem recebimentos — fórmula COUNTIFS

**O que NÃO fazer:**
- Não alterar valores ou status originais
- Não adicionar novos itens ao AP/AR (não temos a posição completa)

**Verificação:**
- Problema 6 (AR vencido sem ação): ✅ Ações recomendadas por item
- Problema 8 (freelancers sem contrato): ✅ Flaggado com ação
- Problema 9 (13o com atraso): ✅ Preservado, visível

---

### Aba 7: `Dashboard`

**Fonte:** Todas as abas anteriores

**O que fazer:**
- Seção "Receita Real vs. Reportada": tabela comparando DRE original vs. DRE corrigida (receita, EBITDA, margem)
- Seção "Saúde do Caixa": saldo atual, burn rate real, runway real
- Seção "Alertas": contagem de flags (NFs duplicadas, despesas sem doc, lançamentos não conciliados, AR vencido)
- Tudo via fórmulas cross-sheet

**O que NÃO fazer:**
- Não fazer gráficos (foco em dados limpos, não em apresentação — isso fica para o PPT)

---

## Problemas que NÃO serão resolvidos na planilha

Estes problemas são operacionais/de processo e não se resolvem corrigindo dados:

- **Problema 8 (freelancers sem contrato):** Flaggado, mas resolver requer ação jurídica real
- **Problema 9 (13o com atraso):** Fato consumado, não há correção retroativa
- **Problema 10 (52% não conciliados):** Conciliação real requer match manual entre extrato e ERP, item a item — pode ser um próximo passo, mas não é escopo desta planilha
- **Problema 7 (NFs pendentes/atraso):** Problema de processo de faturamento, não de dados

Esses problemas serão endereçados no material de apresentação como recomendações operacionais.
