<div align="center">
  <img src="assets/header.svg" alt="VeroData Header" width="100%">

  <br><br>

  <!-- Badges de Tecnologia -->
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white" alt="Jupyter">
  <img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white" alt="Pandas">
  <img src="https://img.shields.io/badge/Scikit--Learn-F7931E?style=for-the-badge&logo=scikit-learn&logoColor=white" alt="Scikit-Learn">
</div>

---

## 🌍 Sobre o Projeto
Repositório oficial da equipe **VeroData** para o campeonato **CIRS DataQuest 2026**, organizado pela American Statistical Association (ASA). 

Nosso objetivo é aplicar técnicas de análise de séries temporais, inferência estatística e modelagem preditiva sobre os dados globais do *World Population Prospects 2024 (WPP2024)* da ONU.

## 📂 Estrutura do Repositório

```text
Dataquest-VeroData/
├── assets/                     # Elementos visuais institucionais (SVG, badges)
├── data/                       # Datasets oficiais WPP 2024 e malha Natural Earth
├── docs/                       # Diretrizes da competição e notas metodológicas
├── notebooks_v2/               # Pipeline analítico modular (Engenharia de Dados e Análises)
│   ├── _common.py              # Biblioteca compartilhada de funções e estilos
│   ├── 00_Engenharia_de_Dados.ipynb
│   ├── 01_Analise_Historica.ipynb
│   ├── 02_Projeções_e_Incerteza.ipynb
│   └── _src/                   # Scripts de compilação automatizada
├── relatorio_v2/               # Relatório executivo, figuras em alta resolução e tabelas
├── requirements.txt            # Dependências Python para reprodução rápida
├── environment.yml             # Configuração para ambientes Conda
├── INSTRUCOES_REPRODUCIBILIDADE.md # Guia passo a passo para a equipe
└── README.md
```

## 🚀 Como Executar o Projeto Localmente

Consulte o [Guia de Reprodução Local](INSTRUCOES_REPRODUCIBILIDADE.md) para detalhes completos.

```bash
# 1. Instalar as dependências
pip install -r requirements.txt

# 2. Executar e atualizar todos os notebooks e relatórios automaticamente:
python notebooks_v2/_src/build.py 00 01 02
```

## 📊 Dicionário de Dados Rápido

<details>
  <summary><b>Clique para expandir o Dicionário das Variáveis WPP2024</b></summary>
  <br>
  
  | Variável | Tipo | Descrição |
  |---|---|---|
  | `LocID` | Numérica | Identificador único da localidade/país |
  | `Time` | Temporal | Ano da observação ou projeção |
  | `TPopulation1Jan` | Contínua | População total em 1º de Janeiro (em milhares) |
  | `LEx` | Contínua | Expectativa de vida ao nascer (ambos os sexos) |
  | `TFR` | Contínua | Taxa de Fecundidade Total (filhos por mulher) |
  
  *Para a lista completa, consulte o arquivo `WPP2024_Demographic_Indicators_notes.csv` na pasta `data/`.*
</details>

<br>

## 👨‍💻 Equipe
* **[Caio Lima/@CAIO-LIMA-37]**
* **[Pedro Henrique]**
* **[Rodrigo Graim]**
