---
categories: python
layout: default
permalink: /python/carrinho_com_pendulo_livre/
tags: [2GDL, Conservativo]
title: Carrinho com pêndulo livre
---   

# Carrinho com Pêndulo Livre

Esse artigo mostra como fazer a simulação de um sistema massa-mola acoplado a um pêndulo, ambos em movimento livres.

As bibliotecas necessárias para essa simulação são:

```python
from sympy import *
import numpy as np
import matplotlib.pyplot as plt
from matplotlib import animation
from scipy.integrate import odeint
from matplotlib.animation import PillowWriter
```

As variáveis de parametros do sistema são:

| Variável | Descrição | Unidade |
| --- | --- | --- |
| \\(t\\) | Tempo | s |
|\\(g\\)| Gravidade | m/s² |
|\\(m_1\\)| Massa do carrinho | kg |
|\\(m_2\\)| Massa do pêndulo | kg |
|\\(l_1\\)| Posição de equilíbrio da mola| m |
|\\(l_2_\\)| Comprimento do pêndulo| m |
|\\(k_1_\\)| Constante elástica da mola | N/m |

As funções do sistema que são em relação ao tempo são:

| Função | Descrição | Unidade |
| --- | --- | --- |
| \\(x_1(t)\\) | Posição horizontal do carrinho | m |
| \\(x_2(t)\\) | Posição horizontal do pêndulo | m |
| \\(y_2_(t)\\) | Posição vertical do pêndulo| m |
| \\(\theta_2_(t)\\) | Ângulo do pêndulo com a horizontal| m |

Apenas as funções \\(x_1(t)\\) e \\(\theta_2_(t)\\) são suficiente para definir qualquer estado do sistema. Mas calcular as variáveis \\(x_2(t)\\) e \\(y_2(t)\\) vão ser úteis lá na frente na hora de fazer as animações.

A posição vertical do carrinho (\\(y_1(t)\\)) é sempre igual a \\(0\\).

Definindo os parâmetros e as funções no algorítimo (e as derivadas das funções):

```python
t, g, m1, m2, l1, l2, k1 = symbols('t g m_1 m_2 l_1 l_2 k_1')
x1, x2, y2, the2 = symbols(r'x_1  x_2 y_2 \theta_2', cls = Function)
the2 = the2(t)
the2_d = diff(the2, t)
the2_dd = diff(the2_d, t)
```

Definindo as outras funções do sistema:

```python
x1 = x1(t)
x2 = x1 + l2*cos(the2)
y2 = l2*sin(the2)
x1_d = diff(x1, t)
x2_d = diff(x2, t)
y2_d = diff(y2, t)
x1_dd = diff(x1_d, t)
x2_dd = diff(x2_d, t)
y2_dd = diff(y2_d, t)
```

Calculando as energias cinéticas e potenciais do sistema, lembrando que elas são definidas como:

Energia Cinética: \\( T = \frac{1}{2} m (\dot x^2 + \dot y^2) \\)

Energia Potencial: \\( V = mgy \\)

Lagrangiano: \\( \mathcal{L} = T- V \\)


```python
#energia cinética
T1 = 1/2 * m1 * x1_d**2
T2 = 1/2 * m2 * (x2_d**2 + y2_d**2)
T = T1 + T2

#energia potencial
V1 = 1/2 * k1 * (l1 - x1)**2
V2 = m2 * g * y2
V = V1 + V2

#lagrangiano
L = T - V
```

Equação de lagrange é dada por:

\\[
\frac{\mathrm{d}}{\mathrm{d}t} \Bigg(\frac{\partial \mathcal{L}}{\partial \dot q_i} \Bigg) - \frac{\partial \mathcal{L}}{\partial \dot q_i} = 0
\\]

Onde q_i é cada uma das variáveis do sistema, ou seja, \\(x_1(t)\\) e \\(\theta_2_(t)\\).

Calculando para cada uma das variáveis:

```python
EL1 = diff(L, x1) - diff(diff(L, x1_d), t)
EL2 = diff(L, the2) - diff(diff(L, the2_d), t)
```

Agora queremos saber a solução da equação acima para \\(\ddot x_1(t)\\) e \\(\ddot \theta_2_(t)\\). No algorítimo isso se traduz como:
 
