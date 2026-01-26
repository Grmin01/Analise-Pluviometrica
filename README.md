Pipeline de Precipitação (1994–2024) — Estação 83698 × ERA5 × ETA-BESM

Pipeline em R para padronizar, comparar e gerar produtos (tabelas + gráficos) a partir de séries de precipitação de:

Estação (83698): dados diários → agregação anual e mensal

ERA5-Land: série anual (tp_anual.csv) e mensal (opcional)

ETA-BESM: série anual (Hist/RCP4.5/RCP8.5) e mensal long por cenário

Período padrão: 1994–2024.

Sumário

1. Requisitos

2. Configuração rápida

3. Entradas esperadas

4. Saídas geradas

5. Gráficos gerados (checklist)

6. Observações importantes

7. Problemas comuns (FAQ)

1. Requisitos
R e pacotes

O script exige:

readr, readxl, dplyr, tidyr, stringr, purrr, lubridate

ggplot2, writexl, tibble

Kendall (obrigatório)

trend (obrigatório)

plotrix (opcional, somente para o Gráfico 04 — Taylor)

Instalação (se precisar):

install.packages(c(
  "readr","readxl","dplyr","tidyr","stringr","purrr","lubridate",
  "ggplot2","writexl","tibble","Kendall","trend","plotrix"
))

2. Configuração rápida

No topo do script, ajuste os parâmetros:

YEAR_MIN      <- 1994
YEAR_MAX_HIST <- 2024


E principalmente a seção de caminhos:

DIR_ESTACAO <- "C:/.../01_DADOS ESTAÇÃO"
DIR_ETA     <- "C:/.../02_ETA_BESM_pluviometria_pronto"
DIR_ERA5    <- "C:/.../ERA5_LAND_BAIXADO"
DIR_OUTROOT <- "C:/.../05_RESULTADOS"

ESTACAO_FORCADA <- "C:/.../dados_83698_H_1994-01-01_2024-12-31.xlsx"
ETA_FORCADO     <- "C:/.../ETA_BESM_ANUAL_1994_2064.csv"
ERA5_FORCADO    <- "C:/.../tp_anual.csv"

ETA_MENSAL_OU_DIARIO_FORCADO <- "C:/.../ETA_pluv_long.csv"

ERA5_MENSAL_FORCADO <- ""  # opcional


Execute o script inteiro no RStudio.

3. Entradas esperadas
3.1 Estação (diária) — .xlsx/.csv (obrigatório)

Precisa ter:

coluna de data parecida com: data, date, dia, data_medicao

coluna de precipitação parecida com: precip, chuva, rain, mm

O script tenta datas em: ymd, dmy, ymd_hms.

3.2 ERA5 anual — .csv (opcional, mas recomendado)

Campos esperados:

ano: ano / year / yyyy

precip: valor / tp / precipitacao / precip / pr / p

Unidade: se os valores parecerem estar em metros, o script converte automaticamente para mm (× 1000).

3.3 ETA-BESM anual — .csv/.xlsx (opcional)

Campos esperados:

ano: ano / year / yyyy

cenário: cenario / scenario

precip: precipitacao / precip / tp / valor / mm

Cenários são normalizados para:

Historico

RCP 4.5

RCP 8.5

3.4 ETA mensal long — .csv (opcional; necessário p/ gráficos 18–20)

Campos esperados:

cenário: cenario/scenario

precip mensal: prec_mm / prec / precipitacao / valor / mm

e ou:

data/date (extrai ano/mês)

ou ano + mes

4. Saídas geradas

O script cria uma pasta com timestamp em DIR_OUTROOT:

05_RESULTADOS/
  └── YYYY-MM-DD_HHhMM/
      ├── _tabelas_csv/
      ├── _qa_preprocessamento/
      ├── GRAFICO_01_...
      ├── GRAFICO_02_...
      └── ...

4.1 Tabelas (em _tabelas_csv/)

TABELA_06_ERA5_ANUAL_PADRONIZADA_1994_2024.csv (se ERA5 existir)

TABELA_07_ESTACAO_ANUAL_PADRONIZADA_1994_2024.csv

TABELA_09_ESTATISTICAS_DESCRITIVAS_ANUAL_1994_2024.csv + .xlsx

TABELA_10_ZEROS_ANUAL_DIARIO.csv + .xlsx

TABELA_11_METRICAS_UNIFICADAS.csv + .xlsx

