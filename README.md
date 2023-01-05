# EURIQA API Guide

This guides provides the minimal info for external users to use the EURIQA API correctly.

EURIQA provides the following API interface which submits a list of circuits and collect the list of probabiilty histograms:
`prob_vector = run_on_EURIQA(circuits, num_shots)`

For the purpose of code development, one can use the following example.
Please refer to the `euriqa_interface.py` for the docstrings of `run_on_EURIQA` for any input/output format issue.


```
import qiskit
import numpy as np
from euriqa_interface import run_on_EURIQA


qc0 = qiskit.QuantumCircuit(2)
qc0.rxx(np.pi/2,0,1)

qc1 = qc0.copy()
qc1.rx(np.pi/2,0)

qasm_circuits = [c.qasm() for c in  [qc0, qc1]]


# Do necessary preprocessing to generate list of circuits in OpenQASM2.0 string format.
# For local test:
probs_local = run_on_EURIQA(qasm_circuits,run_simulation_local=True)
# For actual experiment:
# probs_exp = run_on_EURIQA(qasm_circuits,num_shots=100)

print(probs_local)
# Then do post-processing using probs
```

Expected output:
```
[array([0.5, 0. , 0. , 0.5]), array([0.25, 0.25, 0.25, 0.25])]
```
