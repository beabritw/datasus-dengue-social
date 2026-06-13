# Projeto Data Major — Tópicos de Banco de Dados · IESB 2026

> **Análise de padrões de hospitalização em casos de dengue com base no perfil clínico e social do paciente**
> + Dados via SINAN - DataSus
> **Docente:** Rodrigo Gonçalves

---

## Integrantes e Responsabilidades

- Beatriz Brito do Rosário - **Extract**
- Davi de Souza Lopes - **Transform**
- Lucas de Siqueira Cavalcanti - **Load**
- Lucas Montalvão Ramires - **Classificação**

---


## Dados
 
- **Fonte:** SINAN — DATASUS (FTP público)
- **Período:** 2022–2023
- **Volume bruto:** ~9,6 milhões de registros
- **Volume após limpeza:** ~6,9 milhões de registros (retenção de ~71,9%)
- **Tamanho da camada Gold:** ~23,34 MB (Parquet comprimido)
### Dimensões analisadas na clusterização
 
- **Sociais:** Raça/Cor, Escolaridade, Zona de Residência
- **Demográficas:** Sexo, Faixa Etária
- **Clínicas:** Sintomas (febre, cefaleia, exantema, mialgia etc.), Classificação clínica
- **Epidemiológicas:** Tipo de notificação, Hospitalização, Evolução do caso
> **Decisão de design:** registros com Raça/Cor e Escolaridade nulos foram mantidos como categoria `"Não Informado"`, pois a ausência de dados sociais não é aleatória — ela tende a caracterizar populações com menor acesso a serviços de saúde estruturados, sendo em si um indicador de vulnerabilidade.
 


---

## Estrutura de Pastas (Google Drive)

```
Topicos_BD/
├── raw/
│   ├── sinan/
│   │   └── sinan_dengue_raw/        ← parquets consolidados do SINAN
│   └── ibge/
│       └── ibge_municipios.parquet  ← tabela de municípios brasileiros
├── processed/                       ← saída do Transform
└── notebooks/                       ← notebooks .ipynb
```

> **IMPORTANTE — Acesso compartilhado:**  
> A pasta `Topicos_BD` está no Drive da Beatriz. Para executar qualquer etapa do pipeline sem precisar baixar os dados novamente, **solicite acesso de edição a ela e adicione a pasta como atalho no seu próprio Drive.** Cada notebook do pipeline (Transform, Load, Classificação) já aponta para esse mesmo caminho.

Cada etapa lê da pasta anterior e escreve na sua. 

---


Todas as etapas rodam no **Google Colab** com acesso ao Google Drive montado.

---

## 📎 Referências

- [Documentação PySUS / SINAN](https://pysus.readthedocs.io/en/latest/databases/SINAN.html)
- [API IBGE Localidades](https://servicodados.ibge.gov.br/api/docs/localidades)
- [Dicionário de dados SINAN Dengue — DATASUS](https://datasus.saude.gov.br/transferencia-de-arquivos/)
- [Arquitetura Medallion](https://www.databricks.com/blog/what-is-medallion-architecture)


Projeto desenvolvido para a disciplina de **Tópicos em Banco de Dados**.
 
---
 
*SINAN — Ministério da Saúde / DATASUS. Dados públicos disponíveis em: https://datasus.saude.gov.br/sinan*
 