Extras (quando aplicável):

ERA5_ANUAL_1994_2024.csv

ESTACAO_ANUAL_1994_2024.csv

ESTACAO_MENSAL_1994_2024.csv

ERA5_MENSAL_1994_2024.csv (se ERA5 mensal existir)

ETA_BESM_HIST_ANUAL_1994_2024.csv (se ETA existir)

PARES_ERA5_ESTACAO_1994_2024.csv (se houver pareamento)

PARES_ETA_ESTACAO_1994_2024.csv (se houver pareamento)

5. Gráficos gerados (checklist)

A existência depende dos dados disponíveis.

Validação

 GRAFICO_01_DISP_ERA5_VS_ESTACAO_1994_2024.png

 GRAFICO_02_DISP_ETA_HIST_VS_ESTACAO_1994_2024.png

 GRAFICO_03_QQ_ERA5_VS_ESTACAO_1994_2024.png

 GRAFICO_04_TAYLOR_ERA5_VS_ESTACAO_1994_2024.png (plotrix)

Mensal / Heatmap

 GRAFICO_05_CLIMA_MENSAL_ERA5_1994_2024.png (ERA5 mensal)

 GRAFICO_06_CLIMA_MENSAL_ESTACAO_1994_2024.png

 GRAFICO_07_HEATMAP_ERA5_1994_2024.png (ERA5 mensal)

 GRAFICO_08_HEATMAP_ESTACAO_1994_2024.png

Distribuições / Comparações

 GRAFICO_09_DENSIDADE_3BASES_1994_2024.png

 GRAFICO_13_DISTRIB_ANUAL_FONTES_1994_2024.png

 GRAFICO_21_PRECIP_ANUAL_FONTES_1994_2024.png

Anomalias

 GRAFICO_10_ANOMALIA_ERA5_1994_2024.png

 GRAFICO_11_ANOMALIA_ESTACAO_1994_2024.png

 GRAFICO_12_ANOMALIA_ETA_CENARIOS_1994_2024.png

 GRAFICO_17_ANOMALIA_POR_CENARIO_BASELINE_ESTACAO_1994_2024.png

Média móvel (5 anos)

 GRAFICO_14_MM5_ERA5_1994_2024.png

 GRAFICO_15_MM5_ESTACAO_1994_2024.png

 GRAFICO_16_MM5_ETA_CENARIOS_1994_2024.png

ETA mensal por cenário

 GRAFICO_18_HEATMAP_ETA_HIST_1994_2024.png

 GRAFICO_19_HEATMAP_ETA_RCP45_1994_2024.png

 GRAFICO_20_HEATMAP_ETA_RCP85_1994_2024.png

Mensal por década

 GRAFICO_22_MENSAL_POR_DECADA_ERA5_1994_2024.png (ERA5 mensal)

 GRAFICO_23_MENSAL_POR_DECADA_ESTACAO_1994_2024.png

6. Observações importantes
ERA5: metros vs mm

ERA5 (tp) pode vir em metros. O script verifica a mediana e, se parecer “muito pequeno”, converte para mm.

ETA “costurado” (anual)

Para comparações anuais (1994–2024), o ETA vira duas linhas:

ETA-BESM (Hist+RCP 4.5) = Histórico (1994–2005) + RCP 4.5 (2006–2024)

ETA-BESM (Hist+RCP 8.5) = Histórico (1994–2005) + RCP 8.5 (2006–2024)

ETA base única (densidade)

No Gráfico 09, ETA é tratado como “uma base”:

1994–2005: Histórico

2006–2024: média anual entre RCP 4.5 e RCP 8.5

7. Problemas comuns (FAQ)

1) “🚩 Pacotes faltando …”
Instale com install.packages(...).

2) “Não encontrei coluna …”
O cabeçalho do seu arquivo não bate com os padrões. Renomeie as colunas ou amplie os padrões em .pick_col().

3) “não consegui interpretar a coluna de data”
Padronize para YYYY-MM-DD (ex.: 1994-01-01).

4) Não gerou Gráfico 04 (Taylor)
Instale plotrix.

5) Não gerou Gráficos 5/7/22 (ERA5 mensal)
Você não tem o CSV mensal; defina ERA5_MENSAL_FORCADO ou garanta que exista um arquivo com “mensal/tp_mensal” próximo ao anual para o auto-discovery.
