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

A seleção de variáveis foi realizada através de correlação, desvio padrão e de meios legais.
Para a variável "gênero" foi decidido excluí-la, uma vez que não é permitido legalmente considerar esse recorte para fins financeiros. Abaixo query:



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

Já a variável "gênero" foi
PERGUNTAR SOBRE SALARIO E DEPENDENTES





