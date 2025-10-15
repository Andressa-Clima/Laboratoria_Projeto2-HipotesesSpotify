📘 Documentação do Projeto — Etapas de Tratamento e Análise de Dados
🧩 Metas de Aprendizagem
🔵 Conectar/Importar Dados para as Ferramentas

Descompactei os documentos e incluí no Google Drive em Sheets, em planilhas separadas.
Importei as tabelas com ajuda desse vídeo:
Eliabe Silva — Como criar uma tabela no BigQuery vinculada ao Google Sheets…

A partir disso, fiz uma tabela para cada planilha e importei todas para o mesmo projeto.

🔵 Identificar e Tratar Valores Nulos

Na análise dos dados, foram identificados valores nulos em três tabelas:

technical_info: coluna key (95 nulos);

competidores: coluna in_shazam_charts (50 nulos);

track_in: um único valor nulo (linha 575), corrigido após a conversão de um dado originalmente em formato string.

Esses registros não serão excluídos, mas serão desconsiderados nas análises, por enquanto.

🔵 Identificar e Tratar Valores Duplicados

Foram identificados registros duplicados em diferentes tabelas, cada um tratado de forma específica:

competitors: duplicados possuem IDs distintos e foram mantidos;

technical_info: uma linha duplicada com IDs diferentes foi preservada devido à necessidade desses IDs como chaves estrangeiras e à ausência de impacto significativo na análise;

spotify: músicas de mesmo artista com valores distintos em streams foram filtradas, mantendo apenas a versão com maior reprodução por artista, resultando em uma nova tabela consolidada sem duplicatas.

🔵 Identificar e Gerenciar Dados Fora do Escopo de Análise

As colunas key e mode da tabela technical_info foram removidas da análise usando EXCEPT.
A coluna key, anteriormente já analisada com diversos valores nulos, não se apresentou tão importante, por isso foi desconsiderada.

🔵 Identificar e Tratar Dados Discrepantes em Variáveis Categóricas

Os caracteres especiais foram retirados com o comando:

SELECT REGEXP_REPLACE('Texto@com#caracteres$especiais!', r'[^a-zA-Z0-9]', '') AS resultado;


Foi feita uma nova tabela com as colunas desejadas, substituindo track_name por track_name_limpo.

🔵 Identificar e Tratar Dados Discrepantes em Variáveis Numéricas

Streams — track-in

MAX = 3.703.895.074

MIN = 2.762

AVG = 514.336.542.00105524…

🔵 Verificar e Alterar os Tipos de Dados

Foi detectada uma inconsistência na tabela track_in, especificamente na linha 575 da coluna 9 (streams).
Os valores registrados nessa célula correspondiam a dados da planilha technical_info ao invés do número total de streams.
A discrepância foi corrigida diretamente na base de dados, pois a coluna foi lida e exportada corretamente como integer.

🔵 Unir (JOIN) as Tabelas de Dados

Antes de começar, iniciei o vídeo sobre chaves primárias e estrangeiras (Programação Dinâmica — Diferença entre Chave Primária e Chave Estrangeira?)
para entender melhor o segundo vídeo sobre LEFT JOIN (Programação Dinâmica — Diferença entre INNER JOIN e OUTER JOIN | Tipos de JOINS em…).

Resolvi usar o comando LEFT JOIN para manter as informações da tabela principal track-inV2 e juntar apenas as posteriores (competidores e technical_info).
Além disso, decidi criar uma tabela à parte com REPLACE TABLE, pois se houvessem erros não impactaria track-inV2.

-- Cria uma nova tabela com os dados combinados
CREATE OR REPLACE TABLE hipoteses-projt2-andressalima.track_in_spotify.tracks_completos AS
SELECT
  track.*,  -- Todas as colunas da tabela track-inv2
  comp.* EXCEPT(track_id),  
  tech.* EXCEPT(track_id)   
FROM
  hipoteses-projt2-andressalima.track_in_spotify.track-inv2 AS track
LEFT JOIN
  hipoteses-projt2-andressalima.track_in_spotify.competitionV2 AS comp
ON track.track_id = comp.track_id
LEFT JOIN
  hipoteses-projt2-andressalima.track_in_spotify.technical_infov2 AS tech
ON track.track_id = tech.track_id;


As colunas foram organizadas também nessa ordem:

CREATE OR REPLACE TABLE hipoteses-projt2-andressalima.track_in_spotify.spotify_tabela_unificada AS
SELECT
  track_id,
  track_name,
  artist_s_name,
  artist_count,
  released_date,
  in_spotify_playlists,
  in_spotify_charts,
  streams,
  in_apple_playlists,
  in_apple_charts,
  in_deezer_playlists,
  in_deezer_charts,
  in_shazam_charts,
  total_playlists,
  bpm,
  danceability__,
  valence__,
  energy__,
  acousticness__,
  instrumentalness__,
  liveness__,
  speechiness__
FROM hipoteses-projt2-andressalima.track_in_spotify.spotify_tabela_unificada;

🔵 Criar Novas Variáveis

Em competitors, foi criada uma nova tabela com a variável de total de playlists (deezer + spotify):

