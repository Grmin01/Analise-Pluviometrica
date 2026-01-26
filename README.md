Este script executa um pipeline completo para padronizar, comparar e visualizar séries de precipitação entre:

Estação (83698): dados diários (entrada) → agregação anual e mensal

ERA5-Land: série anual (obrigatória se existir) e mensal (opcional)

ETA-BESM: série anual (Histórico + cenários RCP) e série mensal long (por cenário)

O foco do pipeline é o período 1994–2024 (configurável), gerando tabelas padronizadas, métricas de validação, estatísticas, e 23 gráficos (dependendo da disponibilidade de arquivos).

1) O que o script gera

Ao final, o script cria uma pasta com timestamp em DIR_OUTROOT, contendo:

Gráficos (.png) na raiz da pasta de saída

Tabelas (.csv / .xlsx) na subpasta: /_tabelas_csv

(opcional/previsto) pasta /_qa_preprocessamento para QA (neste trecho do script ela é criada, mas não é preenchida)

Principais produtos

Tabelas

Tabela 06: ERA5 anual padronizada (1994–2024)

Tabela 07: Estação anual padronizada (1994–2024)

Tabela 09: Estatísticas descritivas anuais (1994–2024)

Tabela 10: Relatório de zeros e NA (anual e diário)

Tabela 11: Métricas de comparação pareada (correlação, viés, MAE, RMSE)

Gráficos

Validação (dispersão, QQ, Taylor)

Anomalias (ERA5, Estação, ETA por cenário)

Climatologia mensal + heatmaps

Comparação anual entre fontes (linhas)

Distribuições (boxplot anual, densidade)

Média móvel (5 anos)

Distribuição mensal por década

2) Requisitos
R (pacotes)

O script carrega (e falha se faltar) os pacotes:

readr, readxl, dplyr, tidyr, stringr, purrr, lubridate

ggplot2, writexl, tibble

Além disso:

Kendall (obrigatório)

trend (obrigatório)

plotrix (opcional — necessário apenas para o Gráfico 04 (Taylor))

Se plotrix não estiver instalado, o script avisa e não gera o gráfico Taylor.

Parâmetros principais

No topo do script:

YEAR_MIN      <- 1994
YEAR_MAX_HIST <- 2024

3) Estrutura de pastas (caminhos)

Edite a seção CAMINHOS (AJUSTE AQUI):

DIR_ESTACAO: pasta da Estação

DIR_ETA: pasta do ETA

DIR_ERA5: pasta do ERA5

DIR_OUTROOT: pasta onde será criada a saída com timestamp

O script está configurado para usar “travamentos” (caminhos forçados) com arquivos específicos:

ESTACAO_FORCADA (obrigatório)

ETA_FORCADO (anual) (se existir)

ERA5_FORCADO (anual) (se existir)

ETA_MENSAL_OU_DIARIO_FORCADO (obrigatório para heatmaps ETA mensais / gráficos 18–20)

ERA5_MENSAL_FORCADO (opcional; se vazio, o script tenta “descobrir” um mensal perto do anual)

4) Formato esperado dos arquivos de entrada
4.1 Estação (diária) — Excel/CSV

Entrada: ESTACAO_FORCADA

O script tenta localizar colunas por padrões (case-insensitive). Precisa de:

Data: uma coluna parecida com data, date, dia, data_medicao

Precipitação: uma coluna contendo algo como precip, chuva, rain, mm

A data é interpretada em ordem:

ymd

dmy

ymd_hms

Saída interna:

precipitacao_dia (mm/dia)

agregação anual: soma por ano → estacao_anual

4.2 ERA5 anual — CSV

Entrada: ERA5_FORCADO (ex.: tp_anual.csv)

Precisa de:

ano: ano, year, yyyy

precip: valor, tp, precipitacao, precip, rain, pr, p

Atenção unidade: o script detecta valores pequenos (mediana < ~10) como provável metros (tp do ERA5) e converte para mm multiplicando por 1000.

