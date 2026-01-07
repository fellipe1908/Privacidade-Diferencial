# k-Nearest Neighbors with Differential Privacy (KNN-DP)

Este repositório contém uma implementação robusta do algoritmo **k-Nearest Neighbors (k-NN)** integrada com garantias de **Privacidade Diferencial (DP)**. A implementação utiliza o **Mecanismo Exponencial** para seleção de classes e um sistema rigoroso de **Gerenciamento de Orçamento de Privacidade ($\epsilon$)**.

## 1. Fundamentação Teórica

### 1.1 Privacidade Diferencial
A Privacidade Diferencial fornece uma garantia matemática de que a saída de um algoritmo não revela se um registro individual específico estava presente no conjunto de dados de treinamento. Um algoritmo $M$ é $\epsilon$-diferencialmente privado se, para quaisquer dois bancos de dados adjacentes $D_1$ e $D_2$:

$$Pr[M(D_1) \in S] \leq e^\epsilon \cdot Pr[M(D_2) \in S]$$

### 1.2 Mecanismo Exponencial
Para problemas de classificação onde a saída é categórica, utilizamos o Mecanismo Exponencial. Ele seleciona uma resposta $s$ do conjunto de classes possíveis $S$ com probabilidade proporcional à sua utilidade:

$$Pr(s) \propto \exp\left( \frac{\epsilon \cdot u(D, s)}{2 \Delta u} \right)$$

Onde:
*   **$u(D, s)$ (Função de Utilidade):** Definida como a contagem de vizinhos da classe $s$ dentro do raio $r$.
*   **$\Delta u$ (Sensibilidade):** Para o KNN baseado em contagem, a sensibilidade global é $1$, pois a adição ou remoção de um único registro altera a contagem de vizinhos de qualquer classe em no máximo $1$.

## 2. Implementação e Controle de Orçamento

A principal contribuição desta versão é o **Controle Estrito de Orçamento de Privacidade**, corrigindo vulnerabilidades de consultas ilimitadas.

### 2.1 Gestão de Epsilon ($\epsilon$)
O orçamento de privacidade é tratado como um recurso exaurível:
*   **`epsilon_limit`**: Define o teto máximo de perda de privacidade permitido para a instância do modelo.
*   **`epsilon_spent`**: Rastreia o consumo acumulado através de **Composição Básica**.
*   **Exaustão de Orçamento**: O modelo monitora cada consulta. Se uma predição solicitada exceder o saldo restante, o sistema interrompe a operação e levanta uma exceção de segurança, impedindo ataques de reconstrução.

### 2.2 Composição de Consultas
Ao realizar predições em lote (batch), o orçamento solicitado para a operação é distribuído uniformemente entre as amostras, garantindo que o custo total do lote respeite a privacidade diferencial local e global.

## 3. Arquitetura do Código

O modelo foi estruturado na classe `KNN_DP` com os seguintes métodos principais:

*   `fit(X, y)`: Armazena os dados de treino em conformidade com as restrições de tipos numéricos.
*   `predict_private_exponential(X_test, epsilon_requested)`: Ponto de entrada para consultas privadas. Valida o orçamento e aplica a composição.
*   `get_remaining_budget()`: Retorna o saldo disponível para consultas futuras.

## 4. Exemplo de Uso Técnico

```python
import numpy as np
from modelo import KNN_DP

# Instanciação com Raio de Busca (r) e Limite de Privacidade (epsilon)
knn_dp = KNN_DP(r=1.2, epsilon_limit=1.0)
knn_dp.fit(X_train, y_train)

# Tentativa de predição com consumo de 0.5 do orçamento
try:
    epsilon_consulta = 0.5
    y_pred = knn_dp.predict_private_exponential(X_test, epsilon_requested=epsilon_consulta)
    print(f"Predição concluída. Orçamento restante: {knn_dp.get_remaining_budget():.4f}")
except ValueError as e:
    print(f"Violação de Privacidade: {e}")
```

## 5. Considerações de Segurança
1.  **Proteção contra Overfitting:** O ruído introduzido pelo mecanismo exponencial atua como um regularizador natural.
2.  **Invariância a Consultas Repetitivas:** O controle de `epsilon_spent` impede que um atacante reduza a variância do ruído através de múltiplas consultas sobre o mesmo ponto de dado.
3.  **Estabilização Numérica:** A implementação utiliza normalização de utilidade (subtração do valor máximo) antes da exponenciação para prevenir *overflow* numérico.
