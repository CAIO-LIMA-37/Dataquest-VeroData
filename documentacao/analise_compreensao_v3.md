# 📘 Documento de Compreensão e Arquitetura: Pipeline Demográfico `notebooks_v3`
**Projeto:** CIRS DataQuest 2026 (American Statistical Association - ASA)  
**Equipe:** VeroData  
**Base de Dados:** World Population Prospects 2024 (WPP2024) - Divisão de População da ONU  
**Data da Análise:** Outubro de 2026  
**Autor:** Antigravity (Pair Programming com Pedro Caio)

---

## 🧭 Sumário Executivo

Este documento consolida o entendimento aprofundado de todo o material construído na pasta `notebooks_v3`. 

Diferente das duas versões anteriores (`notebooks/` e `notebooks_v2/`), que continham análises exploratórias fragmentadas e gráficos genéricos sem um fio condutor teórico forte, a **versão 3 (`notebooks_v3`)** estabelece um **pipeline científico e metodológico de alto nível**, fundamentado diretamente na literatura demográfica contemporânea mais respeitada do mundo:
1. **O Framework de Clusterização Multi-Estágio de Araújo et al. (2026)** (*"Characterising mortality dynamics across countries and time using a multi-stage clustering approach"*).
2. **A Dinâmica da Idade Limite (*Threshold Age*) de Aburto et al. (PNAS, 2020)** (*"Dynamics of life expectancy and life span equality"*).

O projeto deixou de ser uma mera "visualização de dados da ONU" para se tornar uma **pesquisa computacional de modelagem longitudinal**, capaz de responder simultaneamente a duas perguntas cruciais:
- **Como caracterizar o estado epidemiológico de um país em qualquer ano histórico?** (Estágio 1 - Redução de dimensionalidade para um "alfabeto" de 4 Estados de Mortalidade).
- **Como os países navegaram por esses estados ao longo de 74 anos (1950–2023) e quais arquétipos geopolíticos de transição demográfica emergiram?** (Estágio 2 - Mineração e clusterização de séries temporais / sequências longitudinais).

---

## 🏛️ 1. O Diagnóstico Comparativo: Por que a Versão 3 Faz Sentido?

| Dimensão | Versões Anteriores (v1 e v2) | Versão Atual (`notebooks_v3`) |
| :--- | :--- | :--- |
| **Abordagem Analítica** | Gráficos exploratórios desconexos (pirâmides, fecundidade, projeções com variantes) sem tese central. | **Modelagem em Dois Estágios** com objetivo claro: descobrir trajetórias latentes de desenvolvimento em saúde. |
| **Fundamentação Teórica** | Empírica/ad-hoc, sem diálogo explícito com artigos acadêmicos da demografia matemática. | Ancorada em **Araújo (2026)** e **Aburto (2020)**, permitindo justificar cada decisão técnica na banca avaliadora da ASA. |
| **Tratamento de Dados** | Filtros frouxos que misturavam regiões mundiais, países insulares e territórios dependentes. | Filtro de soberania (`ISO3`) e corte populacional ($\ge 1\text{M}$ hab.), eliminando ruídos estocásticos de micro-populações. |
| **Integridade Temporal** | Erro fatal no pré-processamento que eliminava países por causa da inconsistência do ano 2101 da ONU. | **Descoberta e correção do horizonte temporal (2100)**, garantindo integridade de 100% da série histórica (161 países). |
| **Métodos de ML** | Sem clusterização formal ou com algoritmos aplicados sem critério comparativo de validação. | **Benchmarking formal**: K-Means vs. Hierárquico vs. GMM, avaliados por Silhouette Score e BIC, justificando demograficamente $K=4$. |
| **Narrativa para a ASA** | Apresentação de relatórios descritivos padrão. | **História completa de políticas públicas**, com mapas de tapete (*indexplots*), detecção de divergências regionais e cenários futuros. |

---

## 🔬 2. Pilares Teóricos da Modelagem

### 2.1 O Modelo Multi-Estágio de Araújo et al. (2026)
Ao analisar séries demográficas de múltiplos países ao longo de décadas, surgem dois desafios matemáticos concorrentes:
1. **Heterogeneidade Transversal:** Em um mesmo ano (ex: 1960), países ricos e países pobres vivem realidades epidemiológicas radicalmente distintas.
2. **Dependência Longitudinal:** O estado de um país no ano $t$ é fortemente condicionado ao ano $t-1$, mas a velocidade da transição varia drasticamente (saltos tecnológicos rápidos vs. estagnações prolongadas por guerras/epidemias).

