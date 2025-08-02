Quantum Galton Board – Exponential Walk Simulation
This project implements a Quantum Galton Board (QGB) using Qiskit to simulate a biased quantum walk that approximates an exponential distribution. Inspired by classical Galton boards (used to demonstrate binomial distributions), the quantum version uses entanglement and controlled swaps to steer amplitude across qubits.

📁 File Overview
Hadamard_Gaussian_Analysis.py
This script builds and simulates a quantum walk biased toward lower-indexed qubits, aiming to replicate exponential decay behavior.

⚙️ Dependencies
Make sure you have the following installed:

bash
Copy
Edit
pip install qiskit qiskit-aer matplotlib numpy scipy
🚀 How to Run
bash
Copy
Edit
python Hadamard_Gaussian_Analysis.py
📌 Code Walkthrough
1. Imports
python
Copy
Edit
from qiskit import ...
Qiskit: For quantum circuit construction and simulation.

NumPy & SciPy: Math tools for processing and distribution fitting.

Matplotlib: Visualization.

Counter: Tallying outcomes.

2. Quantum Peg Logic
python
Copy
Edit
def biased_peg(qc, control, mid):
Implements a single biased "peg" unit:

cswap: Swaps amplitude between mid and left to encourage leftward bias.

ry: Applies a rotation to make the split uneven → more amplitude goes left.

3. Circuit Construction
python
Copy
Edit
def generate_exponential_walk(n_qubits, n_levels, shots=8192):
Steps:
Initialize a circuit with n_qubits.

qc.x(qr[mid]): Starts the ball at the center.

qc.h(qr[0]): Superposition on control qubit.

Loop through n_levels (layers):

Each layer resets and re-prepares the control.

Pegs are placed at different positions across the layer.

Amplitude is funneled leftward to simulate exponential decay.

Measure all position qubits (qubits 1 to end).

4. Simulation and Raw Output
python
Copy
Edit
sim = AerSimulator()
result = sim.run(qc).result()
counts = result.get_counts()
Circuit is simulated using Qiskit's noiseless AerSimulator.

Output is shown as a histogram of bitstrings indicating where the "ball" landed.

5. Postprocessing
python
Copy
Edit
def postprocess_and_plot_exponential(counts):
Decode bitstrings to find where the ball landed (based on 1-hot position).

Group into blocks of 8 to reduce variance and better observe trends.

Fit to an exponential distribution using SciPy.

Compute KL Divergence and Total Variation Distance (TVD) for accuracy.

📊 Output
The script generates two main plots:

Raw QGB Output Histogram (bitstring counts).

Block-Summed Histogram vs. Ideal Exponential (with fitted curve).

📈 Metrics
To evaluate how closely the simulation resembles exponential decay:

KL Divergence: Measures information loss between simulation and ideal.

TVD: Measures maximum difference between the two distributions.

📚 Notes
The quantum circuit is composed of layered biased pegs to mimic exponential bias.

Controlled-swap (CSWAP) and RY gates control directionality and entanglement.

The simulation mimics multiple passes through pegs just like classical balls falling in a board.

📌 Example Usage Output
bash
Copy
Edit
KL Divergence (Exponential Walk): 0.037
TVD (Exponential Walk): 0.118
📦 Author Notes
This is part of a broader exploration of QGB variants, including Gaussian and Hadamard walks.

Contributions welcome for improving noise robustness and implementing real-device backends.
