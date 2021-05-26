---
title: Derivation of Compton Wavelength
date: 2021-05-26 00:43:25
categories: 科學
tags:
- photon
- conservation of energy
- conservation of momentum
- Compton wavelength
mathjax: true
---

Here is one page from my general physics notes. The derivation is still beautiful to me even though so many years passed. I want to preserve these and sharpen my Markdown skill so I try to type them here.

Thanks for [this post][] which introduces the math functions of Markdown in great detail.

<!-- more -->

## Background

A photon of initial wavelength $\lambda_i$ collides a rest electron with mass $m_e$ at a plane. Then, we define that the angle between the initial path of the photon and its final path is $\phi$. And, the angle between the initial path of the photon and the motion path of the electron is $\theta$. The final wavelength of photon is $\lambda_f$ while the frequency doesn't change. The velocity of electron is $\vec{v}$ which implies its momentum $\vec{p} = \frac{m_e \vec{v}}{\sqrt{1- v^2/c^2}}$ after the correction of special relativity.

### Conservation of energy

For a photon of frequency $f$, its energy $E$:
$$ E = hf = \frac{hc}{\lambda} \qquad where \quad c = f\lambda $$
and $h$ is the Planck constant.

From mass-energy equivalence, a particle of mass $m$ has a rest energy $ E = mc^2 $.

The sum energy of initial photon and rest electron equals the sum energy of final photon and motional electron since the law of conservation of energy.

$$ E_{pi} = \frac{hc}{\lambda_i} \text{,}\quad E_{ei} = m_ec^2 $$
$$ E_{pf} = \frac{hc}{\lambda_f} \text{,}\quad E_{ef} = \frac{m_ec^2}{\sqrt{1-\frac{v^2}{c^2}}} $$

Now, we have equation $\text{(1)}$ by plusing the formers.

$$ E_{pi}+E_{ei}=E_{pf}+E_{ef} $$
$$ \Downarrow $$
$$\begin{equation}\label{eq1}
\frac{hc}{\lambda_i} + m_ec^2 = \frac{hc}{\lambda_f} + \frac{m_ec^2}{\sqrt{1-\frac{v^2}{c^2}}}
\end{equation}$$

### Conservation of momentum

If we define the direction of the motion path of initial photon is x-axis, then we know that the initial momentum of photon $\vec{p_{pi}} = \lbrace p_{pix} ,\; p_{piy} \rbrace$ where $p_{pix} = \frac{h}{\lambda_i}$ and $p_{piy} = 0$. The x-component of the final momentum of photon $p_{pfx} = \frac{h}{\lambda_f} \cos{\phi}$ and its y-component $p_{pfy} = \frac{h}{\lambda_f} \sin{\phi}$.

From the precondition, the momentum of rest electron can be presented as
$$ \vec{p_{ei}} = \lbrace p_{eix} ,\; p_{eiy} \rbrace = \text{\{0, 0\}} $$
and its final momentum:
$$ \vec{p_{ef}} = \frac{m_e \vec{v}}{\sqrt{1- v^2/c^2}} = \lbrace p_{efx} ,\; p_{efy} \rbrace = \left\{ \frac{m_ev}{\sqrt{1-\frac{v^2}{c^2}}} \cos{\theta} ,\; -\frac{m_ev}{\sqrt{1-\frac{v^2}{c^2}}} \sin{\theta} \right\} $$

Conclude these components. We can obtain equation $\text{(2)}$ and $\text{(3)}$ from the law of conservation of momentum.

\begin{equation}\label{eq2}
p_{pix} + p_{eix} = p_{pfx} + p_{efx} \quad \Rightarrow \quad

\frac{h}{\lambda_i} = \frac{h}{\lambda_f} \cos{\phi} + \frac{m_ev}{\sqrt{1-\frac{v^2}{c^2}}} \cos{\theta}
\end{equation}

\begin{equation}\label{eq3}
p_{piy} + p_{eiy} = p_{pfy} + p_{efy} \quad \Rightarrow \quad

0 = \frac{h}{\lambda_f} \sin{\phi} - \frac{m_ev}{\sqrt{1-\frac{v^2}{c^2}}} \sin{\theta}
\end{equation}

## Derivation

Start from squaring the equation $\text{(2)}$.

