# Womanium_project
# Quantum Galton Box Simulation with Qiskit

This repository implements a **quantum version of the Galton Box (Plinko)** using Hadamard-based quantum walks, inspired by the Universal Statistical Simulator framework proposed by Carney and Varcoe (arXiv:2202.01735).

## Overview

The classical Galton Box demonstrates the central limit theorem by simulating random left/right bounces of particles through pegs. This project recreates that behavior using a quantum circuit where:

# Quantum Galton Board (QGB) — Generalized Construction

This implementation simulates a Quantum Galton Board (QGB) inspired by the *Universal Statistical Simulator* paper. The QGB models the statistical behavior of a classical Galton board using quantum circuits. As the number of pegs increases, the output distribution of the quantum circuit approximates a binomial (→ Gaussian) distribution.

## 🧱 Qubit Layout

For `num_pegs = n`, we use the following qubit allocation:

| Role              | Count     | Index Range                 |
|-------------------|-----------|-----------------------------|
| Control qubit     | 1         | `0`                         |
| Ball wires        | `2n`      | `1` to `2n` (pairs of wires for each peg) |
| Output wires      | `n`       | `2n+1` to `3n`              |

### ➕ Total qubits required:
```
total_wires = 1 (control) + 2n (ball/intermediate) + n (outputs) = 3n + 1
```

## 🔁 Circuit Logic (Per Peg `i`)

1. **Randomize control**:
```python
qml.Hadamard(wires=0)
```

2. **Controlled SWAP between two ball wires**:
```python
wire_a = 1 + 2*i
wire_b = wire_a + 1
qml.ctrl(qml.SWAP, control=0)(wires=[wire_a, wire_b])
```

3. **Inverted CNOT to rebalance control**:
```python
qml.CNOT(wires=[wire_b, 0])
```

4. **Transfer to unique output wire**:
```python
wire_out = 2*num_pegs + 1 + i
qml.SWAP(wires=[wire_b, wire_out])
```

## 📏 Measurement

After running the circuit:
```python
qml.sample(wires=[2*num_pegs + 1 + i for i in range(num_pegs)])
```

Each shot produces a bitstring. The **Hamming weight** of this bitstring is the output bin (column), analogous to where the ball lands in the classical Galton board.

## 🎯 Output Distribution

The simulated output distribution approximates a **binomial distribution**, which converges to a **Gaussian distribution** for large `num_pegs`.

## 📈 Example Plot

We recommend plotting the Hamming weight frequencies and overlaying a Gaussian:

```python
from scipy.stats import norm

# Simulated histogram
counts = Counter(weights)
x_vals = sorted(counts.keys())
simulated_probs = [counts[x] / shots for x in x_vals]

# Gaussian overlay
mu = expected_mean   # e.g., center around μ = 16
sigma = sqrt(var)    # e.g., variance = 5
x_smooth = np.linspace(min(x_vals), max(x_vals), 300)
gaussian = norm.pdf(x_smooth, loc=mu, scale=sigma)
gaussian /= np.sum(gaussian)

# Plot
plt.bar(x_vals, simulated_probs)
plt.plot(x_smooth, gaussian)
```

## 🧠 Mapping to the Paper

| Paper Concept                      | Circuit Action                        |
|-----------------------------------|----------------------------------------|
| Ball hits peg → splits            | Hadamard on control                    |
| Peg logic                         | CSWAP + inverted CNOT + output SWAP    |
| Output column                     | Hamming weight of output wires         |
| Mid-circuit control randomness    | New Hadamard each round                |
| Gaussian emerges with more pegs   | Seen in output distribution            |