```python
sols = solve([EL1, EL2], (x1_dd, the2_dd))
```

Para cada uma das funções definidas e suas derivadas, utilizmos a função 'lambdify' do sympy para converter uma expressão simbólica em uma numérica.

```python
x1_f = lambdify(x1, x1)
x2_f = lambdify((x1, l2, the2), x2)
y2_f = lambdify((x1, l2, the2), y2)

x1_d_f = lambdify(x1_d, x1_d)
the2_d_f = lambdify(the2_d, the2_d)
x1_dd_f = lambdify((t, g, m1, m2, l1, l2, k1, x1, x1_d, the2, the2_d), sols[x1_dd])
the2_dd_f = lambdify((t, g, m1, m2, l1, l2, k1, x1, x1_d, the2, the2_d), sols[the2_dd])
```

Definindo a função \\( \frac{\mathrm{d}S}{\mathrm{d}t} \\) em que, dada os parâmetros do sistema, retorna os valores numéricos de \\( \dot x_1 \\), \\( \dot \theta_2 \\), \\( \ddot x_1 \\), \\( \ddot \theta_2 \\), temos:

```python
def dSdt(S, t, g, m1, m2, l1, l2, k1):
  x1, the2, x1_d, the2_d = S

  return [
          x1_d_f(x1_d),
          the2_d_f(the2_d),
          x1_dd_f(t, g, m1, m2, l1, l2, k1, x1, x1_d, the2, the2_d),
          the2_dd_f(t, g, m1, m2, l1, l2, k1, x1, x1_d, the2, the2_d)
  ]
```

Definindo os parâmetros da simulação:

Período e número de amostragens:
- Tempo inicial: 0 segundos;
- Tempo final: 20 segundos;
- Número de quadros: 1000.

Parâmetros do sistema:
- Gravidade: 9.81 m/s²;
- Massa do carrinho: 1 kg;
- Massa do pêndulo: 1 kg;
- Posição de equilíbrio da mola: 1 m;
- Comprimento do pêndulo: 1 m;
- Constante elástica da mola: 100 N/m

Condição Inicial:
- Posição horizontal do bloco: 1 m;
- Ângulo do pêndulo (com a horizontal): pi/2 rad;
- Velocidade do carrinho: 0 m/s;
- Velocidade angular do pêndulo: 0.01 rad/s;

Esses parâmetros nos dará um sistema em que o carrinho começa em equilíbro (a sua posição horizontal é igual ao de equilíbrio da mola) e o pêndulo está na posição vertical, com uma leve pertubação para o sentido horário.

```python
tt = np.linspace(0, 20, 1000)
params = (9.81, 1, 1, 1, 1, 100) #g, m1, m2, l1, l2, k1
inic = (1, np.pi/2, 0, -0.01) #x1, the2, x1_d, the2_d
out = odeint(dSdt, inic, tt, args = params)
```

Dado todas as informações necessárias par ao sistema, podemos começar gerando um gráfico com os valores de \\(\ddot x_1(t)\\) e \\(\ddot \theta_2_(t)\\) em funçao do tempo:

```python
plt.plot(out.T[0])
plt.plot(out.T[1])
plt.show()
```
Resultado:

![Gráfico posição versus Tempo](https://i.imgur.com/T2tfhmA.png)

- Linha azul: \\(\ddot x_1(t)\\)
- Linha laranja: \\(\ddot \theta_2_(t)\\)

Também podemos fazer uma animação para ver o comportamento desse sistema em tempo real:

```python
def animate(i):
  l1, l2 = 1, 1
  x1 = out.T[0][i]
  y1 = 0
  the2 = out.T[1][i]
  x2 = x2_f(x1, l2, the2)
  y2 = y2_f(x1, l2, the2)
  lin.set_data([[0, x1, x2],
                [0, y1, y2]])

fig, ax = plt.subplots(1,1, figsize = (8, 8))
ax.set_ylim(-2, 3)
ax.set_xlim(-2, 3)
lin, = plt.plot([], [], 'bs:', lw = 3, ms = 8)
ani = animation.FuncAnimation(fig, animate, frames= 1000, interval= 50)
ani.save("carr_pend_.gif', writer='pillow',fps=20)
```

Resultado final:
![Animação Carrinho com Pêndulo Livre](https://i.imgur.com/N1IPjkk.png)