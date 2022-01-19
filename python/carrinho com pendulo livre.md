---
categories: python
layout: default
permalink: /python/carrinho_com_pendulo_livre/
tags: [2GDL, Conservativo]
title: Carrinho com p�ndulo livre
---   

# Carrinho com P�ndulo Livre

Esse artigo mostra como fazer a simula��o de um sistema massa-mola acoplado a um p�ndulo, ambos em movimento livres.

As bibliotecas necess�rias para essa simula��o s�o:

```python
from sympy import *
import numpy as np
import matplotlib.pyplot as plt
from matplotlib import animation
from scipy.integrate import odeint
from matplotlib.animation import PillowWriter
```

As vari�veis de parametros do sistema s�o:

| Vari�vel | Descri��o | Unidade |
| --- | --- | --- |
| \\(t\\) | Tempo | s |
|\\(g\\)| Gravidade | m/s� |
|\\(m_1\\)| Massa do carrinho | kg |
|\\(m_2\\)| Massa do p�ndulo | kg |
|\\(l_1\\)| Posi��o de equil�brio da mola| m |
|\\(l_2_\\)| Comprimento do p�ndulo| m |
|\\(k_1_\\)| Constante el�stica da mola | N/m |

As fun��es do sistema que s�o em rela��o ao tempo s�o:

| Fun��o | Descri��o | Unidade |
| --- | --- | --- |
| \\(x_1(t)\\) | Posi��o horizontal do carrinho | m |
| \\(x_2(t)\\) | Posi��o horizontal do p�ndulo | m |
| \\(y_2_(t)\\) | Posi��o vertical do p�ndulo| m |
| \\(\theta_2_(t)\\) | �ngulo do p�ndulo com a horizontal| m |

Apenas as fun��es \\(x_1(t)\\) e \\(\theta_2_(t)\\) s�o suficiente para definir qualquer estado do sistema. Mas calcular as vari�veis \\(x_2(t)\\) e \\(y_2(t)\\) v�o ser �teis l� na frente na hora de fazer as anima��es.

A posi��o vertical do carrinho (\\(y_1(t)\\)) � sempre igual a \\(0\\).

Definindo os par�metros e as fun��es no algor�timo (e as derivadas das fun��es):

```python
t, g, m1, m2, l1, l2, k1 = symbols('t g m_1 m_2 l_1 l_2 k_1')
x1, x2, y2, the2 = symbols(r'x_1  x_2 y_2 \theta_2', cls = Function)
the2 = the2(t)
the2_d = diff(the2, t)
the2_dd = diff(the2_d, t)
```

Definindo as outras fun��es do sistema:

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

Calculando as energias cin�ticas e potenciais do sistema, lembrando que elas s�o definidas como:

Energia Cin�tica: \\( T = \frac{1}{2} m (\dot x^2 + \dot y^2) \\)

Energia Potencial: \\( V = mgy \\)

Lagrangiano: \\( \mathcal{L} = T- V \\)


```python
#energia cin�tica
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

Equa��o de lagrange � dada por:

\\[
\frac{\mathrm{d}}{\mathrm{d}t} \Bigg(\frac{\partial \mathcal{L}}{\partial \dot q_i} \Bigg) - \frac{\partial \mathcal{L}}{\partial \dot q_i} = 0
\\]

Onde q_i � cada uma das vari�veis do sistema, ou seja, \\(x_1(t)\\) e \\(\theta_2_(t)\\).

Calculando para cada uma das vari�veis:

```python
EL1 = diff(L, x1) - diff(diff(L, x1_d), t)
EL2 = diff(L, the2) - diff(diff(L, the2_d), t)
```

Agora queremos saber a solu��o da equa��o acima para \\(\ddot x_1(t)\\) e \\(\ddot \theta_2_(t)\\). No algor�timo isso se traduz como:
 
```python
sols = solve([EL1, EL2], (x1_dd, the2_dd))
```

Para cada uma das fun��es definidas e suas derivadas, utilizmos a fun��o 'lambdify' do sympy para converter uma express�o simb�lica em uma num�rica.

```python
x1_f = lambdify(x1, x1)
x2_f = lambdify((x1, l2, the2), x2)
y2_f = lambdify((x1, l2, the2), y2)

x1_d_f = lambdify(x1_d, x1_d)
the2_d_f = lambdify(the2_d, the2_d)
x1_dd_f = lambdify((t, g, m1, m2, l1, l2, k1, x1, x1_d, the2, the2_d), sols[x1_dd])
the2_dd_f = lambdify((t, g, m1, m2, l1, l2, k1, x1, x1_d, the2, the2_d), sols[the2_dd])
```

Definindo a fun��o \\( \frac{\mathrm{d}S}{\mathrm{d}t} \\) em que, dada os par�metros do sistema, retorna os valores num�ricos de \\( \dot x_1 \\), \\( \dot \theta_2 \\), \\( \ddot x_1 \\), \\( \ddot \theta_2 \\), temos:

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

Definindo os par�metros da simula��o:

Per�odo e n�mero de amostragens:
- Tempo inicial: 0 segundos;
- Tempo final: 20 segundos;
- N�mero de quadros: 1000.

Par�metros do sistema:
- Gravidade: 9.81 m/s�;
- Massa do carrinho: 1 kg;
- Massa do p�ndulo: 1 kg;
- Posi��o de equil�brio da mola: 1 m;
- Comprimento do p�ndulo: 1 m;
- Constante el�stica da mola: 100 N/m

Condi��o Inicial:
- Posi��o horizontal do bloco: 1 m;
- �ngulo do p�ndulo (com a horizontal): pi/2 rad;
- Velocidade do carrinho: 0 m/s;
- Velocidade angular do p�ndulo: 0.01 rad/s;

Esses par�metros nos dar� um sistema em que o carrinho come�a em equil�bro (a sua posi��o horizontal � igual ao de equil�brio da mola) e o p�ndulo est� na posi��o vertical, com uma leve pertuba��o para o sentido hor�rio.

```python
tt = np.linspace(0, 20, 1000)
params = (9.81, 1, 1, 1, 1, 100) #g, m1, m2, l1, l2, k1
inic = (1, np.pi/2, 0, -0.01) #x1, the2, x1_d, the2_d
out = odeint(dSdt, inic, tt, args = params)
```

Dado todas as informa��es necess�rias par ao sistema, podemos come�ar gerando um gr�fico com os valores de \\(\ddot x_1(t)\\) e \\(\ddot \theta_2_(t)\\) em fun�ao do tempo:

```python
plt.plot(out.T[0])
plt.plot(out.T[1])
plt.show()
```
Resultado:

![Gr�fico posi��o versus Tempo](https://i.imgur.com/T2tfhmA.png)

- Linha azul: \\(\ddot x_1(t)\\)
- Linha laranja: \\(\ddot \theta_2_(t)\\)

Tamb�m podemos fazer uma anima��o para ver o comportamento desse sistema em tempo real:

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
![Anima��o Carrinho com P�ndulo Livre](https://i.imgur.com/N1IPjkk.png)