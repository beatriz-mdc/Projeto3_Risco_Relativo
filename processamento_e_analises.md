# Etapas do projeto

***1. Processamento e preparação da base de dados***

Identificar e tratar valores nulos
tabela clientes_info
7199 last_month_salary
943 number_dependents
média 6668,57
mediana 5400
substituir os salarios nulos pela mediana e os dependentes por 0

consulta substituir nulos dependentes

CREATE OR REPLACE TABLE `projeto-3-459118.risco_relativo.clientes_info` AS
SELECT
user_id,
age,
sex,
last_month_salary,
COALESCE(number_dependents, 0) AS number_dependents,
FROM `projeto-3-459118.risco_relativo.clientes_info`;

consulta substituir nulos salario

CREATE OR REPLACE TABLE `projeto-3-459118.risco_relativo.clientes_info` AS
SELECT
user_id,
age,
sex,
COALESCE(last_month_salary, 5400) AS last_month_salary,
number_dependents,
FROM `projeto-3-459118.risco_relativo.clientes_info`;

a falta de registro de salário não está relacionada a inadimplencia

SELECT 
COUNT (*)
FROM `projeto-3-459118.risco_relativo.teste`
WHERE last_month_salary IS NULL AND default_flag = 0;

o resultado deu que a maioria 7069 não é inadimplente.

Identificar e tratar valores duplicados

Valores duplicados foram encontrados somente na tabela emprestimos, o que faz sentido, visto que uma pessoa pode possuir vários emprestimos

SELECT 
user_id,
COUNT (*) as duplicatas
FROM `projeto-3-459118.risco_relativo.emprestimos`
GROUP BY user_id
HAVING COUNT (*) >1

Identificar e gerenciar dados fora do escopo da análise

A seleção de variáveis foi realizada através de correlação, desvio padrão e conceitos legais.
Para a variável "gênero" foi decidido excluí-la, uma vez que não é permitido legalmente considerar esse recorte para fins financeiros. Abaixo query para a nova tabela:

CREATE OR REPLACE TABLE `projeto-3-459118.risco_relativo.clientes_info` AS
SELECT  
* EXCEPT (sex),
FROM `projeto-3-459118.risco_relativo.clientes_info`;

A partir de correlação relacionanos as variáveis referentes aos períodos de atraso no pagamento de um empréstimo. 
Foram relacionadas as variáveis "more_90_days_overdue", "number_times_delayed_payment_loan_60_89_days" e "number_times_delayed_payment_loan_30_59_days".
Abaixo a query utilizada e sua tabela:

SELECT  
CORR (more_90_days_overdue, number_times_delayed_payment_loan_60_89_days) as correlacao,
CORR (more_90_days_overdue, number_times_delayed_payment_loan_30_59_days) as correlacao,
CORR (number_times_delayed_payment_loan_60_89_days, number_times_delayed_payment_loan_30_59_days) as correlacao
FROM `projeto-3-459118.risco_relativo.detalhes_emprestimo`;