A solução adotada em `notebooks_v3` divide o problema em dois níveis:
- **Estágio 1 (Discretização Estática):** Agrupa todas as observações país-ano ($N \times T = 161 \times 74 = 11.914$ pontos) num espaço multivariado de 6 dimensões de mortalidade, mapeando cada observação para um estado discreto $s \in \{0, 1, 2, 3\}$.
- **Estágio 2 (Trajetória Longitudinal):** Transforma a história de cada país em uma cadeia temporal finita $S_i = (s_{i, 1950}, s_{i, 1951}, \dots, s_{i, 2023})$ e agrupa os países de acordo com a semelhança de suas jornadas temporais completas.

### 2.2 A Dinâmica da Idade Limite (*Threshold Age*) de Aburto et al. (PNAS, 2020)
Historicamente, acreditava-se que o aumento da expectativa de vida ao nascer ($e_0$) sempre caminhava de mãos dadas com o aumento da igualdade na duração da vida (redução da dispersão das idades ao morrer). Aburto et al. provaram matematicamente a existência de uma **idade limite** ($a^* \approx 65\text{ a }70$ anos):
- **Ganhos em idades $< a^*$ (Infância e Juventude):** Evitar mortes precoces empurra os indivíduos para a velhice, **aumentando a expectativa de vida e a igualdade** (o desvio-padrão das mortes cai).
- **Ganhos em idades $> a^*$ (Idosos e Octogenários):** Evitar mortes na velhice aumenta a expectativa de vida, mas **aumenta a desigualdade** (alguns vivem até os 80, outros até os 105, esticando a cauda da distribuição).
- Esse conceito é o motor que valida a escolha das 6 features do Estágio 1.

---

## 🗺️ 3. Arquitetura do Pipeline de Dados (`notebooks_v3`)

```mermaid
graph TD
    A["Raw Data (ONU WPP 2024)<br/>WPP2024_Demographic_Indicators_Medium.csv<br/>40.8 MB"] --> B["Notebook 1: Limpeza & Filtros"]
    B -->|Filtro ISO3 + Pop >= 1M + Corte 2100| C["new_data/wpp2024_clean.csv<br/>161 Países x 151 Anos (24.311 linhas)"]
    
    C --> D["Notebook 2: EDA & Dinâmicas de Idade"]
    D -->|Validação de Hipóteses Demográficas| D1["5 Países Sentinela: SWE, JPN, RUS, BRA, USA"]
    D -->|Testes de Aburto (2020)| D2["Transição Q5 vs Q1560 vs LEx"]
    D -->|Projeções até 2100| D3["Convergência vs Disparidade Persistente"]
    
    C --> E["Notebook 3: Modelagem Estágio 1 (Estados de Mortalidade)"]
    E -->|Normalização de 6 Features| E1["X_scaled: 11.914 observações x 6 features"]
    E1 -->|Benchmarking K=2 a 10| E2["K-Means vs GMM (BIC) vs Hierárquico"]
    E2 -->|Grade 3x3 Silhueta| E3["Seleção Ótima: K=4 (Score: 0.43)"]
    E3 -->|Ordenação Ordinal 0 a 3| F["new_data/wpp2024_estados_mortalidade.csv"]
    
    F --> G["Notebook 4: Modelagem Estágio 2 (Trajetórias Temporais)"]
    G -->|Pivoteamento| G1["Matriz de Sequências: 161 Países x 74 Anos (1950-2023)"]
    G1 -->|Visualização de Transição| G2["Indexplot Global (Tapete Ordenado)"]
    G2 -->|Próximos Passos| G3["Clusterização DTW/Hamming + Geopandas + Síntese ASA"]
```

---

## 🔍 4. Detalhamento Passo a Passo dos Notebooks

### 4.1 Notebook 1: Importação, Filtros e Pré-processamento
* **Arquivo:** `1_Importacao_Filtros_e_Pre-processamento.ipynb`
* **Entrada:** `original_data/WPP2024_Demographic_Indicators_Medium.csv`
* **Saída:** `new_data/wpp2024_clean.csv`