\begin{align}
\frac{h}{\lambda_i} - \frac{h}{\lambda_f} \cos{\phi} & = \frac{m_ev}{\sqrt{1-\frac{v^2}{c^2}}} \cos{\theta}  &\text{additive identity from (2)} \tag{2'}\\

\left( \frac{h}{\lambda_i} - \frac{h}{\lambda_f} \cos{\phi} \right)^2 & = \left( \frac{m_ev}{\sqrt{1-\frac{v^2}{c^2}}} \cos{\theta} \right)^2  &\text{square of (2')} \tag{4}\\

\frac{h^2}{\lambda_i^2} - 2\frac{h^2}{\lambda_i\lambda_f}\cos{\phi} + \frac{h^2}{\lambda_f^2} \cos^2{\phi} & = \frac{m_e^2 v^2}{1-\frac{v^2}{c^2}} \cos^2{\theta}  &\tag{4'}\\
\end{align}

Square the equation $\text{(3)}$, too.

\begin{align}
\frac{h}{\lambda_f} \sin{\phi} & = \frac{m_ev}{\sqrt{1-\frac{v^2}{c^2}}} \sin{\theta}  &\text{additive identity from (3)} \tag{3'}\\

\left( \frac{h}{\lambda_f} \sin{\phi} \right)^2 & = \left( \frac{m_ev}{\sqrt{1-\frac{v^2}{c^2}}} \sin{\theta} \right)^2  &\text{square of (3')} \tag{5}\\

\frac{h^2}{\lambda_f^2} \sin^2{\phi} & = \frac{m_e^2 v^2}{1-\frac{v^2}{c^2}} \sin^2{\theta}  &\tag{5'}\\
\end{align}

Now, plus $\text{(4')}$ and $\text{(5')}$.

\begin{align}

\left( \frac{h^2}{\lambda_i^2} - 2\frac{h^2}{\lambda_i\lambda_f}\cos{\phi} + \frac{h^2}{\lambda_f^2} \cos^2{\phi} \right) + \frac{h^2}{\lambda_f^2} \sin^2{\phi} & = \frac{m_e^2 v^2}{1-\frac{v^2}{c^2}} \cos^2{\theta} + \frac{m_e^2 v^2}{1-\frac{v^2}{c^2}} \sin^2{\theta}  &\tag{6}\\

\frac{h^2}{\lambda_i^2} - 2\frac{h^2}{\lambda_i\lambda_f}\cos{\phi} + \frac{h^2}{\lambda_f^2} (\cancelto{1}{\cos^2{\phi} + \sin^2{\phi}}) & = \frac{m_e^2 v^2}{1-\frac{v^2}{c^2}} (\cancelto{1}{\cos^2{\theta} + \sin^2{\theta}})  &\tag{6'}\\

\frac{h^2}{\lambda_i^2} - 2\frac{h^2}{\lambda_i\lambda_f}\cos{\phi} + \frac{h^2}{\lambda_f^2} & = \frac{m_e^2 v^2}{1-\frac{v^2}{c^2}}  &\tag{6''}\\
\end{align}

$\text{(6'')}$ applies substitution of $\sin^2\theta + \cos^2\theta = 1$ on $\text{(6')}$.

Put $\text{(6'')}$ in mind. It's time to handle the equation $\text{(1)}$!

\begin{align}
\frac{h}{\lambda_i} + m_ec & = \frac{h}{\lambda_f} + \frac{m_ec}{\sqrt{1-\frac{v^2}{c^2}}}  &\text{$(1)\div c$} \tag{1'}\\

\frac{h}{\lambda_i} + m_ec - \frac{h}{\lambda_f} & = \frac{m_ec}{\sqrt{1-\frac{v^2}{c^2}}}  &\text{additive identity from (1')} \tag{1''}\\

\left( \frac{h}{\lambda_i} + m_ec - \frac{h}{\lambda_f} \right)^2 & = \left( \frac{m_ec}{\sqrt{1-\frac{v^2}{c^2}}} \right)^2  &\text{square of (1'')} \tag{7}\\

\left[ \left( \frac{h}{\lambda_i} - \frac{h}{\lambda_f} \right) + m_ec \right]^2 & = \frac{m_e^2 c^2}{1-\frac{v^2}{c^2}}  &\tag{7'}\\
\end{align}

Then, minus $\text{(6'')}$ from $\text{(7')}$, which equals squared $\text{(1)}$ minus the sum of squared $\text{(2)}$ and squared $\text{(3)}$, to get the equation $\text{(8)}$.

$$\begin{equation} \tag{8}
\left[ \left( \frac{h}{\lambda_i} - \frac{h}{\lambda_f} \right) + m_ec \right]^2 - \left( \frac{h^2}{\lambda_i^2} - 2\frac{h^2}{\lambda_i\lambda_f}\cos{\phi} + \frac{h^2}{\lambda_f^2} \right) = \left( \frac{m_e^2 c^2}{1-\frac{v^2}{c^2}} \right) - \left( \frac{m_e^2 v^2}{1-\frac{v^2}{c^2}} \right)
\end{equation}$$

Simplify $\text{(8)}$ from left hand side.

\begin{aligned}
L.H.S. & = \left[ \left( \frac{h}{\lambda_i} - \frac{h}{\lambda_f} \right) + m_ec \right]^2 - \left( \frac{h^2}{\lambda_i^2} - 2\frac{h^2}{\lambda_i\lambda_f}\cos{\phi} + \frac{h^2}{\lambda_f^2} \right) \\

& = \left( \frac{h}{\lambda_i} - \frac{h}{\lambda_f} \right)^2 + 2m_ec \left( \frac{h}{\lambda_i} - \frac{h}{\lambda_f} \right) + m_e^2 c^2 - \frac{h^2}{\lambda_i^2} + \frac{2h^2}{\lambda_i\lambda_f}\cos{\phi} - \frac{h^2}{\lambda_f^2} \\

& = \cancel{\frac{h^2}{\lambda_i^2}} - \frac{2h^2}{\lambda_i\lambda_f} + \cancel{\frac{h^2}{\lambda_f^2}} + 2m_ech \frac{\lambda_f - \lambda_i}{\lambda_i \lambda_f} + m_e^2 c^2 - \cancel{\frac{h^2}{\lambda_i^2}} + \frac{2h^2}{\lambda_i\lambda_f}\cos{\phi} - \cancel{\frac{h^2}{\lambda_f^2}} \\

& = \frac{2h^2}{\lambda_i\lambda_f}\cos{\phi} - \frac{2h^2}{\lambda_i\lambda_f} + 2m_ech \frac{\lambda_f - \lambda_i}{\lambda_i\lambda_f} + m_e^2 c^2 \\

& = \frac{2h^2}{\lambda_i\lambda_f} \left( \cos\phi -1 \right) + \frac{2h^2}{\lambda_i\lambda_f} \frac{m_ec}{h} (\lambda_f - \lambda_i) + m_e^2 c^2 \\

& = \frac{2h^2}{\lambda_i\lambda_f} \left[ \left( \cos\phi -1 \right) + \frac{m_ec}{h} (\lambda_f - \lambda_i) \right] + m_e^2 c^2 \\
\end{aligned}

Deal with the other side.

\begin{aligned}
R.H.S. & = \left( \frac{m_e^2 c^2}{1-\frac{v^2}{c^2}} \right) - \left( \frac{m_e^2 v^2}{1-\frac{v^2}{c^2}} \right) \\

& = \frac{m_e^2 \left( c^2 - v^2 \right) }{1-\frac{v^2}{c^2}} \\

& = \frac{m_e^2 \left( c^2 - v^2 \right) }{\frac{c^2 - v^2}{c^2}} \\

& = m_e^2 \frac{\left( c^2 - v^2 \right) }{\left( c^2 - v^2 \right) } c^2 \\

& = m_e^2 c^2 \\
\end{aligned}

And combine the both sides.

$$
\xcancel{\frac{2h^2}{\lambda_i\lambda_f}} \left[ \left( \cos\phi -1 \right) + \frac{m_ec}{h} (\lambda_f - \lambda_i) \right] + \cancel{m_e^2 c^2} = \cancel{m_e^2 c^2}
$$
$$ \Downarrow $$
$$ \lambda_f - \lambda_i = \frac{h}{m_ec} \left( 1 - \cos\phi \right) $$

We also can define that $\Delta \lambda \equiv \lambda_f - \lambda_i$ and obtain a clearer presentation of Compton wavelength.

$$
\Delta\lambda = \frac{h}{m_ec} \left( 1 - \cos\phi \right)
$$


[this post]: https://blog.maxkit.com.tw/2020/02/markdown.html "如何在 Markdown 輸入數學公式及符號"