# Recommender System Test — Análise de Teste A/B de Sistema de Recomendação

## Objetivo
Avaliar se a introdução de um sistema de recomendação melhorado (grupo B) aumentou as taxas de conversão em até 14 dias após o cadastro — para visualizações de produto, adições ao carrinho e compras — em comparação ao modelo atual (grupo A). O critério de sucesso definido foi um aumento mínimo de **10%** em cada etapa do funil, com significância estatística avaliada por testes de proporção (z-test).

---

## O teste A/B
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

## O que você encontra no repositório

- recommender_system_test.ipynb — Notebook com todo o pipeline: carregamento, EDA, pré-processamento, agregações, testes estatísticos, visualizações e conclusões.

- README.md — este arquivo.

---

## Principais perguntas respondidas

- **O experimento foi corretamente instrumentado e os dados parecem confiáveis?**  
  Sim — foram verificadas consistência de tipos, presença de valores ausentes/`details` parcialmente preenchida (mas sem impacto na análise) e checagem temporal dos eventos. Não foram identificados vieses temporais relevantes entre os grupos (período de Natal foi observado como causa de variação natural).

- **Os usuários do grupo A e do grupo B estão presentes e balanceados na janela de observação (14 dias)?**  
  Ambos os grupos estão presentes ao longo da janela de observação. Foi detectada sobreposição de usuários entre A e B (441 usuários) e estes foram removidos para garantir independência entre os grupos. Após limpeza (`df_clean`), o tamanho das amostras ficou: **A = 7.432 usuários**, **B = 5.763 usuários** (total ≈ 13.195).

- **Como ficaram as taxas de conversão em cada etapa do funil (`product_page` → `product_cart` → `purchase`) dentro de 14 dias?**  
  (valores calculados no notebook a partir dos eventos dentro da janela de 14 dias)
  - `product_page`: A = **66,15%**, B = **64,24%** → **B ≈ −2,9%** (relativo a A).  
  - `product_cart`: A = **31,54%**, B = **32,84%** → **B ≈ +4,1%**.  
  - `purchase`: A = **34,07%**, B = **32,37%** → **B ≈ −4,99%**.

- **O efeito observado atinge o aumento mínimo esperado (≥ 10%) em alguma etapa do funil?**  
  Não — nenhuma das etapas apresentou aumento de ≥ 10% em B comparado a A. O maior ganho observado (relativo) foi na etapa `product_cart` (~+4,1%), abaixo do threshold de 10%.

- **Há evidência estatística de diferença entre os grupos (teste A/B)?**  
  Sim — o z-test de comparação de proporções aplicado às conversões de `purchase` (com usuários duplicados removidos) retornou **p-value ≈ 0,03596**. Com α = 0.05, rejeita-se a hipótese nula de igualdade das proporções, ou seja, a diferença nas taxas de compra entre A e B é **estatisticamente significativa**.

---

## Conclusões

- **Resumo numérico (purchase, janela 14 dias, após remoção de sobreposição):**
  - Usuários que compraram: **A = 2.555 / 7.432 → 34,38%**, **B = 1.881 / 5.763 → 32,64%**.  
  - Diferença absoluta (B − A): **−0,01739** (≈ −1,74 pontos percentuais).  
  - Diferença relativa: **B ≈ −5,06%** em relação a A.  
  - Teste estatístico: **z-test p ≈ 0,03596** → diferença estatisticamente significativa (α = 0,05).

- **Interpretação prática:**  
  Embora exista diferença estatisticamente significativa entre os grupos na métrica de **purchase**, o efeito observado **não aponta para o benefício esperado do novo sistema de recomendação** (meta definida: ≥ +10% em cada etapa do funil). Na verdade, a taxa de compra do grupo B ficou **menor** (~−5% relativo a A). Houve um pequeno aumento em `product_cart` (~+4%), mas novamente abaixo do critério de sucesso e sem evidência de efeito escalável por si só.

- **Recomendações operacionais e próximas etapas:**
  1. **Não fazer rollout amplo do novo sistema** com base nesses resultados — o objetivo mínimo de +10% não foi atingido e a métrica de compra piorou.  
  2. **Investigar heterogeneidade do efeito:** analisar por coortes (data de signup), canais de aquisição, dispositivo e país dentro da UE para identificar subgrupos onde o sistema pode performar melhor.  
  3. **Aprofundar diagnósticos de funnel:** entender por que `product_cart` cresceu enquanto `purchase` caiu (ex.: problemas no checkout, mudanças no preço/UX, intenção vs. conversão final).  
  4. **Revisar experimento e amostragem:** o dataset incluiu sobreposição de usuários (441) — a remoção foi correta; considere A/A tests ou pré-registro de métricas para futuros testes.  
  5. **Testes adicionais:** iterar no algoritmo de recomendação e executar novas variantes (testes A/B com receitas, ranking alternativo, ou personalização por segmento) com monitoramento de métricas de retenção e ARPU além da conversão em 14 dias.  
  6. **Avaliar custo/benefício:** se pequenas melhorias no funil podem justificar investimento, considere deploy controlado (canary) com monitoramento contínuo antes de escalar.

- **Observação final:** o notebook contém todas as análises detalhadas (EDA, verificações de qualidade, agregações por usuário, gráficos temporais e o cálculo do z-test). Use as figuras e tabelas do notebook e da apresentação PDF para comunicar esses resultados a stakeholders não técnicos — a mensagem principal deve destacar que **o experimento não alcançou a meta operacional de +10% e requer iteração**, não um rollout imediato.
