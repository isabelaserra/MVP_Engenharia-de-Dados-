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

Depois de limpar os dados na camada silver, eu tinha uma tabela única e muito grande, em que as mesmas informações se repetiam várias vezes. Por exemplo, o nome de cada banco aparecia em milhares de linhas. Por isso, organizei a camada gold em um modelo estrela, que é bastante usado em análise de dados.

Nesse modelo, existe uma tabela central, chamada fato, onde cada linha é uma reclamação. Em volta dela ficam as tabelas de dimensão, que guardam as informações descritivas uma única vez, cada uma com um número de identificação (id). A tabela fato guarda só esses ids e as informações do atendimento.

Tabelas criadas

Tabela	O que guarda	Linhas
fato_reclamacao	Uma linha por reclamação, com os ids das dimensões, data, tempo de resposta, se foi respondida, avaliação e nota	514.326
dim_empresa	Empresas do setor bancário	323
dim_local	Região, estado e cidade do consumidor	6.243
dim_problema	Área, assunto, grupo do problema e problema	1.280
dim_consumidor	Sexo e faixa etária	27

Criei as tabelas em SQL, dentro de um notebook no Databricks. Para cada dimensão, selecionei as combinações únicas das colunas (com SELECT DISTINCT) e numerei cada uma com ROW_NUMBER(), gerando o id. Depois montei a tabela fato ligando a silver a cada dimensão com JOIN, para trazer os ids correspondentes.

Para conferir se nada tinha se perdido, comparei a quantidade de linhas: a fato ficou com 514.326 linhas, exatamente igual à silver. Também achei curioso que a dim_consumidor ficou com 27 linhas, que são as 3 opções de sexo (F, M e Não informado) combinadas com as 9 faixas etárias.

Também defini as chaves primárias das dimensões e as chaves estrangeiras da fato, o que permitiu visualizar o relacionamento entre as tabelas no Catalog do Databricks.


<img width="805" height="661" alt="Captura de Tela 2026-09-24 às 22 19 19" src="https://github.com/user-attachments/assets/f30b9fb3-218e-47f8-8e8c-47ec9cc4229c" />

Registrei a descrição de cada tabela e coluna no próprio Unity Catalog do Databricks. Usei o recurso de sugestão automática de descrições da ferramenta e revisei cada uma, ajustando as que não refletiam o significado real dos dados, como o caso do tempo de resposta, que fica vazio quando a empresa não respondeu.

<img width="1341" height="972" alt="image" src="https://github.com/user-attachments/assets/d325d74d-2705-4eee-8c39-884c278366d3" />

## 4. Pipeline de Dados (Etapa 4.4):
Dividi o pipeline em notebooks separados, um para cada etapa, seguindo a arquitetura em camadas (bronze, silver e gold). Fiz assim porque um único notebook ficaria muito grande e difícil de corrigir. Cada notebook lê a tabela salva pelo anterior.

Notebook	O que faz	Tabela gerada
[importação]	Lê os arquivos CSV e salva os dados brutos	bronze_reclamacoes
[silver]	Limpa e padroniza os dados	silver_reclamacoes
[gold]	Monta o modelo estrela	fato_reclamacao e 4 dimensões
[análise]	Responde às perguntas de negócio	—

Rodei os notebooks manualmente, nessa ordem, e conferi a quantidade de linhas em cada etapa. Todas as tabelas foram salvas em formato Delta no Databricks, no schema consumidor, como mostra o print abaixo.

## 5.Qualidade de Dados (Etapa 4.5): 
O primeiro problema que encontrei foi com o arquivo de junho. Ele veio compactado em .zip e, sem perceber, fiz a carga dele junto com os outros. Quando olhei os dados, apareceram linhas com caracteres ilegíveis. Contando as linhas pela coluna arquivo_origem, descobri que 74.594 delas vinham do .zip. Para resolver, descompactei o arquivo com o WinZip, enviei o CSV para o volume e refiz a carga.

Depois, padronizei os nomes das colunas, que vinham com acentos e espaços, como "Faixa Etária". Renomeei todas para letras minúsculas, sem acento e com "_" no lugar dos espaços, por exemplo faixa_etaria. Isso facilitou bastante na hora de escrever os códigos.

Também conferi os tipos de dados com o printSchema. Depois da correção do .zip, o Spark identificou corretamente a data de finalização como data e o tempo de resposta e a nota como números.

Ao olhar as primeiras linhas, percebi que algumas colunas de texto tinham espaços sobrando, como na região, que aparecia como "S " e "N ". Usei a função trim em todas as colunas de texto para remover esses espaços.

Em seguida, contei os valores vazios de cada coluna. Na coluna sexo, preenchi os vazios com "Não informado". Já no tempo de resposta e na nota do consumidor, mantive os vazios, porque eles têm um significado: a empresa não respondeu ou o consumidor não avaliou a reclamação.

Por último, verifiquei as linhas duplicadas e removi 1.044 delas, o que representa cerca de 0,2% da base.

Além desses tratamentos, filtrei apenas o segmento "Bancos, Financeiras e Administradoras de Cartão", que é o foco do trabalho. No final, a tabela silver_reclamacoes ficou com 514.326 reclamações.

## 6. Análise de Dados (Etapa 4.5):
Pergunta 1: Quais empresas do setor bancário recebem mais reclamações?

O Nubank foi a empresa com mais reclamações no período, com 67.575 (13,1% do total). Depois vieram o Banco do Brasil, com 44.332 (8,6%), o Santander, com 37.170 (7,2%), o Bradesco, com 37.078 (7,2%), e a Caixa, com 30.908 (6,0%). As 10 primeiras empresas do ranking somam 56,5% de todas as reclamações, ou seja, mais da metade.

