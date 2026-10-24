# Amazon Insights · Product Performance & Customer Insights

Tech Challenge Fase 04 · POSTECH Data Analytics

Dashboard analítico em Power BI que investiga a relação **Preço → Desconto → Engajamento → Satisfação do Cliente** no portfólio de produtos da Amazon e transforma os achados em recomendações para a liderança.

**Pergunta de negócio:** quais produtos, categorias e estratégias comerciais merecem maior atenção e quais oportunidades podem ser identificadas a partir dos dados disponíveis?

---

## Acesso rápido

| Entregável | Link |
|---|---|
| Vídeo executivo (até 7 min) | [youtu.be/sOfy78gpGvw](https://youtu.be/sOfy78gpGvw) |
| Dashboard Power BI (.pbix) | [AmazonInsights.pbix](https://github.com/gioandradebonucci/Amazon-Product-Performance-Customer-Insights-/blob/main/AmazonInsights.pbix) · [baixar](https://github.com/gioandradebonucci/Amazon-Product-Performance-Customer-Insights-/raw/main/AmazonInsights.pbix) |
| Base de dados (CSV) | [amazon.csv](https://github.com/gioandradebonucci/Amazon-Product-Performance-Customer-Insights-/blob/main/amazon.csv) · [link raw (fonte do Power BI)](https://raw.githubusercontent.com/gioandradebonucci/Amazon-Product-Performance-Customer-Insights-/refs/heads/main/amazon.csv) |
| Base original (Kaggle) | [Amazon Sales Dataset](https://www.kaggle.com/datasets/karkavelrajaj/amazon-sales-dataset) |
| Repositório | [github.com/gioandradebonucci/Amazon-Product-Performance-Customer-Insights-](https://github.com/gioandradebonucci/Amazon-Product-Performance-Customer-Insights-) |

---

## Como abrir o dashboard

1. Baixe o arquivo [`AmazonInsights.pbix`](https://github.com/gioandradebonucci/Amazon-Product-Performance-Customer-Insights-/raw/main/AmazonInsights.pbix) e abra no **Power BI Desktop**.
2. Os dados são lidos direto deste repositório (parâmetro `pCaminho` aponta para o link raw do `amazon.csv`), então **não é preciso ter o CSV na máquina**. É necessário acesso à internet para atualizar.
3. Se pedir credenciais ao atualizar, escolha **Anônimo** (repositório público) e nível de privacidade **Público**.
4. Para usar uma cópia local do CSV: Página Inicial > Transformar dados > **Editar parâmetros** > `pCaminho`. Nesse caso, na consulta `_Origem` troque `Web.Contents(pCaminho)` por `File.Contents(pCaminho)`.

### Conferência após atualizar (Visão Executiva, sem filtros)

| Cartão | Valor esperado |
|---|---|
| Produtos no portfólio | 1.351 |
| Avaliações | 23,8 Mi |
| Nota média ponderada | 4,16 |
| Desconto médio | 46,7% |
| Avaliações nos top 5% de produtos | 47,0% |
| Portfólio em risco | 14,6% |
| Produtos em alerta (página Preços) | 44 |

---

## Estrutura do dashboard

1. **Visão Executiva** — KPIs, treemap de engajamento por subcategoria, saúde do portfólio por quadrante e principais achados.
2. **Produtos e Categorias** — dispersão engajamento × nota, matriz com drill-down Categoria > Subcategoria > Item, tabela de produtos e "joias escondidas".
3. **Preços e Estratégia** — nota por faixa de desconto, correlação desconto × nota por subcategoria, mapa de calor desconto × preço e tabela de alertas.
4. **Customer Insights** — temas citados nas reviews, delta de nota por tema, termos típicos de produtos mal e bem avaliados (lift) e leitura das avaliações.
5. **Recomendações** — quatro ações com botão para a página que as comprova e mensagens para a liderança.
6. **Detalhe do Produto** (página oculta) — botão direito em qualquer produto > **Detalhar**.

Navegação por botões no topo de todas as páginas, com segmentadores de categoria, faixa de preço, quadrante e subcategoria.

---

## Preparação dos dados (Power Query)

- **Leitura e limpeza:** todas as colunas lidas como texto e convertidas com cultura fixa `en-US` (evita nota 42 em vez de 4,2); remoção de linhas vazias e de URLs dentro do texto das reviews.
- **Deduplicação** por `product_id`, mantendo o registro com mais avaliações; sinalização de variantes (mesmo nome, IDs diferentes).
- **Conversão de moeda:** preços de INR para USD pelo parâmetro `pTaxaCambio` (0,0105). Todas as colunas e medidas monetárias herdam a conversão.
- **Hierarquia de categoria** em 3 níveis (N1 > N2 > Folha), separada em dimensão própria (`dCategoria`) para drill-down nativo.
- **Faixas** de preço, desconto e nota, cada uma com coluna de ordenação.
- **Benchmarks por subcategoria:** mediana de nota e desconto, satisfação relativa, agressividade do desconto e share de engajamento.
- **Quadrantes** por mediana global de nota e de avaliações: Estrela, Joia Escondida, Risco de Reputação e Cauda Longa.
- **Alerta de desconto:** desconto ≥ 60%, avaliações acima da mediana e nota abaixo de 4,0.
- **Curvas ABC** por engajamento e por exposição comercial, e índice de relevância (0 a 100).
- **Customer Insights:** 10 temas detectados por palavras-chave nas reviews, sentimento léxico com tratamento de negação e flag de menção negativa.

## Modelo de dados

| Tabela | Papel |
|---|---|
| `fProdutos` | 1 linha por produto, com todo o tratamento acima |
| `dCategoria` | Dimensão de categoria (hierarquia) |
| `fTemas` | Ponte produto × tema mencionado |
| `dTema` | Dimensão de tema, agrupada em famílias (Produto, Experiência, Comercial, Logística, Pós-venda) |
| `_Medidas` | Medidas DAX organizadas em pastas: Base, Relativas, Portfólio e Risco, Preço e Desconto, Customer Insights, Detalhe |

---

## Principais achados

- **Engajamento concentrado:** 68 produtos (5% do catálogo) somam cerca de 47% das 23,8 milhões de avaliações.
- **Desconto alto não compra satisfação:** a nota média cai de 4,17 (até 20% de desconto) para 3,97 (acima de 80%). Em Fones, Cozinha e TV a correlação desconto × nota fica entre -0,23 e -0,33.
- **Risco concentrado:** 197 produtos (14,6%) têm alto alcance e nota abaixo da mediana, e respondem por 20% das avaliações. Dos 44 em alerta de desconto, 17 são fones de ouvido.
- **Oportunidade:** 188 produtos superam a nota dos pares com 100+ avaliações, mas têm menos de 1% do engajamento da subcategoria.

## Recomendações

1. **Rever a política de desconto em áudio e vestíveis.** Limitar descontos acima de 60% a produtos com nota 4,0+ e revisar os 44 itens em alerta.
2. **Dar visibilidade às joias escondidas.** Vitrine, busca patrocinada e recomendação, medindo a migração para o quadrante Estrela.
3. **Atacar a causa da insatisfação em áudio e chamadas.** Exigir padrão mínimo de microfone e áudio dos fornecedores e acelerar o fluxo de troca (pós-venda).
4. **Proteger onde o volume está.** Monitorar mensalmente o grupo de alto alcance e nota abaixo da mediana, com meta de levar itens ao quadrante Estrela ou descontinuar.

**Mensagens para a liderança**
1. Poucos produtos formam a imagem da loja: cuidar deles vale mais que ampliar o catálogo.
2. Desconto alto é sinal de alerta, não de sucesso.
3. Há demanda a destravar em produtos bem avaliados e pouco vistos.
4. O cliente reclama de áudio e pós-venda: resolver isso protege a nota onde o volume está.

---

## Limites conhecidos

- A base não tem vendas: **engajamento = volume de avaliações** e a Exposição Comercial é um proxy (preço × avaliações), não faturamento.
- Correlação não é causa: descontos altos podem refletir produtos de entrada, e não serem a causa da nota menor.
- Os textos das páginas Visão Executiva e Recomendações são estáticos e refletem a base completa (não mudam com filtros).
- Temas e sentimento vêm de léxico de palavras-chave, uma aproximação e não um modelo de linguagem.

---

## Tecnologias

Power BI Desktop · Power Query (M) · DAX · Modelo semântico em estrela

## Autoria

Giovanna de Andrade Bonucci · [@gioandradebonucci](https://github.com/gioandradebonucci)

Projeto desenvolvido para o Tech Challenge da pós-graduação POSTECH em Data Analytics.
