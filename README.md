# MVP_Engenharia-de-Dados-
MVP_Engenharia de Dados 


Independente do modelo, você deve criar um #Catálogo de Dados: um documento/sistema que descreve cada tabela e cada coluna do seu modelo. Isso existe porque, na prática, dados sem documentação viram "caixa preta", ninguém sabe o que significa cada campo, quais valores são válidos, de onde vieram. No Databricks já existe esse catálogo disponível para ser utilizado que é o Unity Catalog. 

O catálogo deve conter, para cada tabela existente e seus campos (colunas da tabela):

- Descrição do contexto e do que consiste os dados da tabela.

- Nome e descrição do que cada campo naquela tabela representa

- Tipo de dado (texto, número inteiro, decimal, data, booleano) de cada campo

- Domínio de valores de cada campo: para numéricos, os valores mínimos e máximos esperados; para categóricos, as categorias possíveis. Essa informação pode vim na descrição do campo.

- Linhagem dos dados: de qual fonte o dado veio e se houve alguma transformação ou junção para compô-lo. Essa informação pode vim na descrição do campo ou exibido de forma diagramável (Databricks auxilia nesse aspecto).

