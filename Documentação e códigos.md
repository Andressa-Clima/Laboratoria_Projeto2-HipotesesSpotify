# 📘 Documentação do Projeto — Etapas de Tratamento e Análise de Dados

## 🧩 Metas de Aprendizagem

### 🔵 Conectar/Importar Dados para as Ferramentas

Descompactei os documentos e incluí no Google Drive em Sheets, em planilhas separadas.
Importei as tabelas com ajuda desse vídeo:
Eliabe Silva — Como criar uma tabela no BigQuery vinculada ao Google Sheets…

A partir disso, fiz uma tabela para cada planilha e importei todas para o mesmo projeto.

### 🔵 Identificar e Tratar Valores Nulos

Na análise dos dados, foram identificados valores nulos em três tabelas:

- `technical_info`: coluna key (95 nulos);
- `competidores`: coluna in_shazam_charts (50 nulos);
- `track_in`: um único valor nulo (linha 575), corrigido após a conversão de um dado originalmente em formato string.

Esses registros não serão excluídos, mas serão desconsiderados nas análises, por enquanto.

### 🔵 Identificar e Tratar Valores Duplicados

Foram identificados registros duplicados em diferentes tabelas, cada um tratado de forma específica:

- `competitors`: duplicados possuem IDs distintos e foram mantidos;
- `technical_info`: uma linha duplicada com IDs diferentes foi preservada devido à necessidade desses IDs como chaves estrangeiras e à ausência de impacto significativo na análise;
- `spotify`: músicas de mesmo artista com valores distintos em streams foram filtradas, mantendo apenas a versão com maior reprodução por artista, resultando em uma nova tabela consolidada sem duplicatas.

### 🔵 Identificar e Gerenciar Dados Fora do Escopo de Análise

As colunas `key` e `mode` da tabela `technical_info` foram removidas da análise usando `EXCEPT`.
A coluna `key`, anteriormente já analisada com diversos valores nulos, não se apresentou tão importante, por isso foi desconsiderada.

### 🔵 Identificar e Tratar Dados Discrepantes em Variáveis Categóricas

Os caracteres especiais foram retirados com o comando:

```sql
SELECT REGEXP_REPLACE('Texto@com#caracteres$especiais!', r'[^a-zA-Z0-9]', '') AS resultado;
