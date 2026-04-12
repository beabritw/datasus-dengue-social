# Projeto Data Major — Tópicos de Banco de Dados · IESB 2026

> **Predição de hospitalização em casos de dengue com base no perfil clínico e epidemiológico do paciente utilizando dados do SINAN**
> + Análise via Clusterização K-means  
> **Docente:** Rodrigo Gonçalves  

---

## Integrantes e Responsabilidades

- Beatriz Brito do Rosário - **Extract**
- Davi de Souza Lopes - **Transform**
- Lucas de Siqueira Cavalcanti - **Load**
- Lucas Montalvão Ramires - **Classificação**

---

## Estrutura de Pastas (Google Drive)

```
Topicos_BD/
├── raw/
│   ├── sinan/
│   │   └── sinan_dengue_raw/        ← parquets consolidados do SINAN (36.4 MB)
│   └── ibge/
│       └── ibge_municipios.parquet  ← tabela de municípios brasileiros
├── processed/                       ← saída do Transform
└── notebooks/                       ← notebooks .ipynb
```

> **IMPORTANTE — Acesso compartilhado:**  
> A pasta `Topicos_BD` está no Drive da Beatriz. Para executar qualquer etapa do pipeline sem precisar baixar os dados novamente, **solicite acesso de edição a ela e adicione a pasta como atalho no seu próprio Drive.** Cada notebook do pipeline (Transform, Load, Classificação) já aponta para esse mesmo caminho.

Cada etapa lê da pasta anterior e escreve na sua. 

---

## Etapa 1 — Extract (`extract.ipynb`)

**Responsável:** Beatriz Brito  
**Fontes:** DATASUS (FTP público) + IBGE (API pública)  
`raw/sinan/sinan_dengue_raw/` e `raw/ibge/ibge_municipios.parquet`

### O que foi extraído

| Fonte | Registros | Formato salvo |
|---|---|---|
| SINAN / DATASUS | 6.102.102 | Parquet (36.4 MB) |
| IBGE API | 5.571 municípios | Parquet |

### Fonte 1 — SINAN (DATASUS)

O SINAN (Sistema de Informação de Agravos de Notificação) é o sistema federal onde médicos notificam casos de doenças. Os dados de dengue ficam disponíveis em um servidor FTP público do Ministério da Saúde.
A biblioteca `pysus` acessa o FTP do DATASUS, baixa os arquivos `.DBC` (formato binário proprietário do DATASUS) e os converte automaticamente para `.parquet`. São 104 arquivos no total.
Com 6,1 milhões de registros e 121 colunas, o Pandas carregaria tudo em memória de uma vez e travaria o Colab. O Spark lê os 104 parquets em paralelo como um único DataFrame distribuído.

**Período coletado: 2022 e 2023**  
Esses são os dois anos mais recentes com dados completos. Anos mais recentes ainda têm subnotificação por atraso de fechamento dos registros. O período também captura o grande surto de dengue que o Brasil enfrentou nesses anos, tornando o dataset rico e representativo.

**Seleção de colunas:**  

| Grupo | Colunas |
|---|---|
| Variável alvo | `HOSPITALIZ` |
| Perfil epidemiológico | `CS_SEXO`, `NU_IDADE_N`, `ID_MUNICIP`, `ID_OCUPA_N`, `NU_ANO` |
| Sinais clínicos | `FEBRE`, `MIALGIA`, `CEFALEIA`, `VOMITO`, `NAUSEA`, `DOR_RETRO`, `DOR_COSTAS`, `EXANTEMA`, `LEUCOPENIA` |
| Comorbidades | `DIABETES`, `HIPERTENSA`, `RENAL`, `HEPATOPAT`, `HEMATOLOG` |
| Classificação e desfecho | `CLASSI_FIN`, `EVOLUCAO` |

Quantidade de memória: **36,4 MB**.

### Fonte 2 — IBGE (API pública)

```
https://servicodados.ibge.gov.br/api/v1/localidades/municipios
```

O campo `ID_MUNICIP` do SINAN é um código numérico sem nome legível. A tabela do IBGE é o "dicionário" que traduz esse código para nome do município, UF e região — essencial para análises geográficas e para calcular taxas de hospitalização por localidade.

**Detalhe importante — diferença de dígitos:**  
O IBGE usa código de 7 dígitos (`5300108`) e o SINAN usa 6 dígitos (`530010`). O 7º dígito do IBGE é um dígito verificador que o SINAN não registra. O Extract já cria a coluna `id_municipio_sinan` com os primeiros 6 dígitos, resolvendo isso preventivamente para o Transform:

```python
df['id_municipio_sinan'] = df['id_municipio'].str[:6]
```

---

## 🔧 Etapa 2 — Transform (`transform.ipynb`)

**Responsável:** Davi de Souza Lopes  
**Lê de:** `raw/`  
**Salva em:** `processed/`

### Como ler os arquivos

```python
# SINAN — leia com PySpark
df_sinan = spark.read.parquet('raw/sinan/sinan_dengue_raw')

# IBGE — leia com pandas
df_ibge = pd.read_parquet('raw/ibge/ibge_municipios.parquet')
```

### Problemas críticos a resolver

**1. Variável alvo `HOSPITALIZ`** 
**2. Campos binários — padronização obrigatória**
**3. Campo de idade `NU_IDADE_N`**
**4. Join SINAN × IBGE**
**5. Outros**
- Remover `CLASSI_FIN = 5` (casos descartados de dengue)
- Verificar nulos em todas as colunas e tratar com justificativa
- Converter tipos onde necessário (datas, numéricos)


---

Todas as etapas rodam no **Google Colab** com acesso ao Google Drive montado.

---

## 📎 Referências

- [Documentação PySUS / SINAN](https://pysus.readthedocs.io/en/latest/databases/SINAN.html)
- [API IBGE Localidades](https://servicodados.ibge.gov.br/api/docs/localidades)
- [Dicionário de dados SINAN Dengue — DATASUS](https://datasus.saude.gov.br/transferencia-de-arquivos/)
