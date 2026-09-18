
# Quantum Entanglement Detector
 
Applying classical machine learning to a quantum physics problem: detecting two qubit entanglement from measurement data alone.
 
## Problem
 
Two qubits can be entangled (correlated) or separable (independent). Determining which requires computing an entanglement measure from the full quantum state, which becomes expensive as system size grows.
 
This project asks a different question: **can a classifier decide from measurement outcomes alone, without reconstructing the state?**
 
## Approach
 
```
quantum state  ->  15 Pauli expectation values  ->  classifier  ->  entangled / separable
```
 
| Stage | Method |
|---|---|
| Data generation | 5,000 two qubit states via Qiskit `random_statevector` |
| Labeling | Concurrence, threshold 0.1 |
| Features | 15 two qubit Pauli expectation values (IX ... ZZ) |
| Models | Logistic Regression baseline, Random Forest |
| Split | 4,000 train / 1,000 test, stratified |
 
The negative class is constructed deliberately as a tensor product of two independent single qubit states, which guarantees concurrence equal to zero. Random sampling alone almost never produces separable states, as the concurrence distribution shows.
 
## Results
 
| Model | Accuracy | F1 | ROC AUC |
|---|---|---|---|
| Logistic Regression (baseline) | 0.4990 | 0.4248 | |
| Random Forest | **0.9270** | **0.9202** | **0.9805** |
 
The linear baseline performs at chance level on the exact same features. This is the main finding: the decision boundary is non linear, which is consistent with concurrence being a non linear function of the state amplitudes, `C = 2|ad - bg|`.
 
### Confusion matrix (Random Forest)
 
```
              predicted
              sep   ent
true  sep  [ 506     3 ]
      ent  [  70   421 ]
```
 
Errors are asymmetric by a factor of 23 to 1. Separable states are detected almost perfectly (99.4 percent), entangled states less so (85.7 percent).
 
### Error analysis
 
| Group | Mean concurrence |
|---|---|
| Missed entangled states | 0.2484 |
| All entangled states | 0.6025 |
 
Errors concentrate in weakly entangled states near the labeling threshold, not uniformly across the class. The model reliably detects strong entanglement and struggles where measurement statistics closely resemble separable states. The residual error is driven by threshold ambiguity rather than by model capacity.
 
## Repository structure
 
```
entanglement-detector/
├── notebooks/
│   ├── 01_data_generation.ipynb    # generate + label 5,000 states
│   ├── 02_features.ipynb           # Pauli expectation feature matrix
│   └── 03_modeling.ipynb           # baseline, model, evaluation
├── data/
│   ├── labels.csv                  # state_id, source, concurrence, label
│   ├── states.npy                  # (5000, 4) complex amplitudes
│   ├── X_features.npy              # (5000, 15) feature matrix
│   └── y_labels.npy                # (5000,) binary target
├── requirements.txt
└── README.md
```
 
Notebooks must be run in order. Each reads from and writes to `data/`.
 
## Reproducing
 
```bash
pip install -r requirements.txt
```
 
Run `01`, then `02`, then `03`. Random seed is fixed at 42 throughout.
 
## Limitations
 
Stated openly rather than left for the reader to find.
 
1. **Simulated data.** States come from the Qiskit statevector simulator, not from quantum hardware. The physics governing them is real, but there is no device noise.
2. **Threshold artifact.** States with concurrence between 0 and 0.1 are labeled separable despite being weakly entangled. In the 5,000 state dataset this affects a small number of cases, and it is the main source of remaining error.
3. **Two qubits only.** At this size, computing concurrence directly is cheaper than any model. The value here is proof of principle: the signal exists in measurement data and a classifier can extract it.
## Next steps
 
- Extend to 3 and 4 qubit systems, where direct computation becomes costly and the approach starts to pay off
- Replace exact expectation values with finite shot measurements, then measure accuracy as a function of shot count
- Validate on real IBM Quantum hardware to test robustness against device noise
## Stack
 
Python, Qiskit, NumPy, pandas, scikit learn, matplotlib
 