CREATE OR REPLACE TABLE hipoteses-projt2-andressalima.track_in_spotify.competitionV2 AS 
SELECT *,
COALESCE(in_apple_playlists, 0) + COALESCE(in_deezer_playlists, 0) AS total_playlists
FROM hipoteses-projt2-andressalima.track_in_spotify.competition;


Para a data:

CREATE OR REPLACE TABLE hipoteses-projt2-andressalima.track_in_spotify.track-inV2 AS 
SELECT *,
SAFE.DATE(
  CAST(released_year AS INT64),
  CAST(released_month AS INT64),
  CAST(released_day AS INT64)
) AS released_date
FROM hipoteses-projt2-andressalima.track_in_spotify.track-inv2;

🔵 Construir Tabelas de Dados Auxiliares
-- Cria a tabela com a contagem de vezes do artista
CREATE OR REPLACE TABLE hipoteses-projt2-andressalima.track_in_spotify.spotify_tabela_com_contagem_artistas AS
-- Primeiro calculamos quantas vezes cada artista aparece
WITH contagem_artistas AS (
  SELECT
    artist_s_name,
    COUNT(*) AS total_musicas_artista
  FROM hipoteses-projt2-andressalima.track_in_spotify.spotify_tabela_unificada
  GROUP BY artist_s_name
)
-- Depois juntamos com a tabela original
SELECT
  u.*,  -- Todas as colunas originais
  c.total_musicas_artista  -- Nova coluna com a contagem
FROM
  hipoteses-projt2-andressalima.track_in_spotify.spotify_tabela_unificada u
LEFT JOIN
  contagem_artistas c
ON
  u.artist_s_name = c.artist_s_name; 


Leve sensação de que faria mais sentido ter essa fase antes de unir as tabelas, não sei.

🟣 Agrupar Dados de Acordo com Variáveis Categóricas

Após carregar os dados para o Power BI, incluímos a visualização de artistas e seus streams por ano.

🟣 Visualizar Variáveis Categóricas

Incluído gráfico de barras de streams por ano.

🟣 Aplicar Medidas de Tendência Central

Foi usada a média para calcular a média de streams por música + streams e música + artista.
A média total foi igual em ambas, por isso seguimos com apenas uma tabela.

🟣 Visualizar a Distribuição dos Dados

Através do Python, criamos histogramas para as variáveis da tabela technical_info.

As variáveis energy__ e bpm parecem se relacionar — músicas com batidas por minuto mais elevadas podem ser percebidas como mais energéticas.
A distribuição de danceability__ acompanha a de energy__, indicando que músicas com maior nível de energia costumam ser mais dançantes.

Com isso, é possível levantar a hipótese de correlações positivas entre bpm e energy, e entre energy e danceability.

🟣 Aplicar Medidas de Dispersão

Aplicamos medidas de dispersão nas variáveis para visualizar como se comportam.
Percebemos que streams possui desvio padrão mais elevado que as demais, em comparação à sua média.

🟣 Visualizar o Comportamento dos Dados ao Longo do Tempo

O número de streams e o de músicas lançadas por ano apresentam visualizações semelhantes, permitindo identificar os principais picos.
Observa-se que, quanto maior o número de músicas lançadas, maior também o volume de streams.
Além disso, a partir de 2020, houve um aumento no número de músicas registradas na base de dados.

🟣 Calcular Quartis, Decis ou Percentis
CREATE OR REPLACE TABLE hipoteses-projt2-andressalima.track_in_spotify.spotify_tabela_unificada AS
SELECT
  *,
  NTILE(4) OVER (ORDER BY streams) AS quartile_streams,
  IF(NTILE(4) OVER (ORDER BY streams) = 4, "alto", "baixo") AS categoria_streams,
  ...
FROM hipoteses-projt2-andressalima.track_in_spotify.spotify_tabela_unificada;


(mantém o código completo como no original)

🟣 Calcular Correlação Entre Variáveis

Interpretação de correlação:
Se o valor de “r” estiver próximo de 1, há forte correlação positiva.
Se estiver próximo de -1, forte correlação negativa.
Se estiver próximo de 0, ausência de correlação linear.

Correlação entre streams + total de playlists: 0.7830 → relação próxima.

Correlação entre streams + danceability: -0.1054

🔴 Aplicar Segmentação

Criamos tabelas para visualizar o comportamento de cada categoria e a média de streams.

🔴 Validar Hipótese

1. **BPM (batidas por minuto) não impacta significativamente o número de streams no Spotify. Não há associação linear relevante. Isso não se confirmou nesta base de dados.** 
2. **O sucesso no Spotify tende a se repetir em outras plataformas.** Rankings próximos sugerem que as pessoas buscam por artistas similares entre plataformas, refletindo uma popularidade multiplataforma.
3. **A presença em playlists está fortemente associada a um maior número de streams. A inclusão em playlists aumenta a visibilidade e facilita a descoberta da música, o que pode reforçar sua importância como estratégia.** 
4. **Artistas com mais músicas possuem mais streams. Lançar mais faixas podem garantir maior consumo.** 
5. **Algumas características influenciam o desempenho médio de streams.** Isso pode indicar uma **preferência por faixas mais melódicas, suaves ou introspectivas**.
