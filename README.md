# MVP_Engenharia-de-Dados-
MVP_Engenharia de Dados 


Independente do modelo, você deve criar um # Catálogo de Dados: um documento/sistema que descreve cada tabela e cada coluna do seu modelo. Isso existe porque, na prática, dados sem documentação viram "caixa preta", ninguém sabe o que significa cada campo, quais valores são válidos, de onde vieram. No Databricks já existe esse catálogo disponível para ser utilizado que é o Unity Catalog. 

O catálogo deve conter, para cada tabela existente e seus campos (colunas da tabela):

- Descrição do contexto e do que consiste os dados da tabela.

- Nome e descrição do que cada campo naquela tabela representa

- Tipo de dado (texto, número inteiro, decimal, data, booleano) de cada campo

- Domínio de valores de cada campo: para numéricos, os valores mínimos e máximos esperados; para categóricos, as categorias possíveis. Essa informação pode vim na descrição do campo.

- Linhagem dos dados: de qual fonte o dado veio e se houve alguma transformação ou junção para compô-lo. Essa informação pode vim na descrição do campo ou exibido de forma diagramável (Databricks auxilia nesse aspecto).

# tópicos

## 1. Contexto de Negócios e Perguntas (Etapa 2. e 4.1): 
perguntas de negócio que foram formuladas, explicação do contexto dos dados brutos e resumo da estrutura desses dados brutos (colunas e tabelas). Explique sobre a licença dos dados.
## 2. Carga dos Dados (Etapa 4.2): 
Explicação da carga de dados, como foi feita e referência ao script no GitHub (se aplicável).
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
