# Nuvio Tech — Diagnóstico de Problemas na Base Financeira

**Arquivo:** `Nuvio_Tech_Base_Financeira.xlsx`
**Data da análise:** 2026-04-21
**Método:** Revisão manual + scripts Python de validação cruzada

---

## Overview

A base financeira da Nuvio Tech apresenta problemas graves em praticamente todas as dimensões: dados transacionais mal-estruturados, categorizações absurdas no ERP, DRE com receita superestimada, fluxo de caixa completamente descolado da realidade do extrato bancário, e controles de cobrança fragilizados. Os problemas não são pontuais — revelam a ausência de processos financeiros básicos, consistente com o contexto de uma empresa cujo financeiro era tocado pelo CEO sem equipe dedicada.

Os problemas se dividem em três camadas:

**Camada 1 — Dados raw corrompidos:** O ERP tem categorização aleatória (38 de 200 lançamentos com categorias absurdas), NFs duplicadas entre clientes diferentes (19 NFs), e 43% dos lançamentos com competência divergente do lançamento. O extrato bancário mistura três bancos em uma coluna de saldo única, impossibilitando reconciliação por banco.

**Camada 2 — Relatórios construídos incorretamente:** A DRE registra apenas MRR (~R$440k/mês), mas o extrato bancário mostra R$9.8M em recebimentos de clientes no Q4 — a receita real é ~8x maior que a reportada. O Caixa_Runway tem saldo inicial R$350k acima do extrato e entradas/saídas com diferenças de ordem de grandeza versus o extrato real.

**Camada 3 — Ausência de controles:** R$97.5k em contas a receber vencidas sem ação, R$4.6M em despesas sem documento no ERP, NFs pendentes ou em atraso para 10 dos 15 clientes, e nenhuma fórmula cross-sheet ligando as abas entre si.

---

## 1. Extrato_Bancario

### Problemas quantitativos

**Saldo consolidado de 3 bancos em coluna única.** O extrato mistura transações de Itaú (110), BTG (43) e Inter (27) com uma única coluna de saldo sequencial. Uma transação no Itaú é seguida por uma no BTG, e o saldo simplesmente continua como se fosse uma conta só. Isso torna impossível reconciliar o saldo por banco individualmente. Na prática, esse "saldo" é fictício — não representa o saldo real de nenhuma conta.

**Saldo implícito inicial (R$2.85M) diverge do Caixa_Runway (R$3.2M).** O saldo antes da primeira transação de Out/25, derivado do extrato, é de R$2,850,000. A aba Caixa_Runway registra R$3,200,000 como saldo inicial. Diferença de R$350,000 sem explicação.

### Problemas qualitativos

**Tipo de transação inconsistente com a descrição.** 24 transações são classificadas como "Tarifa Bancária", mas incluem pagamentos como "Salários e Encargos - Folha" (R$224k), "AWS Services - Infraestrutura" (R$84k), "Impostos DAS/Simples" (R$36k), e "Slack - Licença Enterprise" (R$30k). Nenhuma dessas é uma tarifa bancária — são pagamentos operacionais normais. O campo "Tipo" não é confiável para classificação automática.

**18% das transações sem referência interna.** 32 de 180 lançamentos não possuem "Ref. Interna", dificultando a conciliação com o ERP e com contas a pagar/receber.

**Transferências internas não eliminadas.** 9 transferências entre contas próprias (R$1.77M em saídas) estão no extrato sem marcação clara, podendo inflar artificialmente os volumes de saída se usadas para cálculos agregados sem filtro.

---

## 2. Lancamentos_ERP

### Problemas quantitativos

**43% dos lançamentos com competência divergente do lançamento (86 de 200).** O lag médio é de 38 dias e chega a 60 dias. Há lançamentos de outubro com competência em agosto, o que indica regime de competência completamente descontrolado — ou lançamentos retroativos sem critério.

**19% dos lançamentos com categorização absurda (38 de 200).** Fornecedores são atribuídos a categorias que não fazem sentido algum. Exemplos:

- AWS classificada como "Viagens" (R$143k), "Aluguel e Facilities" (R$241k), "Marketing e Eventos" (R$141k) — AWS é infraestrutura cloud
- Google Cloud como "Freelancers/PJ" (R$244k), "Viagens" (R$242k), "Pessoal" (R$137k)
- Silva & Advogados como "Infraestrutura Cloud" (R$146k) e "SaaS/Ferramentas" (R$99k) — é um escritório de advocacia
- Unimed como "Infraestrutura Cloud" (R$215k) — Unimed é plano de saúde
- Notion como "Impostos" (R$200k) — Notion é uma ferramenta SaaS
- Freelancer A como "Impostos" (R$198k) e "Infraestrutura Cloud" (R$129k)

