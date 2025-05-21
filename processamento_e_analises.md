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

As variáveis categóricas foram visualizadas em gráficos de barras conforme abaixo: 

![image](https://github.com/user-attachments/assets/de7246b9-48e2-437d-9ae8-16ae09c3c8f0)

Aplicar medidas de tendência central

Foram calculadas média e mediana para variáveis numéricas e resumidas nas tabelas abaixo:

![image](https://github.com/user-attachments/assets/216a9829-96b4-49ad-8196-3f2d9577af11)

Ver distribuição

As distribuições foram analisadas através de histogramas desenvolvidos em python no google colab:

[Histogramas](https://colab.research.google.com/drive/1Rt7ojChYG2ZsJia042dfdm3RNwqil2AE?usp=sharing) 

Aplicar medidas de dispersão (desvio padrão)

Foram calculados o desvio padrão das variáveis numéricas:

![image](https://github.com/user-attachments/assets/fa3dd5f6-fb8c-4948-a80f-61070f296a71)

Calcular quartis, decis ou percentis

Os quartis foram calculados a partir da seguinte query:

CREATE OR REPLACE TABLE `projeto-3-459118.risco_relativo.tabelas_2` AS
WITH quartis AS (
  SELECT
    user_id,
    NTILE(4) OVER (ORDER BY age) AS idade_quartil,
    NTILE(4) OVER (ORDER BY last_month_salary) AS salario_quartil,
    NTILE(4) OVER (ORDER BY more_90_days_overdue) AS atraso_quartil,
    NTILE(4) OVER (ORDER BY using_lines_not_secured_personal_assets) AS limite_quartil,
    NTILE(4) OVER (ORDER BY debt_ratio) AS endividamento_quartil,
    NTILE(4) OVER (ORDER BY total_emprestimos) AS emprestimos_quartil,
  FROM
    `projeto-3-459118.risco_relativo.tabelas_2`
)
SELECT
  a.* EXCEPT(number_dependents),
  b.idade_quartil,
  b.salario_quartil,
  b.atraso_quartil,
  b.limite_quartil,
  b.endividamento_quartil,
  b.emprestimos_quartil
FROM
  `projeto-3-459118.risco_relativo.tabelas_2` a
JOIN
  quartis b ON a.user_id = b.user_id

Calcular correlação entre variáveis ​​numéricas

Foram feitas as seguintes correlações:

SELECT  
CORR (last_month_salary, total_emprestimos) as correlacao,
CORR (age, total_emprestimos) as correlacao,
CORR (last_month_salary, using_lines_not_secured_personal_assets) as correlacao,
CORR (more_90_days_overdue, total_emprestimos) as correlacao,
FROM `projeto-3-459118.risco_relativo.tabelas_2`

Aplicar técnica de análise

Calcular risco relativo

O risco relativo foi calculado para os quartis de cada variável.
Abaixo as queries utilizadas e os principais resultados encontrados:

Idade
Q1 = 1.4010416666666667 - MAU PAGADOR
WITH resumo AS (
  SELECT
    idade_quartil,
    COUNT(*) AS total,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS eventos
  FROM `projeto-3-459118.risco_relativo.tabelas_2`
  GROUP BY idade_quartil
),
calculado AS (
  SELECT
    MAX(CASE WHEN idade_quartil = 1 THEN eventos / total END) AS risco_expostos,
    MAX(CASE WHEN idade_quartil > 1 THEN eventos / total END) AS risco_nao_expostos
  FROM resumo
)
SELECT
  risco_expostos,
  risco_nao_expostos,
  SAFE_DIVIDE(risco_expostos, risco_nao_expostos) AS risco_relativo
FROM calculado;

salario
Q1 = 1.5123456790123457 - MAU PAGADOR
WITH resumo AS (
  SELECT
    salario_quartil,
    COUNT(*) AS total,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS eventos
  FROM `projeto-3-459118.risco_relativo.tabelas_2`
  GROUP BY salario_quartil
),
calculado AS (
  SELECT
    MAX(CASE WHEN salario_quartil = 1 THEN eventos / total END) AS risco_expostos,
    MAX(CASE WHEN salario_quartil > 1 THEN eventos / total END) AS risco_nao_expostos
  FROM resumo
)
SELECT
  risco_expostos,
  risco_nao_expostos,
  SAFE_DIVIDE(risco_expostos, risco_nao_expostos) AS risco_relativo
FROM calculado;

endividamento
Q3 = 1.260727798906933 - MAU PAGADOR
WITH resumo AS (
  SELECT
    endividamento_quartil,
    COUNT(*) AS total,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS eventos
  FROM `projeto-3-459118.risco_relativo.tabelas_2`
  GROUP BY endividamento_quartil
),
calculado AS (
  SELECT
    MAX(CASE WHEN endividamento_quartil = 3 THEN eventos / total END) AS risco_expostos,
    MAX(CASE WHEN endividamento_quartil IN (1,2,4) THEN eventos / total END) AS risco_nao_expostos
  FROM resumo
)
SELECT
  risco_expostos,
  risco_nao_expostos,
  SAFE_DIVIDE(risco_expostos, risco_nao_expostos) AS risco_relativo
FROM calculado;

atraso
Q4 = 28.553210390194533 > MAU PAGADOR

WITH resumo AS (
  SELECT
    atraso_quartil,
    COUNT(*) AS total,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS eventos
  FROM `projeto-3-459118.risco_relativo.tabelas_2`
  GROUP BY atraso_quartil
),
calculado AS (
  SELECT
    MAX(CASE WHEN atraso_quartil = 4 THEN eventos / total END) AS risco_expostos,
    MAX(CASE WHEN atraso_quartil IN (1,2,3) THEN eventos / total END) AS risco_nao_expostos
  FROM resumo
)
SELECT
  risco_expostos,
  risco_nao_expostos,
  SAFE_DIVIDE(risco_expostos, risco_nao_expostos) AS risco_relativo
FROM calculado;

limite
Q4 = 20.140195504406798 > MAU PAGADOR

WITH resumo AS (
  SELECT
    limite_quartil,
    COUNT(*) AS total,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS eventos
  FROM `projeto-3-459118.risco_relativo.tabelas_2`
  GROUP BY limite_quartil
),
calculado AS (
  SELECT
    MAX(CASE WHEN limite_quartil = 4 THEN eventos / total END) AS risco_expostos,
    MAX(CASE WHEN limite_quartil IN (1,2,3) THEN eventos / total END) AS risco_nao_expostos
  FROM resumo
)
SELECT
  risco_expostos,
  risco_nao_expostos,
  SAFE_DIVIDE(risco_expostos, risco_nao_expostos) AS risco_relativo
FROM calculado;

emprestimos
Q1 = 1.6139240506329116 > MAU PAGADOR

WITH resumo AS (
  SELECT
    emprestimos_quartil,
    COUNT(*) AS total,
    SUM(CASE WHEN default_flag = 1 THEN 1 ELSE 0 END) AS eventos
  FROM `projeto-3-459118.risco_relativo.tabelas_2`
  GROUP BY emprestimos_quartil
),
calculado AS (
  SELECT
    MAX(CASE WHEN emprestimos_quartil = 1 THEN eventos / total END) AS risco_expostos,
    MAX(CASE WHEN emprestimos_quartil > 1 THEN eventos / total END) AS risco_nao_expostos
  FROM resumo
)
SELECT
  risco_expostos,
  risco_nao_expostos,
  SAFE_DIVIDE(risco_expostos, risco_nao_expostos) AS risco_relativo
FROM calculado;

Aplicar segmentação por Score

O score dos clientes foi construído com base nos quartis encontrados no tópico anterior, onde possuiam um alto valor de risco relativo, ou seja, maior propensão a ser um mau pagador. Dessa forma, foram criados dummies, variáveis binárias, onde o quartil encontrado equivalia a 1 e os demais igual a 0. Foram criados 6 dummies, um para cada tipo de quartil calculado, e, posteriormente, os valores foram somados, assim gerando a nota de cada cliente.
Abaixo a query utilizada:

CREATE OR REPLACE TABLE `projeto-3-459118.risco_relativo.tabelas_2` AS
WITH dummies AS (
SELECT
user_id,
IF (idade_quartil = 1, 1, 0) as dummy_idade,
IF (salario_quartil = 1, 1, 0) as dummy_salario,
IF (atraso_quartil = 4, 1, 0) as dummy_atraso,
IF (limite_quartil = 4, 1, 0) as dummy_limite,
IF (endividamento_quartil = 3, 1, 0) as dummy_endividamento,
IF (emprestimos_quartil = 1, 1, 0) as dummy_emprestimo
FROM `projeto-3-459118.risco_relativo.tabelas_2`

)

SELECT
a.*,
b.dummy_atraso + b.dummy_emprestimo + b.dummy_endividamento + b.dummy_idade + b.dummy_limite + b.dummy_salario as score
FROM `projeto-3-459118.risco_relativo.tabelas_2` a

JOIN

dummies b on a.user_id = b.user_id

O score calculado corresponde ao intervalo de 1 a 6. Para definir o ponto de corte dessa escala, ou seja, a partir de qual nota o cliente pode ser considerado um mau pagador, foi utilizado python no google colab. Abaixo o link com oprompt utilizado com base na tabela desenvolvida em SQL.

import numpy as np
import pandas as pd
from sklearn.metrics import confusion_matrix, roc_curve

df = pd.read_csv('score.csv')  # substitua pelo nome do seu arquivo
print(df.head())

# Exemplo de dados: df com colunas 'score' (1 a 6) e 'default_flag' (0 ou 1)
# df = pd.DataFrame({'score': [...], 'default_flag': [...]})

# Vamos testar todos os pontos de corte possíveis entre 1 e 6
cutoffs = np.arange(1, 6)
results = []

for cutoff in cutoffs:
    # Classificamos como mau pagador se score >= cutoff
    df['pred'] = (df['score'] >= cutoff).astype(int)

    tn, fp, fn, tp = confusion_matrix(df['default_flag'], df['pred']).ravel()

    sens = tp / (tp + fn)  # Sensibilidade (Recall para inadimplentes)
    spec = tn / (tn + fp)  # Especificidade

    youden_j = sens + spec - 1

    results.append({'cutoff': cutoff, 'sensibilidade': sens, 'especificidade': spec, 'youden_j': youden_j})

results_df = pd.DataFrame(results)

# Escolha o ponto de corte que maximiza Youden's J
best_cutoff = results_df.loc[results_df['youden_j'].idxmax()]
print(f"Melhor ponto de corte: {best_cutoff['cutoff']} com Youden's J = {best_cutoff['youden_j']:.3f}")

O resultado encontrado foi: Melhor ponto de corte: 3.0 com Youden's J = 0.584

Para validar o ponto de corte encontrado foi feita a matriz de confusão. Abaixo seu prompt e resultado:

import pandas as pd
from sklearn.metrics import confusion_matrix, ConfusionMatrixDisplay
import matplotlib.pyplot as plt

cutoff = 3

# Criar a previsão binária usando o ponto de corte
df['pred'] = (df['score'] >= cutoff).astype(int)

# Calcular a matriz de confusão
cm = confusion_matrix(df['default_flag'], df['pred'])

# Mostrar a matriz no console
print("Matriz de Confusão:")
print(cm)

# Opcional: mostrar a matriz de confusão com labels e visualização gráfica
disp = ConfusionMatrixDisplay(confusion_matrix=cm,
                              display_labels=['Bom pagador (0)', 'Inadimplente (1)'])
disp.plot(cmap=plt.cm.Blues)
plt.show()

Matriz de Confusão:
[[28280  6673]
 [  140   482]]

 ![image](https://github.com/user-attachments/assets/48a27d72-5c5e-4917-bcc0-354e9888a71d)

Dessa forma, foi validado o ponte de corte sendo score ≥ 3 = mau pagador. 
Com essa nota, a partir da análise da matriz de confusão, é possível acertar uma boa parte dos inadimplentes e também evitar a classificação errada de muitos bons pagadores. Esse ponto de corte é o melhor compromisso entre "perder poucos inadimplentes" e "não bloquear bons clientes".





  









