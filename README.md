# Analise-Pluviometrica

Pipeline completo para **análise pluviométrica** de Campos dos Goytacazes (RJ) com três etapas integradas:

1. **Extração ETA_BESM 20 km** → precipitação **diária** (mm/dia) a partir de arquivos NetCDF.  
2. **Mapa/Contexto ERA5 + Estação INMET** → base cartográfica e validação espacial.  
3. **Processamento Pluviométrico Integrado** → unificação (ERA5, Estação, ETA_BESM), QA, estatísticas (MK, Sen, Pettitt), anomalias e gráficos.

> **Janelas de análise**: Histórico **1994–2024** e Cenários **2006–2064** (RCP 4.5/8.5).  
> **Alvo**: Município de Campos dos Goytacazes (−21.7, −41.3).

---

## ⚙️ Dependências (R)

- Leitura/arrumação: `readr`, `readxl`, `dplyr`, `tidyr`, `stringr`, `stringi`, `lubridate`, `tibble`, `zoo`
- NetCDF: `ncdf4`
- Estatística: `Kendall`, `trend`
- Gráficos: `ggplot2`, `plotrix` (Taylor), `ggspatial`
- Dados geográficos: `sf`, `geobr`
- Exportação: `writexl`

Instalação rápida (R):

```r
pkgs <- c("readr","readxl","dplyr","tidyr","stringr","stringi","lubridate","tibble","zoo",
          "ncdf4","Kendall","trend","ggplot2","plotrix","sf","geobr","ggspatial","writexl")
to_install <- setdiff(pkgs, rownames(installed.packages()))
if (length(to_install)) install.packages(to_install, dependencies = TRUE)
```

---

## 🗂️ Estrutura sugerida de pastas

```
Analise-Pluviometrica/
├── 1-EXTRACAO_ ETA_BESM_20km.R
├── 2-Extrair_ERA5.R
├── 3-Processamento-Pluviometico.R
├── dados/                    # planilhas e csvs de entrada (ERA5/Estação/ETA_BESM)
├── Eta_BESM_20km/            # NetCDFs ETA_BESM 20 km
├── resultados/               # saídas (png/csv/xlsx) com carimbo de data/hora
└── mapas/                    # mapas gerados (png/pdf)
```

> Os scripts criam automaticamente subpastas carimbadas em `resultados/` e `dados/` para cada execução.

---

## 🔄 Fluxo (visão geral)

```text
ETA_BESM (NetCDF) ──► (1) Extração diária (mm/dia) ──► CSVs por arquivo + combinado
                                           │
ERA5 + Estação INMET ──► (2) Mapa/Contexto │
                                           ▼
                              (3) Processamento Integrado
                 ► QA e checagens (unidades, zeros, NA, pareamentos)
                 ► Séries históricas e RCPs coerentes
                 ► Estatísticas (Mann-Kendall, Sen, Pettitt)
                 ► Anomalias (baseline 1994–2024)
                 ► Produtos: heatmaps, boxplots, séries e comparativos
```

---

## 🚀 Passo a passo

### 1) Extração ETA_BESM 20 km — `1-EXTRACAO_ ETA_BESM_20km.R`

**O que faz**  
Lê arquivos **NetCDF** (ETA_BESM 20 km), recorta por **bounding box** local, converte a precipitação para **mm/dia**, força vetores atômicos (evita colunas-lista) e exporta **CSV diário por arquivo** + **CSV combinado**.

**Principais funções**

- `norm_bbox_to_grid(lon_vec, lon_rng)` → Normaliza o *bbox* ao sistema de longitudes da grade (0–360 vs −180–180).
- `decode_time(time_vals, time_units)` → Decodifica o eixo de tempo dos NetCDF e devolve `Date` **vetorial** (crítico para evitar matrizes).
- `find_pr_var(nc)` → Descobre a variável de precipitação entre (`pr`, `precip`, `precipitation`, `tp`) ou por **unidade física**.
- `to_mm_per_day(values, units_text)` → Converte `kg·m⁻²·s⁻¹` para **mm/dia** e trata `mm/h`, `mm/dia`, etc.
- `sanity_fix(mm_day_vec)` → Corrige conversões duplicadas (e.g., divide por 86.400 quando mediana diária > 100 mm).
- `extract_eta_daily_one(nc_path, lat_range, lon_range)` → Extrator robusto: mapeia dims `time/lat/lon`, calcula médias espaciais no *bbox* para cada dia e retorna `data, mm_dia`.
- `parse_scenario(fname)` → Identifica cenário pelo nome do arquivo (`Histórico`, `RCP 4.5`, `RCP 8.5`).

