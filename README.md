# Womanium_project
# Quantum Galton Box Simulation with Qiskit

This repository implements a **quantum version of the Galton Box (Plinko)** using Hadamard-based quantum walks, inspired by the Universal Statistical Simulator framework proposed by Carney and Varcoe (arXiv:2202.01735).

## Overview

The classical Galton Box demonstrates the central limit theorem by simulating random left/right bounces of particles through pegs. This project recreates that behavior using a quantum circuit where:

* Each Hadamard gate models a 50/50 quantum decision (left/right).
* Measurement yields a histogram resembling the classical binomial distribution.

## Quantum Circuit Description

* A `num_layers`-qubit quantum circuit is initialized in the \$|0\rangle^{\otimes n}\$ state.
* Hadamard gates are applied to all qubits.
* Each qubit is measured to produce a bitstring.
* Repeating this experiment multiple times (shots) generates a probability histogram of outcomes.

## Dependencies

Install the required libraries using pip:

```bash
pip install qiskit matplotlib
```

## How to Run

```python
from qiskit import QuantumCircuit, transpile
from qiskit_aer import AerSimulator
from qiskit.visualization import plot_histogram
import matplotlib.pyplot as plt

def quantum_galton_box(num_layers=5, shots=2048):
    qc = QuantumCircuit(num_layers, num_layers)
    for i in range(num_layers):
        qc.h(i)
    qc.measure(range(num_layers), range(num_layers))
    simulator = AerSimulator()
    tqc = transpile(qc, simulator)
    result = simulator.run(tqc, shots=shots).result()
    counts = result.get_counts()
    return qc, counts

qc, counts = quantum_galton_box(num_layers=5, shots=2048)
qc.draw('mpl')
plot_histogram(counts, title="Quantum Galton Box Output (5 Layers)")
plt.show()
```

## Example Output

* Bitstrings such as `00000`, `01010`, `11100` appear.
* The distribution of outputs closely resembles a binomial curve.

## Extensions

* Change `num_layers` to 8 or 10 to see convergence to a Gaussian.
* Use weighted unitary rotations instead of Hadamards for biased deflection.
* Execute the circuit on real quantum hardware (e.g., IonQ or IBMQ).

## Reference

* M. Carney and B. Varcoe, "Universal Statistical Simulator," arXiv:2202.01735 \[quant-ph], 2022.

---

MIT License
