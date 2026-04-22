# Nuvio Tech — Dicionário de Tabelas Financeiras

**Arquivo fonte:** `Nuvio_Tech_Base_Financeira.xlsx`
**Empresa:** Nuvio Tech (SaaS B2B)
**Período:** Q4 2025 (Out–Dez/25)

---

## 1. Extrato_Bancario

**Classificação: 100% RAW DATA**

Registro transacional de movimentações bancárias da empresa em múltiplas instituições (Itaú, BTG, Inter). Contém 180 lançamentos cobrindo Out/25 a Dez/25.

| Coluna | Descrição |
|---|---|
| Data | Data da movimentação |
| Descrição | Descrição livre do lançamento bancário |
| Tipo | Meio de pagamento (TED, PIX, Boleto, Débito Automático, etc.) |
| Valor (R$) | Valor da transação (positivo = entrada, negativo = saída) |
| Saldo (R$) | Saldo acumulado após a transação |
| Ref. Interna | Código de referência interno (nem sempre presente) |
| Banco | Instituição financeira da operação |

Natureza dos dados: extraído diretamente de sistemas bancários. Nenhuma fórmula. Nenhuma agregação. Os dados incluem receitas de clientes (PIX/TED recebidos com referência a NFs), pagamentos operacionais (AWS, folha, aluguel, SaaS), transferências internas entre contas, rendimentos financeiros (CDB), e estornos.

---

## 2. Lancamentos_ERP

**Classificação: 100% RAW DATA**

Lançamentos contábeis do sistema ERP, com 200 registros no período. Difere do extrato bancário por conter dimensões contábeis (centro de custo, categoria, status de conciliação) e por registrar competência versus caixa.

| Coluna | Descrição |
|---|---|
| Data Lançamento | Data do registro no ERP |
| Data Competência | Mês/período de competência contábil (pode divergir do lançamento — há lançamentos de Out/25 com competência em Ago/25 e Set/25) |
| Descrição | Descrição do lançamento (Fornecedor - Categoria) |
| Centro de Custo | Área responsável (Engenharia, Marketing, Comercial, Produto, G&A, CS/Suporte, Infraestrutura) |
| Categoria | Classificação contábil (Receita Recorrente MRR, Receita de Setup, Receita de Consultoria, Outras Receitas, Pessoal, Benefícios, Freelancers/PJ, Infraestrutura Cloud, SaaS/Ferramentas, Impostos, etc.) |
| Valor (R$) | Valor do lançamento (positivo = receita, negativo = despesa) |
| Tipo | Receita ou Despesa |
| NF/Doc | Número da NF (receitas) ou documento (despesas) — nem sempre presente |
| Status | Estado de conciliação: Conciliado, Pendente, Em Aberto, Divergente |
| Fornecedor/Cliente | Contraparte da transação |

Natureza dos dados: extraído diretamente do ERP. Nenhuma fórmula. Os status "Divergente" e "Pendente" indicam itens que ainda não foram conciliados com o extrato bancário, o que é relevante para o fechamento contábil.

---

## 3. Receita_Clientes

**Classificação: MISTA (Raw + Construída)**

Tabela de MRR (Monthly Recurring Revenue) por cliente no Q4 2025, com 15 clientes ativos.

| Coluna | Tipo | Descrição |
|---|---|---|
| Cliente | Raw | Nome do cliente |
| Plano | Raw | Tier contratado (Starter, Growth, Enterprise) |
| MRR Contratado (R$) | Raw | Valor mensal contratado |
| Out/25 Faturado | Raw | Valor efetivamente faturado em outubro |
| Nov/25 Faturado | Raw | Valor efetivamente faturado em novembro |
| Dez/25 Faturado | Raw | Valor efetivamente faturado em dezembro |
| Churn Risk | Raw | Classificação de risco (Baixo, Médio, Alto, Churned) |
| NF Emitida Out/Nov/Dez | Raw | Status da nota fiscal por mês (Emitida, Pendente, Em atraso, N/A) |
| **Linha TOTAL (row 19)** | **Construída** | **Somatório via `=SUM()` das colunas C–F** |

O corpo da tabela (linhas 4–18) é dado raw de gestão comercial/CRM. A linha 19 (TOTAL) é derivada por fórmulas. Observações importantes: Beta SaaS churnou em Dez/25 (faturamento zerado); Delta Cloud sofreu downgrade de R$18k para R$15k em Nov/25; Mu Analytics não faturou em Out/25.