#### Decisões Críticas Implementadas:
1. **Leitura Blindada:** Para prevenir falhas de `ParserError` comuns em CSVs de grande porte da ONU com acentuação e separadores mistos, o notebook inspeciona a primeira linha em `utf-8-sig`, seleciona dinamicamente entre `;` e `,`, e força a conversão numérica de campos com strings nulas (`NaN`, `NULL`).
2. **Seleção de Variáveis Chave:**
   - Identificadores: `Location`, `ISO3_code`, `Time`, `TPopulation1July`.
   - Indicadores Demográficos:
     - `LEx` ($e_0$): Expectativa de vida ao nascer.
     - `Q5` ($_{5}q_0$): Probabilidade de morte antes dos 5 anos (por mil nascidos vivos).
     - `Q1560` ($_{45}q_{15}$): Probabilidade de morte adulta entre 15 e 60 anos (por mil indivíduos).
     - `LE15` ($e_{15}$): Expectativa de sobrevida aos 15 anos.
     - `LE65` ($e_{65}$): Expectativa de sobrevida aos 65 anos.
     - `LE80` ($e_{80}$): Expectativa de sobrevida aos 80 anos.
3. **Filtro de Soberania Nacional:** Exigência de `ISO3_code` preenchido. Isso elimina instantaneamente entidades regionais e blocos agregados (ex: "World", "Sub-Saharan Africa", "Latin America", "High-income countries"), reduzindo de centenas de linhas brutas para 237 entidades com código de país formal.
4. **Filtro de Porte Populacional ($\ge 1\text{ milhão}$ em 2023):** Conforme demonstrado por Araújo (2026), nações com populações minúsculas (ilhas do Pacífico, microestados europeus) sofrem de instabilidade amostral — uma única epidemia ou catástrofe natural causa picos espúrios na curva de mortalidade que exigiriam pesos complexos. O corte em $\ge 1.000$ milhares reduz a amostra para **161 países altamente confiáveis**, cobrindo $>98\%$ da população mundial.
5. **A Grande Descoberta do Bug Temporal da ONU:**
   - No WPP 2024, a coluna de projeção populacional total se estende até o ano de **2101**, mas as estimativas de taxas de mortalidade detalhadas encerram rigorosamente no ano de **2100**.
   - Consequência: Qualquer chamada ingênua a `.dropna()` eliminava **todos os 161 países** do dataset.
   - Solução adotada: Filtragem estrita `Time <= 2100`, seguida de limpeza de resíduos.
   - Resultado: Exatamente **24.311 linhas balanceadas** ($161 \times 151$ anos de 1950 a 2100).

---

### 4.2 Notebook 2: Análise Exploratória de Dados (EDA) e Dinâmicas de Idade
* **Arquivo:** `2_Analise_Exploratoria_de_Dados_(EDA)_e_Dinamicas_de_Idade.ipynb`
* **Entrada:** `new_data/wpp2024_clean.csv`

#### Estrutura Conceitual:
1. **Divisão Histórico vs. Projeção:**
   - Histórico (1950–2023): **11.914 registros** para calibração, treinamento e inferência empírica.
   - Projeções (2024–2100): **12.397 registros** para testes de convergência futura.
2. **Seleção de Países Sentinela da Literatura:**
   - 🇸🇪 **Suécia (`SWE`)** e 🇯🇵 **Japão (`JPN`):** Fronteira tecnológica e sanitária global; evolução sustentada da longevidade com baixa desigualdade.
   - 🇷🇺 **Rússia (`RUS`):** O caso paradigmático de que a longevidade não é estritamente monotônica; choque de mortalidade em homens adultos após o colapso da URSS (alta de `Q1560`).
   - 🇧🇷 **Brasil (`BRA`):** Transição latino-americana típica; queda expressiva da mortalidade infantil nos anos 70–90, mas com freio persistente causado por mortalidade violenta/externa em jovens adultos.
   - 🇺🇸 **Estados Unidos (`USA`):** Estagnação recente e até recuo na expectativa de vida nas últimas duas décadas ("mortes por desespero", opioides, obesidade e desigualdade no acesso à saúde).
3. **Validação Gráfica da Dinâmica de Idade Limite:**
   - O gráfico de dispersão demonstra claramente a não-linearidade:
     - Enquanto $LE_x < 70$, a correlação com $Q_5$ é quase perfeitamente linear e íngreme (cada vida de criança salva gera ganhos massivos em anos vividos).
     - Quando $LE_x > 70$, $Q_5$ estabiliza perto de zero, e a expectativa de vida passa a ser explicada quase inteiramente pelo recuo de $Q_{15\text{-}60}$ e avanço de $LE_{65}$ e $LE_{80}$.
4. **Exploração de Projeções (2024–2100):**
   - Os modelos da ONU projetam uma **convergência demográfica tardia**, mas com um hiato estrutural entre o bloco da Europa/Japão e a América Latina/Leste Europeu que só começa a diminuir após 2070.

