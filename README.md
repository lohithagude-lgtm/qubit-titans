# 0/1 Knapsack via QAOA (p = 1)

Variational quantum solution of a 5-item 0/1 knapsack problem, benchmarked against classical brute force.

**Instance:** values `[10, 4, 7, 5, 8]`, weights `[5, 3, 4, 2, 6]`, capacity `15`.
**Classical optimum:** items 1, 2, 3, 4 (value 26, weight 14).

## Approach
- The inequality constraint is converted to an equality using 4 binary slack bits.
- Cost function: `C(x,s) = -Σ v_i x_i + A (Σ w_i x_i + Σ 2^k s_k − W)²`, mapped to an Ising Hamiltonian on 9 qubits (checked numerically against the QUBO).
- Penalty `A` is scanned to keep the ground state feasible and optimal.
- p=1 QAOA circuit: H → RZ/RZZ cost layer → RX mixer, built in Qiskit and simulated exactly.
- A 5-qubit slack-free "unbalanced penalization" variant is included for comparison.

## Results
| Angles | P(valid) | P(optimal) | E[value \| valid] |
|---|---|---|---|
| Minimise ⟨C⟩ (standard) | 100% | 0.7% | 44% of optimal |
| Tuned to maximise P(optimal)* | 92.8% | 44.3% | 82.8% of optimal |

\*Uses knowledge of the optimum, so it shows the attainable best, not a blind run. With these angles the most probable selection is the true optimum.
Uniform random guessing gives P(valid) = 87.5% and P(optimal) = 3.1%.

Note: best-of-N shot sampling is not a discriminating metric here, since random sampling also finds the optimum on 5 items.

## Hardware metrics (9 qubits, best of 10 transpiler seeds)
| Topology | CX | Depth | SWAPs |
|---|---|---|---|
| all-to-all | 72 | 53 | 0 |
| star | 93 | 172 | 7 |
| line | 138 | 112 | 32 |
| ring | 138 | 112 | 32 |
| 3×3 grid | 112 | 99 | 16 |
| heavy-hex | 147 | 137 | 25 |

The squared constraint makes the interaction graph dense, so sparse topologies need many SWAPs.

## Contents
- `knapsack_qaoa.ipynb`: formulation, circuit, penalty tuning, results, (γ, β) landscape plot, topology metrics.

## Run
```bash
pip install qiskit numpy scipy matplotlib pandas jupyter
jupyter notebook knapsack_qaoa.ipynb
```

## Limitations
p=1 QAOA only modestly beats random sampling at finding the exact optimum. Results are from exact statevector simulation, not real hardware.
