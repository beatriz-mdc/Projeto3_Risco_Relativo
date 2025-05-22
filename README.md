# Projeto Risco Relativo

O projeto é parte do curso de Análise de Dados desenvolvido pela Laboratoria Brasil.

Toda a documentação realizada, bem como os materiais produzidos podem ser consultados nos arquivos anexados ao projeto.

# Sobre o projeto

Num contexto recente, a descida das taxas de juro no mercado desencadeou um aumento notável na demanda por pedidos de crédito. Os clientes viram este movimento do mercado, como uma oportunidade favorável para financiar grandes compras ou consolidar dívidas existentes, o que elevou o fluxo de pedidos de empréstimo no banco “Super Caja”. A equipe de análise de crédito do banco enfrenta um fardo esmagador devido à análise manual necessária para cada solicitação de empréstimo de clientes individuais. Esta metodologia manual resultou num processo ineficiente e atrasado, que afetou negativamente a eficiência e a velocidade com que os pedidos de empréstimo são processados. A situação tornou-se mais crítica devido à preocupação crescente com a taxa de inadimplência, um problema que afeta cada vez mais o setor financeiro, e aumenta a pressão sobre os bancos para identificar e mitigar riscos associados ao crédito.

Para enfrentar esse desafio, a proposta do banco é a automação do processo de análise de crédito usando técnicas de análise avançadas de dados, com o objetivo de melhorar a eficiência, precisão e rapidez na avaliação de pedidos de crédito. Além disso, o banco já tem uma métrica para identificar clientes com pagamento atrasado, o que poderia ser uma ferramenta valiosa para integrar na classificação de risco no novo sistema automatizado.

# Objetivo

O objetivo da análise é identificar o perfil de clientes com risco de inadimplência, montar uma pontuação de crédito através da análise de dados e avaliar o risco relativo, possibilitando assim, classificar os clientes e futuros clientes em diferentes categorias de risco com base na sua probabilidade de inadimplência. Esta classificação permitirá ao banco tomar decisões informadas sobre a quem conceder crédito, reduzindo assim o risco de empréstimos não reembolsáveis. Além disso, a integração destas métricas fortalecerá a capacidade do modelo de identificar riscos, contribuindo para a solidez financeira e a eficiência operacional do Banco.

# Ferramentas e Tecnologias

As ferramentas e tecnologias utilizadas foram:

- BigQuery;
- Google Colab - Linguagem Python;
- Looker Studio.

# Processamento e Análises

Toda a documentação relacionada a exploração dos dados pode ser encontrada no link abaixo:

- [Processamento e análises](https://github.com/beatriz-mdc/Projeto3_Risco_Relativo/blob/main/processamento_e_analises.md)

# Resultados

Melhor ponto de corte: 3.0

Matriz de confusão:

![image](https://github.com/user-attachments/assets/69d4f727-7158-445f-8a81-6904359b6107)

Dessa forma, foi validado o ponte de corte sendo score ≥ 3 = mau pagador. Com essa nota, a partir da análise da matriz de confusão, é possível acertar uma boa parte dos inadimplentes e também evitar a classificação errada de muitos bons pagadores. Esse ponto de corte é o melhor compromisso entre "perder poucos inadimplentes" e "não bloquear bons clientes".

# Recomendações

- Campanhas personalizadas com base na pontuação;
- Criar ofertas de crédito especiais para minimizar riscos entre os reprovados:
  - Crédito com valor reduzido.
  - Condições especiais de pagamento.
- Recuperar clientes classificados como “falsos positivos”:
  - Atualização dos dados cadastrados.
  - Informações sobre como melhorar o score.
- Testar novos pontos de corte, de acordo com a estratégia escolhida pelo banco;
- Treinar o modelo para que melhore sua precisão.