**Saídas**

- `*_daily_campos.csv` (um por NetCDF) e `ETA_BESM_daily_campos.csv` (combinado).  
- **Resumo** por arquivo: número de dias, período, média mm/dia, equivalente anual.

---

### 2) Contexto ERA5/Estação — `2-Extrair_ERA5.R`

**O que faz**  
Gera o **mapa de localização** da Estação INMET **Campos (A603)**, com **buffer** (~10 km), grade, setas de norte e escala. Serve para **validar espacialmente** as séries usadas.

**Blocos/funções destacáveis**

- **Tema de mapa** (`theme_map`) → fundo oceânico, tons neutros para polígonos municipais.  
- `geobr::read_municipality("RJ")` → Base municipal do RJ (2020).  
- `sf` → Cria *feature* da estação, reprojeta para UTM 24S (`31984`) e faz **buffer**.  
- `ggspatial::annotation_north_arrow` e `annotation_scale` → elementos cartográficos.  
- `ggsave()` → exporta **PNG** e **PDF** (com *fallback* se `cairo_pdf` indisponível).

**Saídas**

- `mapa_estacao_inmet_campos.png` e `mapa_estacao_inmet_campos.pdf` (pasta atual).

---

### 3) Processamento Pluviométrico Integrado — `3-Processamento-Pluviometico.R`

**O que faz**  
Integra **ERA5**, **Estação INMET (83698)** e **ETA_BESM**; padroniza, corrige **unidades**, delimita **janelas coerentes** (1994–2024 / 2006–2064), calcula **estatísticas de tendência** e **anomalias**, e produz **gráficos** e **relatórios de qualidade**.

**Principais utilitários**

- `._normalize_names(nms)` → Normaliza nomes das colunas (ASCII, minúsculas, *snake_case*).  
- `._pick_col(df, patterns, required, hint)` → Localiza coluna por **regex** (tolerante a variações).  
- `._read_csv_smart(path)` → Detecta delimitador `;`/`,` e `decimal_mark` (`,`/`.`).  
- `._limit_anos(df, col, min, max)` → Recorte temporal coerente por fonte.  
- `._normalize_cenario(x)` → Padroniza rótulos: `Historico`, `RCP 4.5`, `RCP 8.5`.  
- `._fix_unidade_anual(df, col, fonte)` → Corrige **unidade suspeita** (e.g., ÷ 1000 quando mediana anual > 10.000).

**Leituras e pads**

- **ERA5** (`csv`/`xlsx`): detecta `anual` ou reconstrói a partir do **diário**; exporta `ERA5_ANUAL_1994_2024.csv`.  
- **Estação 83698** (`xlsx`): função `ler_estacao_diaria()` reconstitui diário e anual, com **QA de unidade**.  
- **ETA_BESM**: usa `ETA_BESM_ANUAL_1994_2064.csv` se existir; caso contrário agrega a partir dos diários `*_daily_campos.csv`.

**Estatísticas e inferências**

- `._calc_all_stats(x)` → **Mann-Kendall** (τ, p), **Sen’s slope** (mm/ano) e **Pettitt** (ponto de mudança).  
  - Converte o índice do Pettitt (`cp`) para **ano** de forma segura (`cp_year`).  
- Saídas unificadas em `estatisticas_unificadas.csv/.xlsx`, incluindo **escopo** (por cenário e por base).

**Anomalias (baseline 1994–2024)**

- `calc_anom(df, ref)` → anomalia = `precipitacao - média(ref)`.  
- Gera **anomalias** para ERA5, Estação e ETA_BESM; além de **anomalia por cenário** (Histórico × RCPs).

