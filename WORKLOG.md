# Worklog — Diagnóstico Financeiro Nuvio Tech

**Case:** Barte FDE — Head of Finance
**Início:** 2026-04-21
**Ferramentas:** Claude (Cowork), Python, Excel

---

## Etapas concluídas

### 1. Extração do business case
- **Input:** `Business Case - Barte FDE.pdf`
- **Output:** `Business Case - Barte FDE.md`
- Conversão do enunciado para markdown para referência rápida durante a análise.

### 2. Mapeamento da base financeira
- **Input:** `Nuvio_Tech_Base_Financeira.xlsx` (6 abas, ~640 registros)
- **Output:** `Nuvio_Tech_Dicionario_Tabelas.md`
- Descrição de cada aba: colunas, natureza dos dados, volume de registros.
- Classificação raw vs. construída para cada tabela.
- Identificação de dependências entre abas (e ausência de vínculos cross-sheet).
- Receita_Clientes identificada como visão parcial (apenas MRR).

### 3. Diagnóstico de problemas
- **Input:** Todas as abas + business case
- **Output:** `PROBLEMAS.md`
- Validação cruzada via scripts Python: saldos, categorizações, NFs duplicadas, divergências entre abas.
- 15 problemas catalogados em 3 camadas de severidade (crítico, grave, estrutural).
- Achados principais: DRE com receita errada, Caixa_Runway descolado do extrato por 5–10x, 19% do ERP mal-categorizado, 21 NFs duplicadas entre clientes diferentes.

### 4. Correção de premissa — receita real vs. MRR
- **Descoberta:** Comparação Extrato (R$9.8M recebidos de clientes no Q4) vs. ERP (R$11.5M receitas, ratio 0.9x) vs. Receita_Clientes (R$769k MRR faturado, ratio 0.08x).
- **Conclusão:** A empresa tem receitas de consultoria, setup e outras ~12.7x maiores que o MRR. A Receita_Clientes monitora só MRR. A DRE subestima a receita por ~8x (não superestima como concluído inicialmente).
- **Impacto:** Dicionário e PROBLEMAS.md atualizados. EBITDA negativo da DRE possivelmente fictício — real pode ser positivo.
- 6 dos 15 clientes (R$494k MRR) não aparecem no extrato — possível inadimplência não rastreada.

---

### 5. Plano de construção da planilha corrigida
- **Output:** `PLANO_PLANILHA.md`
- 7 abas planejadas em ordem de dependência: Extrato_Limpo → ERP_Limpo → Receita_Completa → DRE_Corrigida → Caixa_Corrigido → AP_AR_Anotado → Dashboard
- Cada aba com critérios de verificação contra PROBLEMAS.md
- Separação clara entre problemas resolvíveis na planilha e problemas operacionais (para o PPT)

---

## Próximas etapas

- [ ] Implementar `Nuvio_Tech_Corrigida.xlsx` (7 abas, conforme PLANO_PLANILHA.md)
- [ ] Montar material de apresentação (diagnóstico + recomendações para o board)