4.3 ETA-BESM anual — CSV/XLSX

Entrada: ETA_FORCADO

Precisa de:

ano: ano, year, yyyy

cenário: cenario, scenario

precip: precipitacao, precip, tp, rain, mm, valor

Normalização de cenário:

“hist” → Historico

variações de 4.5 → RCP 4.5

variações de 8.5 → RCP 8.5

4.4 ETA mensal (long) — CSV

Entrada: ETA_MENSAL_OU_DIARIO_FORCADO (ex.: ETA_pluv_long.csv)

Precisa de:

cenário: cenario/scenario

precip mensal: prec_mm, prec, precipitacao, valor, mm

E ou:

data/date (o script extrai ano/mês)

ou colunas ano + mes

5) Como executar

Ajuste os caminhos na seção CAMINHOS (AJUSTE AQUI).

Garanta que os pacotes estejam instalados.

Rode o script inteiro no R/RStudio.

Ao iniciar, ele cria uma pasta de saída com timestamp:

DIR_OUTROOT/YYYY-MM-DD_HHhMM/
  |_ _tabelas_csv/
  |_ _qa_preprocessamento/
  |_ (gráficos .png)


Se PRINT_OUTPUTS <- TRUE, o script imprime no console um checklist com o que foi gerado.

6) Saídas geradas (arquivos)
6.1 Tabelas (em /_tabelas_csv)

Sempre que houver dados, o script salva:

ESTACAO_ANUAL_1994_2024.csv

TABELA_07_ESTACAO_ANUAL_PADRONIZADA_1994_2024.csv

Se ERA5 anual existir:

ERA5_ANUAL_1994_2024.csv

TABELA_06_ERA5_ANUAL_PADRONIZADA_1994_2024.csv

ERA5_ANUAL_padronizado_raw.csv

Se ETA anual existir:

ETA_BESM_HIST_ANUAL_1994_2024.csv (apenas cenário histórico, limitado a 1994–2024)

Estatísticas e métricas:

TABELA_09_ESTATISTICAS_DESCRITIVAS_ANUAL_1994_2024.csv + .xlsx

TABELA_10_ZEROS_ANUAL_DIARIO.csv + .xlsx

TABELA_11_METRICAS_UNIFICADAS.csv + .xlsx

Pareamentos (se aplicável):

PARES_ERA5_ESTACAO_1994_2024.csv

PARES_ETA_ESTACAO_1994_2024.csv

Mensal:

ESTACAO_MENSAL_1994_2024.csv

ERA5_MENSAL_1994_2024.csv (se mensal existir)

7) Lista de gráficos (nomes e descrição)

Alguns gráficos dependem da existência das séries correspondentes.

Validação (ERA5/ETA vs Estação)

Gráfico 01 — Dispersão anual ERA5 vs Estação
GRAFICO_01_DISP_ERA5_VS_ESTACAO_1994_2024.png

Gráfico 02 — Dispersão anual ETA (Hist) vs Estação
GRAFICO_02_DISP_ETA_HIST_VS_ESTACAO_1994_2024.png

Gráfico 03 — QQ-plot empírico ERA5 vs Estação
GRAFICO_03_QQ_ERA5_VS_ESTACAO_1994_2024.png

Gráfico 04 — Diagrama de Taylor ERA5 vs Estação (requer plotrix)
GRAFICO_04_TAYLOR_ERA5_VS_ESTACAO_1994_2024.png

Climatologia e heatmaps mensais

Gráfico 05 — Climatologia mensal ERA5 (requer ERA5 mensal)
GRAFICO_05_CLIMA_MENSAL_ERA5_1994_2024.png

Gráfico 06 — Climatologia mensal Estação
GRAFICO_06_CLIMA_MENSAL_ESTACAO_1994_2024.png

Gráfico 07 — Heatmap mensal ERA5 (requer ERA5 mensal)
GRAFICO_07_HEATMAP_ERA5_1994_2024.png