O Itaú aparece duas vezes na lista, como "Banco Itaú Unibanco" e "Cartões Itaú", porque a base separa as empresas pelo nome fantasia. Juntando as duas, seriam 34.976 reclamações (6,8%).

Vale lembrar que bancos com mais clientes tendem a receber mais reclamações. Então esse ranking mostra onde estão as reclamações, mas não quer dizer que um banco atende pior que o outro.

Pergunta 2: Quais assuntos e problemas aparecem com mais frequência?

Quase todas as reclamações mais frequentes são sobre dois assuntos: cartão de crédito e crédito pessoal/empréstimos. A combinação que mais apareceu foi cartão de crédito com problema no cálculo de juros e saldo devedor, com 47.532 reclamações (9,2%).

Os mesmos problemas se repetem nos dois assuntos: cálculo de juros e saldo devedor, renegociação de dívida, negativação indevida de pagamento já feito e cobrança indevida para alterar ou cancelar contrato. Isso mostra que a maior parte dos conflitos está ligada a dívidas e cobranças.

Pergunta 3: Qual é o tempo médio de resposta? Ele influencia a nota do consumidor?

As empresas levaram, em média, 6,3 dias para responder, e cerca de 97% das reclamações foram respondidas. A nota média dos consumidores foi 2,1, numa escala de 1 a 5, o que é bem baixo.

Eu esperava que as respostas mais rápidas tivessem notas melhores, mas não foi o que aconteceu. Quem recebeu resposta em até 3 dias deu nota média 2,0, entre 4 e 7 dias a nota foi 2,25, e acima de 7 dias ficou em 2,15. Então o tempo de resposta quase não mudou a nota.

Pergunta 4: Qual o percentual de reclamações consideradas resolvidas?

A maioria dos consumidores (75,1%) não avaliou a reclamação. Considerando só as que foram avaliadas, 27,2% foram consideradas resolvidas e 72,8% não resolvidas.

Aqui encontrei a resposta para o que vi na pergunta anterior: quem teve o problema resolvido deu nota média 4,03, e quem não teve deu 1,38. Ou seja, o que mais pesa na satisfação é resolver o problema, e não a rapidez da resposta.

Um cuidado é que muitas pessoas não avaliaram, então esses percentuais podem não representar todos os casos.

Pergunta 5: Como as reclamações se distribuem por região e faixa etária?

O Sudeste concentra quase metade das reclamações (49,4%), seguido pelo Nordeste (19,8%), Sul (14,9%), Centro-Oeste (10,2%) e Norte (5,8%). Isso acompanha, em parte, o tamanho da população de cada região.

Por faixa etária, as pessoas entre 21 e 50 anos fazem 75,8% das reclamações, e a faixa de 31 a 40 anos é a que mais reclama (30,2%). Jovens até 20 anos (1,8%) e pessoas com mais de 70 anos (3,1%) aparecem bem menos. Acredito que isso tenha relação com o uso de cartão e empréstimos, que são mais comuns nessas idades intermediárias.

Conclusão da análise

As reclamações contra bancos estão concentradas em poucas empresas e giram principalmente em torno de juros, dívidas e cobranças. O resultado que mais me chamou atenção foi que responder rápido não deixa o consumidor mais satisfeito. O que faz diferença é resolver o problema. Para quem trabalha com prevenção de conflitos, isso mostra que o foco deve estar na solução, e não apenas no prazo.

## 7. Autoavaliação: 

Acredito que consegui atingir os objetivos que defini no início do trabalho. Consegui carregar os dados no Databricks, organizar o pipeline em camadas, montar o modelo estrela e responder às cinco perguntas de negócio. O resultado que mais gostei foi descobrir que, para o consumidor, resolver o problema importa muito mais do que responder rápido, algo que tem tudo a ver com a área de prevenção de conflitos em que trabalho.

A maior dificuldade foi o fato de eu vir da área de humanas e nunca ter feito um projeto desse tipo. Muitos termos e conceitos eram novos para mim, como schema, volume, camadas bronze, silver e gold, modelo estrela e chaves primárias e estrangeiras.

As aulas do curso me deram uma base, mas não foram suficientes para eu conseguir fazer o MVP sozinha. Precisei buscar materiais extras na Alura e no YouTube, além de usar algumas ferramentas de IA, como o ChatGPT, o Claude e o assistente do próprio Databricks, principalmente para entender os erros que apareciam e como corrigir os códigos.

Também tive dificuldades práticas durante o caminho. Na primeira leitura, os dados vieram todos em uma única coluna porque o separador não foi reconhecido. Depois, descobri que o arquivo de junho tinha vindo compactado e tinha sido lido como se fosse um CSV. Tive erros com os tipos de dados, com os nomes de colunas que tinham acentos e espaços e na montagem de alguns gráficos. Cada um desses problemas me ajudou a entender melhor como as coisas funcionam.

Alguns pontos limitam as conclusões do trabalho. A base não tem um número de identificação para cada reclamação, então não tenho certeza absoluta de que as linhas duplicadas que removi eram realmente repetidas. A maioria dos consumidores (75,1%) não avaliou a reclamação, o que pode afetar os percentuais de resolução. Algumas empresas do mesmo grupo aparecem separadas, como no caso do Itaú. E, por fim, analisei apenas três meses, o que não permite ver tendências ao longo do tempo.

Para melhorar o projeto, eu gostaria de:

- incluir mais meses ou anos, para acompanhar a evolução das reclamações;
- agrupar as empresas do mesmo conglomerado, como as duas marcas do Itaú;

