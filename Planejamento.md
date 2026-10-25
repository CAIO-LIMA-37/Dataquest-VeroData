## **Estrutura do Projeto**

**Notebook 1: Importação, Limpeza e Pré-processamento**

* Filtrar a base de dados para isolar países individuais de agregados regionais (utilizando a coluna de código ISO3, se aplicável aos seus dados).
* Aplicar filtros de qualidade técnica recomendados pela literatura, como excluir nações com populações muito pequenas (por exemplo, abaixo de 1 milhão de habitantes) para evitar que essas curvas de mortalidade exijam ajustes de peso e distorçam a análise.


* Selecionar os indicadores cruciais de mortalidade e expectativa de vida para diferentes faixas etárias.

**Notebook 2: Análise Exploratória de Dados (EDA) e Dinâmicas de Idade**

* Investigar a relação histórica entre a expectativa de vida e a igualdade na duração da vida, variáveis que costumam apresentar uma forte correlação positiva.


* Analisar a distribuição da mortalidade explorando o conceito de "idade limite" (threshold age): reduções nas taxas de mortalidade abaixo dessa idade aumentam a expectativa e a igualdade de vida, enquanto reduções acima dessa idade diminuem a igualdade na duração da vida.


* Identificar anomalias e divergências recentes, investigando períodos ou países onde a expectativa de vida e a igualdade se moveram em direções opostas devido a melhorias focadas em idades mais avançadas.



**Notebook 3: Modelagem e Clusterização em Múltiplos Estágios**

* Aplicar a estrutura de clusterização em múltiplos estágios para capturar simultaneamente as mudanças internas de um país ao longo do tempo e as diferenças entre os países.


* **Estágio 1:** Agrupar observações anuais dos países para identificar um conjunto pequeno de "estados de mortalidade" que caracterizam o perfil do país em qualquer ponto no tempo.


* **Estágio 2:** Agrupar os países em clusters definitivos com base em suas sequências temporais de transição entre esses estados de mortalidade.


* Avaliar se os modelos capturam divisões geográficas conhecidas, como a persistente divisão Leste-Oeste na Europa ou padrões distintos na América Latina ligados à mortalidade de jovens adultos.



**Notebook 4: Síntese de Insights para o DataQuest**

* Consolidar os resultados visuais (mapas e gráficos temporais).
* Traduzir a modelagem matemática em uma narrativa direta sobre políticas de saúde, transições demográficas e projeções futuras para apresentar no campeonato.