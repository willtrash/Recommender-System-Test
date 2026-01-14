# Recommender System Test — Análise de Teste A/B de Sistema de Recomendação

## Objetivo
Avaliar se a introdução de um sistema de recomendação melhorado (grupo B) aumentou as taxas de conversão em até 14 dias após o cadastro — para visualizações de produto, adições ao carrinho e compras — em comparação ao modelo atual (grupo A). O critério de sucesso definido foi um aumento mínimo de **10%** em cada etapa do funil, com significância estatística avaliada por testes de proporção (z-test).

---

## Resumo técnico do teste A/B
- **Nome:** recommender_system_test  
- **Período de campanha:** 07-12-2020 a 01-01-2021 (parada de captação de novos usuários: 21-12-2020)  
- **Público:** 15% de novos usuários da região UE  
- **Tamanho esperado:** 6000 participantes  
- **Métrica principal:** conversão nas etapas `product_page`, `product_cart` e `purchase` dentro de 14 dias do cadastro  
- **Hipótese:** grupo B apresenta ≥10% de aumento nas taxas de conversão em cada etapa comparado ao grupo A  
- **Teste estatístico:** z-test para diferença entre proporções

---

## Metodologia
1. **Carregamento e inspeção inicial dos dados**  
   - Identificação de schemas/colunas relevantes (user_id, cohort/group, event_type, event_time, signup_date, region etc.).
2. **Limpeza e pré-processamento**  
   - Conversão de tipos (datas → `datetime`), normalização de nomes de eventos, filtragem para novos usuários na janela do experimento e para a região UE.
   - Tratamento de duplicatas e registro de valores ausentes (quantificação e justificativa do tratamento adotado).
3. **Construção da janela de observação**  
   - Para cada usuário, considerar apenas eventos ocorridos até 14 dias após a data de cadastro.
4. **Análise exploratória (EDA)**  
   - Conversão por etapa do funil por grupo (A vs B).  
   - Distribuição de número de eventos por usuário e por dia.  
   - Verificação da presença e balanceamento de usuários de ambas as amostras ao longo do período.  
   - Identificação de particularidades (picos sazonais, dias com dados faltantes, problemas de instrumentação).
5. **Avaliação estatística (A/B test)**  
   - Cálculo de proporções (p_A, p_B) e diferença absoluta/relativa.  
   - Teste z para diferença de proporções com nível de significância definido (ex.: α = 0.05).  
   - Correção/avaliação adicional se necessário (ex.: teste de homogeneidade, análises por subgrupos).
6. **Conclusões e recomendações**  
   - Interpretação dos resultados (significância estatística vs. tamanho do efeito) e recomendações de negócio (deploy gradual, testes complementares, monitoramento).

> Todas as etapas, queries e código estão detalhados no Jupyter Notebook incluído no repositório.

---

## Checklist de EDA (usado no notebook)
- [ ] Converter colunas de data para `datetime`  
- [ ] Filtrar usuários da região UE e na janela do experimento  
- [ ] Remover/justificar duplicatas e eventos inválidos  
- [ ] Confirmar que ambos os grupos (A e B) possuem usuários suficientes e presentes na janela de observação  
- [ ] Verificar distribuição de eventos por dia (detectar drift ou perda de instrumentação)  
- [ ] Calcular taxas de conversão para cada etapa do funil por grupo e por subperíodos

---

## Exemplo de como executar o z-test (trecho de código)
> Observação: o notebook contém o código completo. Aqui um snippet ilustrativo (Python — `statsmodels`).

```py
# Supondo que você já agregou os dados e tem:
# successes_A = número de usuários em A que converteram em X
# nobs_A = número total de usuários em A
# successes_B, nobs_B = equivalente para B

from statsmodels.stats.proportion import proportions_ztest

counts = [successes_B, successes_A]   # ordem: grupo B, grupo A
nobs = [nobs_B, nobs_A]
stat, pvalue = proportions_ztest(counts, nobs, alternative='larger')  # 'larger' se testamos B > A

print(f"z = {stat:.3f}, p-value = {pvalue:.4f}")
