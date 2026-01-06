=========================================================
KNN COM PRIVACIDADE DIFERENCIAL (MECANISMO EXPONENCIAL)
=========================================================

Este projeto implementa um classificador KNN que utiliza Privacidade 
Diferencial (DP) para proteger os dados de treinamento, focando no 
Mecanismo Exponencial e no controle rigoroso de orçamento.

---------------------------------------------------------
1. PERGUNTAS FREQUENTES (FAQ TÉCNICO)
---------------------------------------------------------

P1: O que é o Mecanismo Exponencial e por que usá-lo aqui?
R: É um mecanismo de DP usado para selecionar a melhor opção entre 
várias categorias (classes). Usamos ele em vez do mecanismo de Laplace 
porque, no KNN, o objetivo final é escolher uma classe (ex: "A" ou "B") 
e não apenas um número. Ele transforma a contagem de vizinhos em uma 
probabilidade, realizando um "sorteio viciado" onde a classe mais 
frequente tem maior chance de vencer, mas mantendo a privacidade.

P2: Como o orçamento de privacidade (Epsilon) é controlado?
R: O orçamento é tratado como um recurso finito. A classe possui um 
'epsilon_limit' (limite total) e um 'epsilon_spent' (gasto acumulado). 
Cada vez que uma predição é solicitada, verificamos se o custo daquela 
pergunta ultrapassa o que resta do limite. Se ultrapassar, o sistema 
bloqueia a resposta para evitar vazamento de dados.

P3: Como as perguntas/consultas foram adaptadas ao mecanismo?
R: Cada consulta agora é tratada como uma transação financeira. 
O usuário deve especificar quanto do seu orçamento total deseja gastar 
naquele lote de teste (epsilon_requested). Esse valor é dividido 
proporcionalmente entre todas as amostras do teste (composição básica), 
garantindo que o risco de privacidade seja quantificável e limitado.

P4: Qual é a sensibilidade (Δu) utilizada e por quê?
R: A sensibilidade é Δu = 1. Isso ocorre porque, no KNN, a adição ou 
remoção de um único indivíduo no conjunto de treino pode alterar a 
contagem de vizinhos de uma classe em, no máximo, 1 unidade.

P5: O que acontece quando o orçamento de privacidade acaba?
R: O código interrompe a execução e levanta um 'ValueError'. Isso é 
uma medida de segurança fundamental em Privacidade Diferencial para 
impedir ataques de consulta exaustiva, onde um atacante faria milhares 
de perguntas para tentar reconstruir os dados originais.

---------------------------------------------------------
2. EXPLICAÇÃO DO FLUXO MATEMÁTICO
---------------------------------------------------------
Sempre que uma predição é feita, o modelo:
1. Conta os vizinhos de cada classe (Utilidade).
2. Aplica a fórmula: Probabilidade ∝ exp( (ε * Utilidade) / (2 * Δu) ).
3. Realiza um sorteio aleatório baseado nessas probabilidades.
4. Deduz o ε gasto do saldo total da instância.

---------------------------------------------------------
3. COMO UTILIZAR
---------------------------------------------------------

```python
from knn_dp import KNN_DP

# Inicialização com limite de orçamento
knn = KNN_DP(r=0.5, epsilon_limit=1.0)
knn.fit(X_train, y_train)

# Consulta controlada
try:
    pred = knn.predict_private_exponential(X_test, epsilon_requested=0.2)
    print(f"Saldo de privacidade: {knn.get_remaining_budget()}")
except ValueError as e:
    print(f"Alerta de segurança: {e}")