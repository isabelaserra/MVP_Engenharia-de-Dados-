# MVP_Engenharia-de-Dados-
MVP_Engenharia de Dados 


# tópicos

## 1. Contexto de Negócios e Perguntas (Etapa 2. e 4.1): 


Perguntas que quero responder

Quais empresas do setor bancário recebem mais reclamações?
Quais são os assuntos e problemas mais frequentes?
Qual é o tempo médio de resposta das empresas? Ele influencia a nota do consumidor?
Qual o percentual de reclamações resolvidas?
Como as reclamações se distribuem por região e faixa etária?

O Consumidor.gov.br é um site do governo onde o consumidor registra uma reclamação e a empresa tem até 10 dias para responder. Depois, o consumidor diz se o problema foi resolvido e dá uma nota de 1 a 5. O serviço é mantido pela Secretaria Nacional do Consumidor (Senacon), do Ministério da Justiça.

Os dados ficam disponíveis na área de Dados Abertos do site, com um arquivo por mês. Baixei os arquivos de junho, julho e agosto de 2026. Os de julho e agosto vieram em CSV, e o de junho veio compactado em .zip, o que me deu um pouco de trabalho na carga.

Os dados são públicos e fazem parte da Política de Dados Abertos do Governo Federal (Decreto nº 8.777/2016). Isso quer dizer que podem ser usados livremente, inclusive em trabalhos como este, desde que a fonte seja citada.

Cada arquivo é uma única tabela, em que cada linha é uma reclamação. As colunas são separadas por ponto e vírgula e são 19 no total.


## 2. Carga dos Dados (Etapa 4.2): 

Os arquivos de julho e agosto vieram em CSV, mas o de junho veio compactado em .zip. No começo, enviei os três arquivos do jeito que baixei, sem perceber essa diferença, e o .zip acabou sendo lido como se fosse um CSV. Para corrigir, descompactei o arquivo de junho no meu computador usando o WinZip e fiz o upload manual do CSV para o volume, pelo Catalog do Databricks. 

Li os três arquivos de uma vez com PySpark, informando que: 
a primeira linha traz o nome das colunas;
as colunas são separadas por ponto e vírgula;
o Spark deveria identificar sozinho o tipo de cada coluna (inferSchema);
só arquivos terminados em .csv deveriam ser lidos (pathGlobFilter), para evitar que o .zip fosse lido de novo.

Salvando a camada bronze

Com os dados lidos, salvei tudo na tabela bronze_reclamacoes, em formato Delta. Nessa etapa não mudei nada no conteúdo, nem mesmo os nomes das colunas, que continuaram com acentos e espaços. Para isso funcionar, precisei ativar uma configuração do Delta chamada mapeamento de colunas (delta.columnMapping.mode = 'name'). Deixei a padronização dos nomes para a camada silver. Esse passo foi sugestão da IA pois eu não sabia como fazer.

Acrescentei apenas duas colunas de controle:

arquivo_origem: de qual arquivo veio cada linha;
data_carga: quando a carga foi feita.

A coluna arquivo_origem foi muito útil, porque foi com ela que descobri que o .zip tinha sido lido por engano na primeira carga.

## 3. Modelagem e Catálogo de Dados (Etapa 4.3): 
Explicação da modelagem com a estrutura das tabelas (catálogo de dados transcrito e screenshots do sistema de catálogo). 
## 4. Pipeline de Dados (Etapa 4.4):
Explique como organizou o processo de pipeline ETL, se tudo foi feito em um único notebook ou se ramificou e como ramificou. Adicione referência aos scripts disponibilizados no Github e screenshots que evidencie que essas tabelas foram salvas (persistidas) na plataforma de nuvem utilizada.
## 5.Qualidade de Dados (Etapa 4.5): 
Quais problemas foram detectados e como resolveu cada um deles, que transformações foram feitas.
## 6. Análise de Dados (Etapa 4.5):
Análise feita e respondendo as perguntas elaboradas na etapa 4.1.
## 7. Autoavaliação: 
Ao finalizar o trabalho, é esperado que o aluno faça uma autoavaliação contendo uma discussão sobre se conseguiu atingir os objetivos delineados antes do início das outras etapas, suas dificuldades encontradas na execução do trabalho, bem como trabalhos futuros para enriquecer o problema e sua solução em seu portfólio.

<img width="805" height="661" alt="Captura de Tela 2026-09-24 às 22 19 19" src="https://github.com/user-attachments/assets/f30b9fb3-218e-47f8-8e8c-47ec9cc4229c" />

As descrições das tabelas e colunas foram geradas com o recurso de comentários por IA do Databricks e revisadas manualmente.
