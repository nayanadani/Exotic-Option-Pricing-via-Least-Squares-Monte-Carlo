# Pricing a Bermudan Asian Option with Least-Squares Monte Carlo

This project implements a **Least-Squares Monte Carlo (LSMC)** algorithm to price an exotic option combining **path dependence** and **early exercise features**.

The option considered is a **Bermudan Asian call option**, whose payoff depends on the running average of the underlying asset and which can be exercised at discrete dates before maturity.

The implementation includes two models for the underlying dynamics:

- Black–Scholes constant volatility model  
- Local volatility model calibrated from market data

---

# Problem

We consider an option written on a stock with initial price $S_0 = 100$ and maturity $T = 1$ year.

The option can be exercised at four dates: $t_1 = 0.25$, $t_2 = 0.5$, $t_3 = 0.75$, and $t_4 = 1$.

The payoff is a **call on the running arithmetic average** of the stock price.

The running average is defined as $A(t_j) = \frac{1}{j}\sum_{i=1}^{j} S(t_i)$ and the payoff at exercise date $t_j$ is $(A(t_j) - K)^+$ with strike $K = 98$ and dividend yield $q = 2\%$.

Because the payoff depends on past prices and early exercise is allowed, this contract is classified as an **exotic option**.

---

# Method: Least-Squares Monte Carlo

Pricing Bermudan or American options with path dependence cannot be done analytically.  
We therefore use the **Least-Squares Monte Carlo method** introduced by Longstaff & Schwartz (2001).

At each exercise date $t_j$, we compare:

- the **immediate payoff** $P_j = (A_j - K)^+$
- the **continuation value** $C_j = \mathbb{E}[V_{j+1} \mid S_j, A_j]$

If $P_j > C_j$, the option is exercised. Otherwise the option continues.

The continuation value is estimated using regression on simulated Monte Carlo paths.

---

# Regression Basis

The continuation value is approximated using a polynomial regression on the state variables:

$1, S_j, S_j^2, S_j^3, A_j, A_j^2, A_j^3$

where $S_j$ is the stock price and $A_j$ is the running average.

The regression is performed only on **in-the-money paths** to reduce bias.

---

# Stock Price Models

## Black–Scholes model

Under the risk-neutral measure, the stock price follows the stochastic differential equation

$dS_t = (r - q) S_t dt + \sigma S_t dW_t$

where $r$ is the risk-free rate, $q$ the dividend yield, and $\sigma$ the volatility.

---

## Local Volatility model

To capture the volatility smile observed in markets, we also price the option using a **local volatility model** with dynamics

$dS_t = (r - q) S_t dt + \sigma_{loc}(S_t, t) S_t dW_t$

The function $\sigma_{loc}(S,t)$ is obtained from a calibration of the implied volatility surface using the **Andreasen–Huge algorithm**.

---

# Monte Carlo Simulation

Stock price paths are simulated using an **Euler–Maruyama scheme**:

$S_{t+\Delta t} = S_t + (r - q) S_t \Delta t + \sigma(S_t,t) S_t \sqrt{\Delta t} Z$

where $Z \sim \mathcal{N}(0,1)$.

Weekly time steps are used with $\Delta t = 1/52$, and we simulate $100{,}000$ Monte Carlo paths.

---

# Results

### Black–Scholes model

Estimated price: $V \approx 6.90$  
Standard error: $SE \approx 0.032$  
95% confidence interval: $[6.84, 6.97]$

---

### Local volatility model

Estimated price: $V \approx 8.05$  
Standard error: $SE \approx 0.027$  
95% confidence interval: $[7.995, 8.102]$

The higher price under the local volatility model is consistent with the volatility smile observed in the implied volatility surface.

---

# References

Longstaff, F. A., & Schwartz, E. S. (2001).  
*Valuing American Options by Simulation: A Simple Least-Squares Approach.*  
Journal of Finance.