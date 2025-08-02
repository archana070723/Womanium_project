from qiskit import QuantumCircuit, ClassicalRegister, QuantumRegister
from qiskit\_aer import AerSimulator
from qiskit.visualization import plot\_histogram
import numpy as np
import matplotlib.pyplot as plt
from collections import Counter
from scipy.stats import expon, norm, entropy

# Biased quantum peg to induce exponential decay

def biased\_peg(qc, control, mid):
left = mid - 1
right = mid + 1
if left != control and mid != control:
qc.cswap(control, left, mid)  # Funnel to the left
qc.ry(np.pi / 4, qc.qubits\[control])  # Entangle with asymmetric amplitude
\# Skipping right cswap to favor decay toward left

def generate\_exponential\_walk(n\_qubits, n\_levels, shots=8192):
qr = QuantumRegister(n\_qubits)
cr = ClassicalRegister(n\_qubits)
qc = QuantumCircuit(qr, cr)
mid = n\_qubits // 2
control = 0
qc.x(qr\[mid])  # Ball starts in center
qc.h(qr\[control])  # Initial superposition on control qubit

```
for level in range(n_levels):
    qc.reset(qr[control])
    qc.h(qr[control])
    offset = mid - level
    for i in range(level + 1):
        if offset + 1 < n_qubits:
            biased_peg(qc, control, offset)
            offset += 2

qc.barrier()
for i in range(1, n_qubits):
    qc.measure(qr[i], cr[i])

sim = AerSimulator()
result = sim.run(qc, shots=shots).result()
counts = result.get_counts()

plt.figure(figsize=(10, 5))
plot_histogram(counts, title="Raw Exponential Walk Output (Layered Biased Pegs)")
plt.tight_layout()
plt.show()
return counts
```

def postprocess\_and\_plot\_exponential(counts):
samples = \[]
for bitstring, freq in counts.items():
index = bitstring\[::-1].find('1')
if index != -1:
samples.extend(\[index] \* freq)

```
block_size = 8
num_blocks = len(samples) // block_size
block_sums = [sum(samples[i * block_size:(i + 1) * block_size]) for i in range(num_blocks)]

hist = Counter(block_sums)
x = sorted(hist)
y = [hist[i] for i in x]

scale = np.std(block_sums)
x_smooth = np.linspace(min(x), max(x), 500)
exponential = len(block_sums) * expon.pdf(x_smooth, scale=scale)

plt.figure(figsize=(10, 5))
plt.bar(x, y, width=0.8, alpha=0.6, label="Simulated Block Sums")
plt.plot(x_smooth, exponential, 'r--', label="Exponential Fit")
plt.xlabel("Block Sum")
plt.ylabel("Frequency")
plt.legend()
plt.title("Exponential Walk Output vs. Ideal Exponential")
plt.grid()
plt.tight_layout()
plt.show()

sim = np.array([hist.get(i, 0) for i in x]) / sum(y)
ideal = expon.pdf(x, scale=scale)
ideal /= np.sum(ideal)
kl = entropy(sim + 1e-12, ideal + 1e-12)
tvd = 0.5 * np.sum(np.abs(sim - ideal))
return kl, tvd
```

# Example usage

if **name** == "**main**":
layers = 4
n\_qubits = 2 \* layers + 2
counts\_exp = generate\_exponential\_walk(n\_qubits, layers)
kl\_exp, tvd\_exp = postprocess\_and\_plot\_exponential(counts\_exp)
print("KL Divergence (Exponential Walk):", kl\_exp)
print("TVD (Exponential Walk):", tvd\_exp)