Isso invalida qualquer análise de despesa por categoria feita a partir do ERP.

**21 NFs duplicadas, 19 delas usadas para clientes diferentes.** Exemplos: NF-1050 aparece 3 vezes para Delta Cloud, Epsilon Data e Alpha Tech. NF-1084 aparece 4 vezes para Alpha Tech, Delta Cloud, Gamma Digital e Epsilon Data. Isso pode indicar dados fabricados, erros de digitação, ou reuso acidental de números de NF.

**R$4.6M em despesas sem documento (NF/Doc).** 34 lançamentos de despesa (17% do total) não têm NF ou documento de suporte, somando R$4,627,010.94. Isso é um risco de compliance grave — despesas sem comprovante não são dedutíveis e não sobrevivem a uma auditoria.

### Problemas qualitativos

**52% dos lançamentos não estão conciliados.** Apenas 96 de 200 (48%) têm status "Conciliado". Os demais são: Pendente (48), Divergente (31), Em Aberto (25). Um terço do ERP está em estado que requer ação.

**Receitas em centros de custo errados.** 11 lançamentos de receita estão atribuídos a centros como "Engenharia" e "G&A" ao invés de "Comercial". Exemplo: Theta Labs MRR em "Engenharia", Kappa Pay Consultoria em "G&A". Isso distorce qualquer análise de P&L por departamento.

**ERP registra 4 categorias de receita — e o extrato confirma que a receita é de fato muito maior que o MRR.** O ERP contém: Receita Recorrente MRR (R$2.6M), Receita de Consultoria (R$3.9M), Receita de Setup (R$2.5M) e Outras Receitas (R$2.4M), totalizando R$11.5M no Q4. A aba Receita_Clientes registra apenas R$769k de MRR faturado — mas o extrato bancário mostra R$9.8M em recebimentos reais de clientes. A razão ERP/Extrato é 0.9x, o que indica que os valores do ERP são consistentes com o caixa real. A empresa de fato tem receitas significativas além do MRR. Porém, a categorização entre MRR, consultoria, setup e outras permanece não confiável — dado o mesmo padrão de categorização aleatória das despesas (mesmo cliente em 3–4 categorias).

**6 dos 15 clientes não aparecem no extrato bancário.** Lambda Fin, Mu Analytics, Nu Robotics, Omicron Health, Pi Logistics e Rho Security têm MRR faturado na aba Receita_Clientes mas nenhum recebimento no extrato no Q4. Isso totaliza R$494k em MRR faturado sem correspondência no caixa — pode indicar inadimplência não rastreada ou recebimentos registrados sem identificação de cliente.

---

## 3. Receita_Clientes

### Problemas quantitativos

**Aba cobre apenas MRR, mas a empresa tem receitas ~12.7x maiores.** Validação cruzada por script Python:

| Fonte | Total Q4 | Ratio vs Extrato |
|---|---|---|
| Receita_Clientes (MRR faturado) | R$769k | 0.08x |
| Lancamentos_ERP (todas receitas) | R$11.5M | 1.2x |
| Extrato_Bancario (recebimentos clientes) | R$9.8M | 1.0x (referência) |

A aba Receita_Clientes rastreia apenas MRR. A empresa possui receitas de consultoria, setup e outras linhas que não aparecem aqui. Isso não é um erro da aba em si — é uma limitação de escopo. O problema é que a DRE usa essa aba como se fosse a visão completa de receita.

**6 de 15 clientes sem recebimento no extrato.** Lambda Fin, Mu Analytics, Nu Robotics, Omicron Health, Pi Logistics e Rho Security somam R$494k em MRR faturado no Q4 mas não aparecem no extrato bancário. Possível inadimplência não rastreada.

**DRE usa valores próximos ao MRR contratado, não ao faturado.** Mesmo considerando apenas o MRR, a DRE de Out/25 (R$442k) corresponde ao MRR contratado, não ao faturado (R$423k — Mu Analytics não faturou R$19k). Nos demais meses os valores também divergem (Nov: DRE R$435.5k vs. faturado R$439k; Dez: DRE R$416.5k vs. faturado R$401k).

### Problemas qualitativos

**10 de 15 clientes com NFs pendentes ou em atraso.** Detalhamento:

- Alpha Tech (Enterprise, R$45k/mês): NF pendente nos 3 meses do Q4
- Rho Security (Enterprise, R$50k/mês): NF pendente em Out, em atraso Nov e Dez
- Nu Robotics (Enterprise, R$60k/mês): NF em atraso Nov, pendente Dez
- Lambda Fin: NF em atraso Nov e Dez
- Beta SaaS: NF em atraso Nov (antes de churnar)

