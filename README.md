🧠 Part 1: Imports and Setup
python :

from qiskit import QuantumCircuit, ClassicalRegister, QuantumRegister
from qiskit_aer import AerSimulator
from qiskit.visualization import plot_histogram
from scipy.stats import norm, expon, entropy
import numpy as np
import matplotlib.pyplot as plt
Import essential modules for building circuits, simulating, visualizing, and comparing with ideal probability distributions.

⚙️ Part 2: Quantum Peg Logic
python:

def quantum_peg(qc, control, mid):
    left = mid - 1
    right = mid + 1
    print("Quantum peg on qubits:", left, mid, right)
    if left != control and mid != control:
        qc.cswap(control, left, mid)   # Swap if control=1 (entangles path)
    qc.cx(mid, control)               # Entangles mid with control
    if mid != control and right != control:
        qc.cswap(control, mid, right) # Another conditional path

        
Implements a quantum "peg": like a pinball bumper that conditionally redirects the quantum ball left or right.

cswap (Fredkin gate) swaps the two targets if the control is 1.

cx (CNOT) entangles the mid qubit with the control to spread amplitude.

🧱 Part 3: QGB Circuit Construction
python :

def generate_qgb_qiskit(n_qubits, n_levels, shots=8192):
    qr = QuantumRegister(n_qubits)
    cr = ClassicalRegister(n_qubits)
    qc = QuantumCircuit(qr, cr)
Creates n_qubits quantum + classical registers and initializes the circuit.

python :

    mid = n_qubits // 2
    qc.x(qr[mid])  # Ball starts in center
    qc.h(qr[0])    # Control qubit is placed in superposition
Initialize the "ball" at the center qubit.

The control qubit (index 0) gets a Hadamard to spread amplitudes.

🔁 Part 4: Apply Layers of Pegs
python :

    for j in range(n_levels):
        mid2 = mid
        print("For level=", j)
        control = 0
Each level represents a layer of pegs (like rows in Galton board).

python :

        if j == 0:
            quantum_peg(qc, control, mid)
At the first level, apply peg only at the center.

python :

        else:
            qc.reset(qr[control])
            qc.h(qr[control])  # Recreate superposition
            mid2 = mid2 - j
For subsequent layers, reset control and spread it again.

python :

            for i in range(j + 1):
                rightmost = mid2 + 1
                if mid2 - 1 != 0 and mid2 + 1 != n_qubits:
                    print("mid2=", mid2)
                    quantum_peg(qc, control, mid2)
                    if i != j:
                        qc.cx(qr[rightmost], qr[control])
                    mid2 = mid2 + 2
Loop through all peg positions in current level.

After each peg, connect control with upcoming pegs (for propagation).

Shifts mid2 by 2 to space the pegs symmetrically.

📏 Part 5: Measurement and Execution
python :

    qc.barrier()
    for i in range(1, n_qubits):
        qc.measure(qr[i], cr[i])
Adds a barrier for circuit clarity and then measures all non-control qubits.

python :

    print(qc.draw())
    sim = AerSimulator()
    job = sim.run(qc, shots=shots)
    result = job.result()
    counts = result.get_counts()
Runs the simulation on Aer and returns the measurement outcomes.

python :

    plt.figure(figsize=(10, 5))
    plot_histogram(counts, title="Raw QGB Measurement Counts")
    plt.show()
    return counts
Plots the raw histogram of outcomes before postprocessing.

🧮 Part 6: Result Postprocessing
(A) Direct distribution comparison
python :

def process_results(counts, dist_type="gaussian"):
Processes histogram against a target distribution: Gaussian or Exponential.

python :

    keys = list(counts.keys())
    values = [int(k, 2) for k in keys]
    total = sum(counts.values())
    normalized_sim = {int(k, 2): v / total for k, v in counts.items()}
Convert bitstrings to integers and normalize the histogram.

python :

    x = np.arange(min(values), max(values)+1)
Define the range of expected output.

python :

    if dist_type == "gaussian":
        mu, sigma = 16, np.sqrt(5)
        ideal = norm.pdf(x, mu, sigma)
    elif dist_type == "exponential":
        ideal = expon.pdf(x, scale=4)
    else:
        ideal = np.ones_like(x) / len(x)
Set the ideal distribution for comparison.

python :

    sim = np.array([normalized_sim.get(i, 0) for i in x])
    kl = entropy(sim + 1e-12, ideal + 1e-12)
    tvd = 0.5 * np.sum(np.abs(sim - ideal))
Compute KL divergence and Total Variation Distance.

🔧 Part 7: Block Sum Postprocessing
python

def process_and_plot_blocked(counts, dist_type="gaussian"):
Group outcomes into blocks of 8 and sum their indices.

python

    for s in strings:
        index = s[::-1].find('1')
        remapped.append(index if index != -1 else 4)
Convert 1-hot outcomes into positions (e.g., output qubit index with '1').

python

    blocks = [sum(remapped[i:i+8]) for i in range(0, len(remapped), 8) if len(remapped[i:i+8]) == 8]
Sum groups of 8 outcomes to mimic classical distribution.

python :

    plt.bar(x_vals, sim, label="QGB Simulated", alpha=0.6)
    plt.plot(x_vals, ideal, label=f"Ideal {dist_type.title()} Distribution", color="r", linestyle="--")
Plot histogram and ideal distribution overlay.

🧪 Final Run
python
layers = 4
n_qubits = 2 * layers + 2
counts = generate_qgb_qiskit(n_qubits, layers)
Use 4 layers (total 10 qubits) and simulate the QGB.
