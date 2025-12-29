# Privacidade-Diferencial

1. Qual função de utilidade utilizada?
  A função de utilidade u(x, c) utilizada foi a contagem de votos (número de vizinhos).
  Para uma instância de teste x e uma classe candidata c, a utilidade é o número de pontos do conjunto de treino classificados como c que estão dentro do raio r de x.
  No código: contagens[cls] += 1 dentro do método predict_instance_private_exponential.
2. Qual sensibilidade global dessa função de utilidade?
   A sensibilidade global é 1.
   A sensibilidade é definida pela mudança máxima na função de utilidade ao adicionar ou remover um registro do conjunto de dados. Como a utilidade é uma contagem simples de vizinhos, remover ou adicionar uma pessoa ao banco de dados pode alterar a contagem de votos de uma classe em, no máximo, 1 unidade.
3. Como aplicou o mecanismo Exponencial?
   O mecanismo foi aplicado para selecionar a classe de forma probabilística, onde classes com mais votos têm maior chance de serem escolhidas, mas existe uma chance não nula de selecionar classes incorretas para garantir privacidade.
   Os passos no código foram:
   Calcular a utilidade (contagem) para cada classe.
   Calcular o score ponderado
   Aplicar a exponencial nesses scores para obter probabilidades não normalizadas.
   Normalizar para que a soma seja 1.
   Sortear a classe usando essas probabilidades (np.random.choice).
4. Qual valor de orçamento foi utilizado a cada uso do mecanismo Exponencial e por quê?

  O orçamento utilizado em cada predição individual foi: epsilon dividido pela Quantidade de linhas em X teste
  Cada predição é uma consulta privada. Pela composição sequencial, o orçamento total é a soma dos orçamentos individuais. Assim, ao final de todas as predições, o orçamento total consumido é exatamente epsilon
ϵ

5.Qual a diferen ̧ca do uso do or ̧camento do trabalho 5 vs. trabalho 6 e porquˆe;
   No Trabalho 5, utilizando o mecanismo de Laplace, o orçamento de privacidade é dividido entre as classes, pois o ruído é adicionado separadamente a cada contagem do histograma de votos. Já no Trabalho 6, com o uso do mecanismo Exponencial, o orçamento de privacidade é dividido entre as predições, uma vez que cada classificação corresponde a uma única consulta privada. Essa diferença ocorre porque o mecanismo Exponencial seleciona a classe de forma probabilística em uma única etapa, enquanto o mecanismo de Laplace exige múltiplas aplicações de ruído, uma para cada classe.

6. Respondido no codigo