Isso significa que o faturamento não é emitido em dia, o que atrasa recebimento e prejudica o fluxo de caixa.

**Beta SaaS churnou mas ainda tem R$38k em AR aberto.** O cliente saiu em Dez/25 (faturamento zerado), mas a NF de Nov/25 (R$38k) está vencida há 46 dias sem ação de cobrança. Probabilidade de recuperação diminui com o tempo, e churn sem encerramento formal é um problema de processo.

**Delta Cloud com downgrade não refletido.** MRR contratado é R$18k, mas a partir de Nov/25 o faturado caiu para R$15k. A aba não possui campo para registrar downgrades, e o AP_AR confirma que a NF de Out/25 (R$18k) está vencida há 72 dias porque "o downgrade não foi refletido na NF" — ou seja, cobraram R$18k quando o cliente já havia renegociado para R$15k.

**Mu Analytics: NF emitida mas nunca enviada.** R$19k de Out/25 com 67 dias de atraso simplesmente porque a NF não foi enviada ao cliente. Falha operacional básica.

---

## 4. DRE_Gerencial

### Problemas quantitativos

**Receita Bruta massivamente subestimada.** A DRE registra R$1.29M de receita no Q4, mas o extrato bancário mostra R$9.8M em recebimentos de clientes e o ERP registra R$11.5M em receitas. O CEO construiu a DRE usando apenas o MRR (~R$440k/mês), ignorando consultoria, setup e outras linhas de receita. A receita real é ~8x maior que a reportada. Consequentemente, todos os indicadores derivados (lucro bruto, EBITDA, margens, resultado líquido) estão errados.

**Alíquota de impostos fixa em 7.5% nos 3 meses.** Impostos sobre receita = exatamente 7.5% da receita bruta em todos os meses. Isso sugere que o CEO aplicou uma alíquota flat manual, não que conferiu os impostos reais devidos. A empresa está no Simples Nacional (DAS), onde a alíquota efetiva varia conforme a faixa de receita acumulada nos últimos 12 meses.

**Nenhuma validação cruzável entre DRE e ERP/Extrato.** As 63 fórmulas da DRE são todas internas à aba (somas de linha, cálculos de margem, totais trimestrais). Não há uma única referência cross-sheet. Todos os inputs são manuais, o que significa que a DRE é um modelo paralelo desconectado dos dados transacionais.

### Problemas qualitativos

**DRE "não auditada" montada manualmente pelo CEO.** O próprio título diz "não auditada" e o business case confirma que o CEO monta a DRE com "dados parciais". Isso é consistente com todos os problemas encontrados — não é um relatório derivado dos dados, é uma estimativa.

**Custos de infraestrutura Cloud crescem 22% no trimestre sem explicação.** AWS+GCP sobe de R$78k em Out para R$95k em Dez (+22%). A aba AP_AR confirma: "Aumento de 17% vs Out" na fatura AWS de Dez. Mas a DRE não registra nenhuma nota ou análise sobre a tendência — num cenário de EBITDA negativo, esse tipo de aumento deveria gerar alerta.

**EBITDA negativo, mas possivelmente fictício.** Out: -R$21k, Nov: -R$23k, Dez: -R$102k. A margem EBITDA vai de -5.2% para -27%. Porém, se a receita real é ~8x maior que a reportada, o EBITDA real pode ser radicalmente diferente (inclusive positivo). A DRE na forma atual não tem utilidade analítica — é preciso reconstruí-la com a receita completa para entender a real situação operacional.

---

## 5. AP_AR

### Problemas quantitativos

**R$97.5k em contas a receber vencidas (5 de 7 itens AR).** Aging:

- Delta Cloud: 72 dias, R$18k (NF errada por falta de ajuste de downgrade)
- Mu Analytics: 67 dias, R$19k (NF nunca enviada)
- Beta SaaS: 46 dias, R$38k (cliente churnou)
- Gamma Digital: 31 dias, R$15k (cobrança de setup nunca enviada)
- Pi Logistics: 26 dias, R$7.5k (risco alto de churn)

Cada item tem uma causa raiz diferente, mas todas apontam para a mesma coisa: inexistência de processo de cobrança.

**R$66k em contas a pagar vencidas com riscos jurídicos.** Três itens:

- Silva & Advogados: R$36k, 11 dias — cobrança contestada por divergência com contrato
- Freelancer B: R$12k, 31 dias — NF não emitida pelo fornecedor
- Freelancer A: R$18k, 16 dias — sem contrato formal

### Problemas qualitativos

**Freelancers operando sem contrato formal.** Freelancer A tem R$18k a pagar sem contrato. Isso expõe a empresa a risco trabalhista (vínculo empregatício) e tributário (sem NF, sem dedutibilidade).