**IMPORTANTE — Escopo limitado ao MRR:** Esta aba rastreia apenas a receita recorrente mensal. Validação cruzada com o Extrato_Bancario mostra que os recebimentos reais de clientes no Q4 totalizam R$9.8M — 12.7x o MRR faturado de R$769k. Isso confirma que a empresa possui receitas significativas além do MRR (consultoria, setup, etc.) que não são monitoradas nesta aba. O ERP (R$11.5M em receitas) é consistente com o extrato (razão 0.9x), reforçando que a Receita_Clientes é uma visão parcial, não a visão completa de faturamento.

---

## 4. DRE_Gerencial

**Classificação: MAJORITARIAMENTE CONSTRUÍDA (com inputs raw)**

Demonstrativo de Resultado do Exercício gerencial (não auditado), estruturado em formato padrão GAAP/BR.

| Seção | Tipo | Descrição |
|---|---|---|
| Receita Bruta (B4:D4) | **Raw** | Input mensal digitado (Out: 442k, Nov: 435.5k, Dez: 416.5k) |
| Impostos sobre Receita (B5:D5) | **Raw** | Input mensal (alíquota ~7.5% da Receita Bruta) |
| Deduções e Devoluções (B6:D6) | **Raw** | Input mensal |
| Receita Líquida (row 7) | **Construída** | `=Receita Bruta + Impostos + Deduções` |
| COGS — Infra Cloud (B10:D10) | **Raw** | Input mensal |
| COGS — Suporte Técnico (B11:D11) | **Raw** | Input mensal |
| COGS — Ferramentas Operação (B12:D12) | **Raw** | Input mensal |
| Lucro Bruto (row 13) | **Construída** | `=Receita Líquida + COGS` |
| Margem Bruta % (row 14) | **Construída** | `=Lucro Bruto / Receita Líquida` |
| Despesas Operacionais (rows 17–25) | **Raw** | 9 linhas de input mensal (Pessoal, Benefícios, Freelancers, Marketing, Aluguel, Jurídico, Viagens, SaaS, Outros) |
| Total Desp. Operacionais (row 26) | **Construída** | `=SUM(rows 17:25)` |
| EBITDA (row 28) | **Construída** | `=Lucro Bruto + Total Desp. Operacionais` |
| Margem EBITDA % (row 29) | **Construída** | `=EBITDA / Receita Líquida` |
| D&A (row 31) | **Raw** | Input mensal (R$3k/mês constante) |
| EBIT (row 32) | **Construída** | `=EBITDA + D&A` |
| Receitas/Despesas Financeiras (rows 34–35) | **Raw** | Inputs mensais |
| Resultado Financeiro (row 36) | **Construída** | `=Rec. Financeira + Desp. Financeira` |
| Resultado Antes IR/CS (row 38) | **Construída** | `=EBIT + Resultado Financeiro` |
| IR/CS (row 39) | **Raw** | Zerado (prejuízo, sem imposto) |
| Resultado Líquido (row 40) | **Construída** | `=Resultado Antes IR + IR/CS` |
| Margem Líquida % (row 41) | **Construída** | `=Resultado Líquido / Receita Líquida` |
| Coluna Q4 2025 (col E) | **Construída** | `=SUM(Out:Dez)` para cada linha |

A DRE é majoritariamente construída por fórmulas (63 fórmulas no total). Os inputs raw são os valores mensais de cada rubrica. Os totais, subtotais, margens e agregações trimestrais são todos derivados.

**ERRO IDENTIFICADO — Receita Bruta massivamente subestimada:** A DRE registra Receita Bruta de R$442k em Out/25, valor próximo ao MRR contratado (R$442k). Porém, validação cruzada entre Extrato_Bancario e Lancamentos_ERP revela que a receita real é muito maior: o extrato mostra R$9.8M em recebimentos de clientes no Q4, e o ERP registra R$11.5M em receitas (razão ERP/Extrato de 0.9x, consistente entre si). A aba Receita_Clientes rastreia apenas MRR (R$769k faturado no Q4), mas a empresa possui receitas significativas de consultoria, setup e outras linhas. A DRE captura apenas o MRR e ignora todo o restante — subestimando a receita por um fator de ~8x. Os valores de Out R$442k, Nov R$435.5k e Dez R$416.5k são inputs manuais que refletem apenas o MRR contratado, não a receita total da empresa.

---

## 5. AP_AR

**Classificação: 100% RAW DATA**

Posição de Contas a Pagar (AP) e Contas a Receber (AR) em 31/12/2025. Contém 16 itens (7 AR + 9 AP).