![image](https://github.com/user-attachments/assets/0474e233-3cbd-4c22-a220-e1065094f992)

Todos os resultados estão próximos de +1, o que indica uma forte correlação entre as variáveis. Entretanto, em casos assim, devemos considerar a variável a ser utilizada a partir de seu desvio padrão. Dessa forma, calculamos o desvio padrão das variáveis.
Abaixo a query utilizada e sua tabela:

SELECT 
STDDEV_SAMP(more_90_days_overdue) AS desvio_padrao,
STDDEV_SAMP(number_times_delayed_payment_loan_60_89_days) AS desvio_padrao,
STDDEV_SAMP(number_times_delayed_payment_loan_30_59_days) AS desvio_padrao,
FROM `projeto-3-459118.risco_relativo.detalhes_emprestimo`;

![image](https://github.com/user-attachments/assets/80e18b4c-ee40-4025-80a3-ee80985b551e)

O resultado mostra que a variável "number_times_delayed_payment_loan_30_59_days" é a que possui maior desvio padrão.
Entretanto, para fins de análise e posterior entendimento dos resultados, foi decidido optar pela variável "more_90_days_overdue", que também possui um alto desvio padrão.

Abaixo query para a nova tabela:

CREATE OR REPLACE TABLE `projeto-3-459118.risco_relativo.detalhes_emprestimo` AS
SELECT  
* EXCEPT (number_times_delayed_payment_loan_30_59_days, number_times_delayed_payment_loan_60_89_days),
FROM `projeto-3-459118.risco_relativo.detalhes_emprestimo`;

Identificar e tratar dados inconsistentes em variáveis ​​categóricas

As variaveis categoricas encontradas na tabela "emprestimos" foram padronizadas para letras minusculas e "other" virou "others".
Abaixo query:

CREATE OR REPLACE TABLE `projeto-3-459118.risco_relativo.emprestimos` AS
SELECT 
loan_id,
user_id,
CASE WHEN LOWER(loan_type) = 'other' THEN 'others'
ELSE LOWER(loan_type)
END AS loan_type_clean
FROM `projeto-3-459118.risco_relativo.emprestimos`;

Identificar e tratar dados discrepantes em variáveis ​​numéricas

Após analisar os outliers encontrados foi decidido não realizar nenhuma exclusão ou tratamento sobre eles, todos foram mantidos.

Criar novas variáveis

Foi criada uma nova tabela onde é contabilizado o total de empréstimos por tipo de empréstimo para cada cliente.
Abaixo a query:

CREATE OR REPLACE TABLE `projeto-3-459118.risco_relativo.total_emprestimos` AS
SELECT 
user_id,
loan_type_clean,
COUNT(loan_id) AS total_emprestimos
FROM `projeto-3-459118.risco_relativo.emprestimos`
GROUP BY user_id, loan_type_clean

Unir tabelas

As tabelas ... foram unidas com left join.
Abaixo a query com a nova tabela criada:

CREATE OR REPLACE TABLE `projeto-3-459118.risco_relativo.tabelas_1` AS
SELECT 
a.*,
b.default_flag,
c.more_90_days_overdue,
c.using_lines_not_secured_personal_assets,
c.debt_ratio
FROM `projeto-3-459118.risco_relativo.clientes_info` AS a
LEFT JOIN 
`projeto-3-459118.risco_relativo.classificacao` AS b
ON a.user_id = b.user_id
LEFT JOIN
`projeto-3-459118.risco_relativo.detalhes_emprestimo` AS c
ON b.user_id = c.user_id

Construir tabelas auxiliares

Com o comando WITH foi criada uma nova tabela onde inserimos uma coluna com o total de emprestimos registrados por cliente.
Abaixo a query:

CREATE OR REPLACE TABLE `projeto-3-459118.risco_relativo.tabelas_2` AS
WITH emprestimos_por_cliente AS (
  SELECT
    user_id,
    SUM(total_emprestimos) AS total_emprestimos
  FROM `projeto-3-459118.risco_relativo.total_emprestimos` AS a
  GROUP BY user_id
)

SELECT
  b.*,
  a.total_emprestimos
FROM `projeto-3-459118.risco_relativo.tabelas_1` AS b
LEFT JOIN emprestimos_por_cliente AS a
  ON a.user_id = b.user_id
  
Após a criação da nova tabela foram encontrados 425 valores nulos na coluna "total_emprestimos". Esses valores nulos indicam que o cliente não possui empréstimo ativo no banco. Dessa forma, esses clientes foram excluídos da base, uma vez que não fazem sentido para nossa análise.
Abaixo query:

DELETE FROM `projeto-3-459118.risco_relativo.tabelas_2`
WHERE total_emprestimos IS NULL

Fazer uma análise exploratória

Agrupar dados de acordo com variáveis ​​categóricas

As variáveis categóricas foram agrupadas conforme as tabelas abaixo:

![image](https://github.com/user-attachments/assets/393e0f2e-e05b-47fd-8fcd-d10394874234)

Ver variáveis ​​categóricas

As variáveis categóricas foram visualizadas em gráficos de barra conforme abaixo: 

![image](https://github.com/user-attachments/assets/de7246b9-48e2-437d-9ae8-16ae09c3c8f0)

Aplicar medidas de tendência central

Foram calculadas média e mediana para as variáveis numéricas e resumidas nas tabelas abaixo:

![image](https://github.com/user-attachments/assets/8e5bb4a8-d005-4cdc-ab06-73313813c5d0)

Ver distribuição














