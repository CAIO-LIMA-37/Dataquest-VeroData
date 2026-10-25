# 🔎 Avaliação do Pré-processamento × Países ausentes no mapa

**Pergunta:** França (e talvez Dinamarca/Finlândia) aparecem cinza no mapa. O erro está no pré-processamento do Notebook 1?

## 1. Veredito

**Não. O pré-processamento NÃO eliminou França, Dinamarca, Finlândia nem Noruega.** Eles estão nos dados, com todos os 74 anos, classificados corretamente.
O problema quase certamente está no **cruzamento (merge) entre os dados e a malha geográfica do mapa**, que é uma etapa posterior.

## 2. Evidências (checadas direto em `new_data/wpp2024_estados_mortalidade.csv`, ano 2023)

| País | ISO3 | População (mil) | Estado em 2023 |
|---|---|---|---|
| França | FRA | 66.438 | 3 |
| Dinamarca | DNK | 5.948 | 3 |
| Finlândia | FIN | 5.601 | 3 |
| Noruega | NOR | 5.519 | 3 |
| Suécia | SWE | 10.551 | 3 |
| Kosovo | **XKX** | 1.700 | 3 |
| Chipre | CYP | 1.344 | 3 |

- A população está em **milhares**: o corte `>= 1000` equivale a 1 milhão. Todos acima passam com folga.
- Contagem de linhas de 2023 no arquivo de estados: 159 com nome simples + 2 com nome entre aspas (`"China, Hong Kong SAR"` e `"China, Taiwan Province of China"`) = **161 países**, o número esperado.
- Notebook 1 imprime 161 países ao final e a matriz tem 161 × 151 = 24.311 linhas, sem buracos.

## 3. Causa provável: a malha do mapa (Natural Earth)

Problema **muito conhecido** no Natural Earth: a coluna `ISO_A3` vem como `-99` para alguns países. Se o merge foi feito por `ISO_A3`, eles ficam sem dados e viram cinza:

| País | `ISO_A3` na malha | Observação |
|---|---|---|
| **França** | `-99` | Também deixa a Guiana Francesa cinza (ela é parte do polígono da França) |
| **Noruega** | `-99` | Pode ser o que você viu na Escandinávia |
| **Kosovo** | `-99` | Nos dados da ONU o código é `XKX` |
| Chipre do Norte, Somalilândia | `-99` | Sem dados na ONU de qualquer forma |

**Dinamarca e Finlândia** normalmente vêm com `DNK` e `FIN` corretos. O que costuma enganar:
- **Groenlândia** é um polígono separado e fica cinza (não tem 1 milhão de habitantes, está fora da amostra). Visualmente parece "faltar a Dinamarca".
- Noruega cinza ao lado pode parecer Finlândia/Dinamarca em mapas pequenos.

> Na imagem, vejo cinza na Groenlândia, na costa nordeste da América do Sul (Guianas) e manchas na Europa. Mas em resolução tão pequena **não consigo confirmar com certeza quais países europeus** são. Por isso a checagem abaixo é importante.

**Como corrigir:** fazer o merge por `ADM0_A3` (ou `ISO_A3_EH`, se existir) em vez de `ISO_A3`, e tratar Kosovo manualmente (`XKX` ↔ `KOS`).

Se o ISO3 foi gerado com `pycountry` a partir de nomes, isso também falha para nomes da ONU como `Türkiye`, `Dem. People's Republic of Korea`, `Kosovo (under UNSC res. 1244)`, `Côte d'Ivoire` etc. Como seus dados **já têm** `Codigo_ISO3`, não precisa de `pycountry`.

## 4. Checagem rápida (cole numa célula)

```python
# Quem está nos dados mas NÃO casou com a malha?
dados = set(df_estados['Codigo_ISO3'].unique())
malha = set(world['ISO_A3'])          # ajuste para o nome da sua coluna/variável
print("Nos dados, fora da malha:", sorted(dados - malha))
print("Malha com -99:", world.loc[world['ISO_A3'] == '-99', 'NAME'].tolist())
```

A primeira lista deve trazer FRA, NOR, XKX (e talvez outros). Isso confirma o diagnóstico.

## 5. Outros pontos do pré-processamento (não causam o mapa, mas valem atenção)

1. **Inconsistência de nomes de colunas entre Notebook 1 e o arquivo salvo.** O Notebook 1 salva `df_clean` com colunas originais (`Location`, `ISO3_code`, `Time`, `TPopulation1July`, `LEx`, `Q5`...). Porém `wpp2024_clean.csv` tem `Pais`, `Codigo_ISO3`, `Ano`, `Populacao_Total`, `Exp_Vida_Nascer`... e os Notebooks 2–4 usam os nomes em português. **Nenhuma célula do Notebook 1 faz esse `rename`.** Rodando o 1 do zero, os notebooks seguintes quebrariam. Sugestão: adicionar um `df_clean.rename(columns={...})` antes de salvar.
2. **Corte de 1 milhão só em 2023.** Correto e coerente com a literatura, mas países que tinham <1M em 2023 (Luxemburgo, Malta, Islândia, Guiana, Suriname, Butão, Fiji...) ficam fora, e aparecem cinza *de propósito*. A legenda "Sem dados / Fora da amostra" cobre isso.
3. **Célula 1.4 deixa o resultado "0 países" como saída gravada.** É didático, mas confunde quem lê depois. Vale um aviso explícito de que aquela célula é um contra-exemplo.
4. **`print("Total de países inicial: 161")` fixo.** Deveria usar `df_filtered['ISO3_code'].nunique()`.
5. **Pontos positivos:** `keep_default_na=False` evita o clássico bug de "NA" (Namíbia) ser lido como nulo; `utf-8-sig`, detecção de separador e `Time <= 2100` estão corretos.

## 6. Resumo

| Item | Status |
|---|---|
| França/Dinamarca/Finlândia/Noruega nos dados | ✅ Presentes, 74 anos |
| Filtro de 1 milhão | ✅ Correto |
| 161 países × 151 anos | ✅ Íntegro |
| Merge com a malha por `ISO_A3` | ⚠️ Provável causa (França, Noruega, Kosovo = `-99`) |
| `rename` de colunas ausente no Notebook 1 | ⚠️ Reprodutibilidade |

**Limitação:** o código que gerou o mapa não está salvo em nenhum notebook que eu consegui ler, então a causa do merge é a hipótese mais forte, não uma confirmação. Se me mostrar a célula do mapa, confirmo em um minuto.