**Honorários advocatícios contestados.** R$36k cobrados por Silva & Advogados com "valor diverge do contrato". Se o escritório de advocacia da empresa tem faturas contestadas, isso sugere falta de controle sobre os próprios contratos.

**13o salário pago com atraso.** A 2a parcela do 13o (R$97.5k) está marcada como "Paga com 2 dias de atraso". Pagamento de 13o fora do prazo gera multa e pode desencadear ações trabalhistas.

**AP_AR não está conectado a nenhuma outra aba.** A posição de contas é um snapshot isolado. Não alimenta a DRE (provisão para devedores duvidosos inexistente), não alimenta o fluxo de caixa (projeções de recebimento não consideram o aging), e não é reconciliável automaticamente com ERP ou extrato.

---

## 6. Caixa_Runway

### Problemas quantitativos

**Entradas e saídas completamente descoladas do extrato bancário.** Validação por script Python:

| Mês | Caixa Entradas | Extrato Entradas | Diferença |
|---|---|---|---|
| Out/25 | R$420k | R$2,847k | -R$2,427k |
| Nov/25 | R$398k | R$5,275k | -R$4,877k |
| Dez/25 | R$365k | R$5,409k | -R$5,044k |

| Mês | Caixa Saídas | Extrato Saídas | Diferença |
|---|---|---|---|
| Out/25 | -R$395k | -R$2,013k | +R$1,618k |
| Nov/25 | -R$385k | -R$2,058k | +R$1,673k |
| Dez/25 | -R$430k | -R$1,808k | +R$1,378k |

As diferenças são de ordem de grandeza. O extrato mostra movimentação 5–10x maior que o fluxo de caixa. A causa raiz é a mesma da DRE: o CEO usou apenas o MRR como proxy de entradas, ignorando consultoria, setup e outras receitas. O Caixa_Runway é uma projeção baseada na DRE incompleta, não uma conciliação do caixa real.

**Saldo inicial diverge do extrato em R$350k.** Caixa_Runway começa Out/25 com R$3,200,000. O extrato implica saldo de R$2,850,000. Diferença de R$350k não explicada — pode ser saldo de aplicações/investimentos não refletido no extrato, mas sem documentação é impossível confirmar.

**Runway de 45 meses é enganoso.** O cálculo de runway só produz resultado em Dez/25 (único mês com burn positivo = R$70.2k), resultando em ~45 meses. Mas esse cálculo usa o saldo final de Dez (R$3.16M) dividido pelo burn de um único mês — não é uma média significativa. A coluna "Média Mensal" calcula runway de 275 meses (23 anos), que é um número absurdo produzido por dividir R$3.16M pelo burn médio de R$11.5k.

### Problemas qualitativos

**Fluxo de caixa é projeção, não conciliação.** As entradas e saídas foram claramente inputs manuais baseados em estimativas (possivelmente derivados da DRE), não em extrato bancário real. Isso invalida o fluxo de caixa como ferramenta de gestão.

**Não separa caixa operacional de não-operacional corretamente.** As transferências internas entre contas (R$1.77M no trimestre) provavelmente estão inflando as movimentações no extrato, mas como o Caixa_Runway nem usa o extrato, isso nem é o problema principal — o problema é que nada bate com nada.

---

## Resumo Executivo dos Problemas

### Por severidade

**Crítico (invalida o relatório):**

1. DRE registra apenas MRR (~R$1.3M no Q4), ignorando ~R$9.8M em receitas reais — P&L inteiro está errado
2. Caixa_Runway descolado do extrato por ordens de grandeza — fluxo de caixa fictício
3. 19% do ERP com categorias absurdas — análise de despesa por rubrica é inviável
4. 21 NFs duplicadas no ERP, 19 delas entre clientes diferentes — integridade dos dados comprometida

**Grave (risco operacional/financeiro):**

5. R$4.6M em despesas sem documento no ERP — risco fiscal e de auditoria
6. R$97.5k em AR vencido sem ação de cobrança
7. 10 de 15 clientes com NFs pendentes ou em atraso
8. Freelancers sem contrato formal — risco trabalhista
9. 13o salário pago com atraso — risco trabalhista
10. 52% dos lançamentos do ERP não conciliados

**Estrutural (ausência de processo):**

11. Nenhuma fórmula cross-sheet — abas são ilhas isoladas
12. Extrato consolida 3 bancos em saldo único — reconciliação por banco impossível
13. DRE montada manualmente pelo CEO com dados parciais
14. Competência diverge do lançamento em 43% do ERP (lag médio 38 dias)
15. Campo "Tipo" do extrato não é confiável (tarifas bancárias que são folha de pagamento)