| Coluna | Descrição |
|---|---|
| Tipo | AR (Accounts Receivable) ou AP (Accounts Payable) |
| Fornecedor/Cliente | Contraparte |
| Descrição | Detalhe da obrigação/direito |
| Valor (R$) | Montante |
| Vencimento | Data de vencimento |
| Status | Vencida, A Vencer, Paga |
| Dias em Atraso | Dias corridos após vencimento |
| Centro de Custo | Área responsável |
| Observação | Notas qualitativas (riscos, pendências) |

Natureza dos dados: snapshot de posição extraído do ERP/financeiro. Sem fórmulas. Os dados revelam problemas operacionais relevantes: R$197.5k em AR vencido (Beta SaaS churnou com NF aberta, Mu Analytics com NF não enviada, Delta Cloud com downgrade não refletido), e R$66k em AP vencido (freelancers sem contrato/NF, honorários contestados).

---

## 6. Caixa_Runway

**Classificação: MAJORITARIAMENTE CONSTRUÍDA (com inputs raw)**

Projeção de posição de caixa e cálculo de runway (meses de sobrevivência) da empresa.

| Linha | Tipo | Descrição |
|---|---|---|
| Saldo Inicial Out (B4) | **Raw** | R$3.2M — saldo de abertura do trimestre |
| Saldo Inicial Nov/Dez (C4, D4) | **Construído** | `=Saldo Final do mês anterior` |
| Entradas Operacionais (row 5) | **Raw** | Input mensal (Out: 420k, Nov: 398k, Dez: 365k) |
| Saídas Operacionais (row 6) | **Raw** | Input mensal (Out: -395k, Nov: -385k, Dez: -430k) |
| Fluxo de Caixa Operacional (row 7) | **Construído** | `=Entradas + Saídas` |
| Aporte Investidores (row 9) | **Raw** | Zerado no período |
| Receitas Financeiras (row 10) | **Raw** | Input mensal |
| Investimentos CAPEX (row 11) | **Raw** | Input mensal |
| Variação de Caixa (row 13) | **Construído** | `=FCO + Aporte + Rec.Fin + CAPEX` |
| Saldo Final (row 14) | **Construído** | `=Saldo Inicial + Variação` |
| Burn Rate Mensal (row 16) | **Construído** | `=-Variação de Caixa` |
| Runway em meses (row 17) | **Construído** | `=IF(Burn<=0, "N/A", Saldo Final / Burn Rate)` |
| Coluna Média Mensal (col E) | **Construído** | `=AVERAGE(Out:Dez)` para cada linha |

A aba possui 27 fórmulas. A empresa termina Dez/25 com R$3.16M de caixa. O runway só é calculável em Dez/25 (único mês com burn positivo = R$70.2k), resultando em ~45 meses. Em Out e Nov a empresa gerou caixa positivo, logo runway aparece como "N/A".

---

## Resumo: Classificação Raw vs. Construída

| Aba | Raw | Construída | Obs |
|---|---|---|---|
| **Extrato_Bancario** | 100% | — | Dados transacionais bancários brutos |
| **Lancamentos_ERP** | 100% | — | Lançamentos contábeis do ERP |
| **Receita_Clientes** | ~95% | ~5% | Corpo raw, linha de totais construída (4 fórmulas) |
| **DRE_Gerencial** | ~35% | ~65% | Inputs mensais raw; totais, subtotais e margens construídos (63 fórmulas) |
| **AP_AR** | 100% | — | Snapshot de posição de contas |
| **Caixa_Runway** | ~30% | ~70% | Saldo inicial + inputs raw; fluxo, saldo final e runway construídos (27 fórmulas) |

### Dependências entre abas

A DRE_Gerencial e Caixa_Runway **podem ser reconstruídas** a partir das abas raw, porém com ressalvas:

- A **Receita Bruta da DRE** registra apenas o MRR (~R$400–440k/mês), ignorando receitas de consultoria, setup e outras linhas que totalizam ~R$9.8M no Q4 segundo o extrato. A DRE subestima a receita real por ~8x. Os valores são inputs manuais sem vínculo com nenhuma outra aba. A DRE precisa ser reconstruída a partir do ERP e do extrato, não apenas da aba Receita_Clientes (que também é parcial — cobre só MRR).
- As **Entradas/Saídas do Caixa_Runway** deveriam derivar do Extrato_Bancario agregado, mas também são inputs manuais sem vínculo ao extrato.
- O **AP_AR** complementa o cenário fornecendo a posição de obrigações e direitos não realizados, essencial para reconciliação mas não referenciada por nenhuma outra aba.
- O **Lancamentos_ERP** e o **Extrato_Bancario** são fontes independentes que deveriam ser conciliadas entre si (a coluna "Status" no ERP indica o progresso dessa conciliação).