---

### 4.3 Notebook 3: Modelagem e Clusterização em Múltiplos Estágios (Estágio 1)
* **Arquivo:** `03_modelagem_estados_mortalidade.ipynb`
* **Entrada:** `new_data/wpp2024_clean.csv` (filtrado para 1950–2023)
* **Saída:** `new_data/wpp2024_estados_mortalidade.csv`

#### Metodologia Matemática do Estágio 1:
1. **Matriz de Features Multivariadas ($11.914 \times 6$):**
   - Vetor de entrada: $\mathbf{x} = [LEx, Q5, Q1560, LE15, LE65, LE80]^T$.
   - Padronização: $z = \frac{x - \mu}{\sigma}$ via `StandardScaler`, garantindo que grandezas em escalas distintas (anos de vida vs. mortes por mil) tenham peso equi-ponderado no cálculo da distância euclidiana.
2. **Benchmarking de Algoritmos:**
   - **K-Means:** Centroides euclidianos, rápido e com limites nítidos de partição no espaço hiperdimensional.
   - **Clusterização Hierárquica Aglomerativa:** Testada com diferentes critérios de ligação (Ward, Complete, Average). Ward apresentou melhor Silhouette, mas com complexidade computacional desnecessária para $N=11.914$.
   - **Gaussian Mixture Models (GMM):** Avaliado pela métrica de informação Bayesiana (BIC). Apresentou queda assintótica sem cotovelo nítido, gerando distribuições probabilísticas elípticas difusas em vez de fronteiras operacionais categóricas.
3. **Diagnóstico dos Diagramas de Silhueta (Matriz 3x3 de $K=2$ a $10$):**
   - $K=2$: Apresenta o maior escore global ($\sim 0.58$), porém é **demograficamente inútil** (apenas rotula países em "ricos" e "pobres", destruindo as nuances da transição).
   - $K=3$: Escore razoável, mas ainda aglutina nações de transição intermediária com nações de colapso sanitário agudo.
   - **$K=4$ (Vencedor com Silhouette Score $\approx 0.43$):**
     - Todas as 4 lâminas de clusters ultrapassam a média da silhueta.
     - **Praticamente ausência total de valores negativos de silhueta**, comprovando que quase nenhuma observação país-ano ficou mal alocada.
     - Permite isolar um grupo de transição crítica (Estado 1), fundamental para capturar os períodos de epidemia de HIV na África e crises de transição econômica.
4. **Ordenação Ordinal e Reversão dos Centroides:**
   - Os clusters nativos do K-Means foram ordenados de forma crescente em relação a `Exp_Vida_Nascer` e remapeados para $0 \to 3$:

| Estado de Mortalidade | Nome Demográfico | $e_0$ (EV Nascer) | $Q_5$ (Mort. Infantil) | $Q_{15\text{-}60}$ (Mort. Adulta) | $e_{15}$ | $e_{65}$ | $e_{80}$ |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Estado 0** | **Crise Aguda / Pré-Transição** | **42.0 anos** | **261.5 ‰** | **438.1 ‰** | 44.3 | 10.9 | 4.8 |
| **Estado 1** | **Transição Inicial / Alta Mortalidade** | **57.1 anos** | **121.9 ‰** | **295.2 ‰** | 51.2 | 12.4 | 5.4 |
| **Estado 2** | **Transição Avançada / Intermediária** | **69.8 anos** | **35.0 ‰** | **169.8 ‰** | 57.6 | 14.4 | 6.2 |
| **Estado 3** | **Pós-Transição / Baixa Mortalidade** | **78.1 anos** | **9.1 ‰** | **95.9 ‰** | 63.9 | 18.2 | 8.4 |

---

### 4.4 Notebook 4: Agrupamento de Trajetórias Temporais (Estágio 2)
* **Arquivo:** `04_modelagem_series_temporais.ipynb`
* **Entrada:** `new_data/wpp2024_estados_mortalidade.csv`
* **Status Atual:** Em construção (o usuário acabou de plotar o **Indexplot** de sequências).

#### Estrutura do que já foi construído:
1. **Construção da Matriz de Sequências:**
   - Operação de `pivot_table`: Linhas = 161 Países, Colunas = 74 Anos (1950 a 2023), Células = Estado de Mortalidade $\in \{0, 1, 2, 3\}$.
