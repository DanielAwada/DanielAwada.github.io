---
layout: default
permalink: /lagrange/pend_inv/
tags: [2GDL, Conservativo]
title: Pêndulo Inverso
---   

# Pêndulo Inverso
## CINEMÁTICA

| Variável | Descrição | Unidade |
| --- | --- | --- |
| \\(M\\) | Massa do bloco| kg |
|\\(m\\)| Massa do pêndulo| kg |
|\\(L\\)| Comprimento do pêndulo| m |
|\\(X\\)| Posição horizontal do bloco| m |
|\\(\dot X\\)| Velocidade horizontal do bloco| m/s |
|\\(\theta\\)| Ângulo do pêndulo| rad |
|\\(\dot \theta\\)| Velocidade angular do pêndulo| rad/s |

```
\[
\begin{cases}
X_m = X - L sin(\theta) \\ 
Y_m =     L cos(\theta) 
\end{cases}
\]
```

Calculando a derivada em função do tempo:

\\[
\begin{cases}
\dot X_m = \dot X - L \dot \theta cos(\theta) \\\ 
\dot Y_m = - L \dot \theta sin(\theta)
\end{cases}
\\]

### Energia Potencial (V):
\\[
V = m g Y_m
\\]

Substituindo a segunda equação do sistema (1) na equação acima, temos que:

\\[
V = m g L cos(\theta)
\\]

### Energia Cinética (T):
\\[
T = \frac{1}{2} M \dot X^(2) + \frac{1}{2} m (\dot X_m^(2) + \dot Y_m^(2))
\\]

Substituindo o sistema de equações (2) na equação acima, temos que:

\\[
T = \frac{1}{2} M \dot X^(2) + \frac{1}{2} m \Big( \dot X^(2) - 2 L \dot \theta \dot X cos(\theta) + L^(2) \dot \theta^(2) \big(cos(\theta)^(2) + sin(\theta)^2 \big) \Big) 
\\]

Lembrando que \\( cos(\theta)^2 + sin(\theta)^2  = 1\\), a equação acima pode ser simplificada como:

\\[
T = \frac{1}{2} (M + m) \dot X^(2) + \frac{1}{2} m L^(2) \dot \theta^2 - m L \dot \theta \dot X cos(\theta)
\\]


## EQUAÇÃO DE LAGRANGE
\\[
\frac{\mathrm{d}}{\mathrm{d}t} \Bigg(\frac{\partial \mathcal{L}}{\partial \dot q_i} \Bigg) - \frac{\partial \mathcal{L}}{\partial \dot q_i} = Q_i
\\]

\\[
L = T - V
\\]

\\[
L = \Bigg[ \frac{1}{2} (M + m) \dot X^(2) + \frac{1}{2} m L^(2) \dot \theta^2 - m L \dot \theta \dot X cos(\theta) \Bigg] - \Bigg[ m g L cos(\theta) \Bigg]
\\]

## EQUAÇÕES DE MOVIMENTO
### Na dimensão X
Resolvendo o Lagrangiano em \\( \dot X \\) nos dará a equação de movimento nessa direção:

\\[
(M + m) \dot X - (mL\ddot \theta cos(\theta) - mL\dot \theta^2 sin(\theta)) = F(t)
\\]

Fazendo a multiplicação do sinal com o parênteses para simplificação:

\\[
(M + m) \dot X - mL\ddot \theta cos(\theta) + mL\dot \theta^2 sin(\theta) = F(t)
\\]

### Na dimensão θ
Resolvendo o Lagrangiano em \( \dot \theta \) nos dará a equação de movimento nessa direção:

\\[
m L^(2) \ddot \theta^(2) - m L \ddot X cos(\theta) + m L \dot X \dot \theta sin(\theta) - m L \dot X \dot \theta sin(\theta) - m g L sin(\theta) = 0
\\]

A equação acima é igual a 0 pois não há nenhuma força de Momento externa no nosso exemplo, caso houvesse, iríamos igualar à ela.

Cancelando as partes \\(+ m L \dot X \dot \theta sin(\theta)\\) e \\(- m L \dot X \dot \theta sin(\theta)\\) temos:

\\[
m L^(2) \ddot \theta^2 - m L \ddot X cos(\theta) - m g L sin(\theta) = 0
\\]

Dividindo tudo por \\(mL\\):
\[
L \ddot \theta^2 - \ddot X cos(\theta) - g sin(\theta) = 0
\]

Temos então, como resultado final, as equações (n) e (m):

\\[
\begin{cases}
(M + m) \dot X - mL\ddot \theta cos(\theta) + mL\dot \theta^(2) sin(\theta) = F(t) \\\ 
L \ddot \theta^2 - \ddot X cos(\theta) - g sin(\theta) = 0
\end{cases}
\\]