Gráfico 08 — Heatmap mensal Estação
GRAFICO_08_HEATMAP_ESTACAO_1994_2024.png

Distribuições e comparação entre fontes

Gráfico 09 — Densidade anual (ERA5, Estação, ETA base única)
GRAFICO_09_DENSIDADE_3BASES_1994_2024.png

Gráfico 13 — Boxplot anual por fonte
GRAFICO_13_DISTRIB_ANUAL_FONTES_1994_2024.png

Gráfico 21 — Série anual por fonte (ERA5, Estação, ETA costurado em 2 linhas)
GRAFICO_21_PRECIP_ANUAL_FONTES_1994_2024.png

Anomalias

Gráfico 10 — Anomalia anual ERA5 (baseline 1994–2024 do próprio ERA5)
GRAFICO_10_ANOMALIA_ERA5_1994_2024.png

Gráfico 11 — Anomalia anual Estação (baseline 1994–2024 da própria Estação)
GRAFICO_11_ANOMALIA_ESTACAO_1994_2024.png

Gráfico 12 — Anomalia anual ETA com 2 cenários (baseline = média ETA Hist 1994–2005)
GRAFICO_12_ANOMALIA_ETA_CENARIOS_1994_2024.png

Gráfico 17 — Anomalia ETA por cenário (baseline = média Estação 1994–2024)
GRAFICO_17_ANOMALIA_POR_CENARIO_BASELINE_ESTACAO_1994_2024.png

Médias móveis (5 anos)

Gráfico 14 — MM5 ERA5
GRAFICO_14_MM5_ERA5_1994_2024.png

Gráfico 15 — MM5 Estação
GRAFICO_15_MM5_ESTACAO_1994_2024.png

Gráfico 16 — MM5 ETA costurado (2 linhas: Hist+RCP45 e Hist+RCP85)
GRAFICO_16_MM5_ETA_CENARIOS_1994_2024.png

ETA mensal por cenário (heatmaps)

Gráfico 18 — Heatmap ETA Histórico (1994–2024, eixo completo)
GRAFICO_18_HEATMAP_ETA_HIST_1994_2024.png

Gráfico 19 — Heatmap ETA RCP 4.5 (janela sem anos vazios 1994–2005; começa ≥ 2006)
GRAFICO_19_HEATMAP_ETA_RCP45_1994_2024.png

Gráfico 20 — Heatmap ETA RCP 8.5 (idem)
GRAFICO_20_HEATMAP_ETA_RCP85_1994_2024.png

Mensal por década

Gráfico 22 — Boxplot mensal por década (ERA5) (requer ERA5 mensal)
GRAFICO_22_MENSAL_POR_DECADA_ERA5_1994_2024.png

Gráfico 23 — Boxplot mensal por década (Estação)
GRAFICO_23_MENSAL_POR_DECADA_ESTACAO_1994_2024.png

8) Observações importantes

Recorte temporal: tudo é filtrado para YEAR_MIN…YEAR_MAX_HIST (padrão 1994–2024).

ERA5 em metros vs mm: há correção automática baseada na mediana dos valores.

ETA “costurado”:

1994–2005: Historico

2006–2024: RCP 4.5 ou RCP 8.5

Isso gera duas linhas de ETA (Hist+RCP45 e Hist+RCP85) nos gráficos comparativos.

Densidade (Gráfico 09): ETA vira uma base única:

1994–2005: Histórico

2006–2024: média entre RCP4.5 e RCP8.5 (ano a ano)

9) Solução de problemas (erros comuns)

“🚩 Pacotes faltando”: instale o pacote indicado (install.packages("...")).

“Não encontrei coluna …”: seu arquivo tem nomes de colunas fora dos padrões esperados.
Ajuste o cabeçalho do arquivo ou amplie os padrões dentro de .pick_col().

“não consegui interpretar data”: a coluna de data está em um formato não reconhecido.
Padronize para YYYY-MM-DD (recomendado).

Sem ERA5 mensal: gráficos 05, 07 e 22 não serão gerados.