2. **Visualização por Indexplot (Gráfico de Tapete Demográfico):**
   - Paleta de cores semântica:
     - 🔴 Estado 0: Vermelho escuro (`#D62839`)
     - 🟠 Estado 1: Laranja (`#F29E4C`)
     - 🟡 Estado 2: Amarelo (`#F1D302`)
     - 🟢 Estado 3: Verde escuro floresta (`#2E8B57`)
   - Ordenação pela soma cumulativa dos estados: Permite enxergar com clareza visual espetacular a "onda verde" varrendo o globo da direita para a esquerda ao longo das décadas, enquanto países da África Subsaariana e zonas de conflito permanecem estagnados nas faixas vermelha e laranja até anos recentes.

---

## 🎯 5. O que Falta Fazer para Concluir com Sucesso o Projeto (Roteiro para a Competição)

Para fechar o **Notebook 4** e preparar a submissão final do **CIRS DataQuest 2026**, os seguintes passos devem ser executados na sequência:

### Passo 1: Cálculo da Matriz de Dissimilaridade de Séries Temporais
- Como todas as 161 séries têm o mesmo comprimento temporal (74 anos) e valores discretos ordinais ($0 \le s \le 3$), as duas melhores métricas são:
  1. **Distância de Hamming com Pesos Ordinais (ou Manhattan / Cityblock entre sequências):** Mede a diferença absoluta acumulada entre os estados ano a ano.
  2. **Dynamic Time Warping (DTW):** Permite comparar trajetórias que realizaram a mesma transição, mas com defasagem temporal (ex: países asiáticos que fizeram nos anos 80 o que a Europa fez nos anos 50).

### Passo 2: Clusterização Final dos Países (Estágio 2)
- Aplicar **Hierarchical Agglomerative Clustering (linkage='ward' ou 'complete')** ou **K-Medoids (PAM)** sobre a matriz de distâncias de trajetórias.
- Avaliar o número ideal de macro-clusters de trajetórias (tipicamente entre $K=3$ e $K=5$ clusters de países).
- **Arquétipos Esperados:**
  - *Cluster A (Pioneiros / Fronteira Contínua):* Europa Ocidental, América do Norte, Oceania e Japão (já começam no Estado 1 ou 2 em 1950 e passam quase todo o pós-guerra no Estado 3).
  - *Cluster B (Transição Rápida / Catch-up Asiático e Latino):* Começam no Estado 0 em 1950 e sobem rapidamente para o Estado 2/3 entre 1970 e 2000 (Coreia do Sul, Chile, Costa Rica, China).
  - *Cluster C (Leste Europeu / Estagnação da Meia-Idade):* Alcançam o Estado 2 cedo, mas ficam presos ou sofrem regressões durante os anos 1990 devido à crise pós-soviética.
  - *Cluster D (Retardatários / Choque Epidêmico e Retomada Tardia):* África Subsaariana e partes do Sul da Ásia (longa permanência no Estado 0, transição para o Estado 1 após 2000 com o combate ao HIV/Malária).

### Passo 3: Visualização Cartográfica e Geográfica
- Cruzamento com as bibliotecas recém-adicionadas (`geopandas` e `pycountry`):
  - Gerar um mapa-múndi coroplético colorindo cada país pelo seu **Cluster de Trajetória do Estágio 2**.
  - Evidenciar visualmente a divisão geopolítica e a correlação espacial das transições demográficas.

### Passo 4: Tradução em Narrativa Executiva para o Relatório da Competição
- A American Statistical Association (ASA) valoriza profundamente a **tradução da modelagem em recomendações pragmáticas de políticas públicas**:
  - **Foco em $Q_5$ nos países do Estado 0 e 1:** Saneamento básico, vacinação infantil e nutrição básica geram os maiores saltos de expectativa de vida por dólar investido.
  - **Foco em Doenças Crônicas e Mortes Externas nos Estados 2 e 3:** Controle de fatores de risco cardiovascular, saúde mental, redução de acidentes de trânsito e violência urbana para destravar ganhos na meia-idade.
  - **Sustentabilidade dos Sistemas Previdenciários e de Cuidados de Longa Duração no Estado 3:** O rápido aumento de $LE_{65}$ e $LE_{80}$ sem contrapartida de fertilidade gera o desafio fiscal do envelhecimento populacional acelerado.

---

## 💡 6. Conclusão da Análise

A decisão de criar a pasta `notebooks_v3` foi **o divisor de águas do projeto**. A estrutura atual é elegante, robusta, computacionalmente reproduzível e com legitimidade teórica inquestionável. O pipeline de código está limpo, bem documentado e pronto para a finalização matemática do Estágio 2 e redação da narrativa campeã para o CIRS DataQuest 2026.
