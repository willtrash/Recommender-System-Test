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
  Ambos os grupos estão presentes ao longo da janela de observação. Foi detectada sobreposição de usuários entre A e B (441 usuários) e estes foram removidos para garantir independência entre os grupos.

- **O efeito observado atinge o aumento mínimo esperado (≥ 10%) em alguma etapa do funil?**  
  Não — nenhuma das etapas apresentou aumento de ≥ 10% em B comparado a A.
  
- **Há evidência estatística de diferença entre os grupos (teste A/B)?**  
  Sim — o z-test de comparação de proporções aplicado às conversões de `purchase` retornou **p-value ≈ 0,03596**. Com α = 0.05 a diferença nas taxas de compra entre A e B é **estatisticamente significativa**, onde grupo A teve uma taxa de conversão geral ligeiramente superior à do grupo B.

---

## Conclusão

- Embora exista diferença estatisticamente significativa entre os grupos na métrica de **purchase**, o efeito observado **não aponta para o benefício esperado do novo sistema de recomendação** (meta definida: ≥ +10% em cada etapa do funil). Na verdade, a taxa de compra do grupo B ficou **menor** (~−5% relativo a A). 

---

## Tecnologias e bibliotecas

- Linguagem: **Python**
- Bibliotecas: **pandas, matplotlib, pyplot, seaborn, proportions_ztest**

---

## Contato

Willian De Souza Pereira — ws13292@gmail.com

GitHub: https://github.com/willtrash

---

## Licença

Este repositório está disponível para estudo e demonstração. Sinta-se à vontade para clonar, adaptar e abrir *issues* com dúvidas ou sugestões.