**Gráficos gerados** (exemplos de nomes)

- `anomalia_precipitacao_cenarios_1994_2024_vs_2006_2064.png`  
- `anomalia_era5_ref_1994_2024.png`, `anomalia_estacao_ref_1994_2024.png`, `anomalia_eta_besm_ref_1994_2064.png`  
- `comparativo_tres_fontes_1994_2024.png` (ERA5 × Estação × ETA_BESM)  
- `boxplot_anual_fontes_hist1994_2024_rcp_ate_2064.png` e `densidade_anual_fontes_hist1994_2024_rcp_ate_2064.png`  
- `dispersao_ERA5_vs_Estacao.png`, `dispersao_ETA_Historico_vs_Estacao.png`  
- `taylor_diagram_era5_estacao_hist_1994_2024.png` e `qqplot_era5_vs_estacao_hist_1994_2024.png`  
- **Mensais** (se diário disponível): `heatmap_mensal_*`, `climatologia_mensal_*`, `boxplots_decadas_*`  
- **Média móvel (5a)**: `media_movel5_estacao.png`, `media_movel5_era5.png`, `media_movel5_eta_besm_1994_2064.png`

**Relatórios de Qualidade (QA)**

- `qualidade_overview_zeros_na.(csv|xlsx)` → contagem de zeros e ausentes por **fonte** e **nível** (anual/diário).  
- `qualidade_eta_cenario_anual.(csv|xlsx)` → métricas QA por **cenário** (ETA_BESM).  
- `qa_resumo_unidades_hist1994_2024_rcp2006_2064.csv` → resumo de unidades/ordens de grandeza.

**Produtos mensais (se houver diário)**

- `make_month_table(diario_df, min_year, max_year)` → tabela ano×mês (mm/mês).  
- `plot_heatmap_mensal`, `plot_clima_mensal`, `plot_box_decadas` → visões sazonais/decadais.  
- `plot_media_movel(df_anual, label, fname, min, max)` → curva anual + média móvel 5a.

---

## 📑 Entradas & Saídas (resumo)

**Entradas**  
- `Eta_BESM_20km/*.nc` (precipitação diária)  
- Planilhas/CSVs de **ERA5** e **Estação INMET 83698** (anual/diário)  
- (Opcional) `ETA_BESM_ANUAL_1994_2064.csv`

**Saídas**  
- `Eta_BESM_20km/*_daily_campos.csv` + `ETA_BESM_daily_campos.csv`  
- `resultados/<timestamp>/...png` (figuras) e `dados/<timestamp>/*.csv|.xlsx`  
- `estatisticas_unificadas.csv/.xlsx`, `comparativos`, `anomalias`, `QA`

---

## 🧪 Boas práticas e QA embutido

- **Unidades**: correção automática (divisão por 1000 ou por 86.400 quando necessário) com logs.  
- **Coerência temporal**: recortes específicos para **Histórico** e **RCPs**.  
- **Pareamentos** para comparações (ERA5 × Estação; ETA histórico × Estação).  
- **Validação** com Taylor e QQ-plot quando houver dados pareados suficientes (≥ 5 anos).

---

## 🧯 Troubleshooting rápido

- **“Estação sem coluna de precip detectável”** → o script tenta fallback na 7ª coluna (G); avalie a planilha.  
- **Mediana anual absurda** (> 10.000 mm) → o código ajusta unidades; confirme fonte.  
- **ETA_BESM sem anual pronto** → o script agrega a partir do **diário** automático.  
- **Plotrix indisponível** → o Taylor diagram é pulado (demais produtos geram normalmente).  
- **Mapas sem `cairo_pdf`** → exporta PDF com *device* padrão.

---

## 📜 Licença

Sugerido: **MIT License** (adicione `LICENSE` conforme a sua preferência institucional).

---

## ✨ Créditos

- INMET / Estação A603 (Campos dos Goytacazes)  
- Copernicus ERA5 / ECMWF  
- ETA_BESM 20 km (INPE/CPTEC)  
- Comunidade R (pacotes citados acima)
