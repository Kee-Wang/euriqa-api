# EURIQA API Guide

This guides provides the minimal info for external users to use the EURIQA-blue API correctly.

EURIQA provides the API interface which submits a list of circuits and collect the list of probabiilty histograms:
`prob_vector = run_on_EURIQA(circuits, num_shots)`. More inforamtion returns is possible but needs to coordinate with the experiment operator in advance.

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


# Do necessary preprocessing including basis transformation to generate list of circuits in OpenQASM2.0 string format.
# For local test (returns exact simulation prob result):
probs_local = run_on_EURIQA(qasm_circuits,num_shots=100, run_simulation_local=True)
# For actual experiment (this shows what it looks like to experimentalists, api calls to the actual machine is only accessable to EURIQA internal computers):
# probs_exp = run_on_EURIQA(qasm_circuits,num_shots=100)

print(probs_local)
# Then do post-processing using probs
```

Expected output:
```
[array([0.5, 0. , 0. , 0.5]), array([0.25, 0.25, 0.25, 0.25])]
```

!!! It is highly recommended provide version info on key modules or requiremnets.txt to best reproduce the working enviroment.

# More on qasm string
Euriqa runs on an older version of qiskit might not recognize the qasm string generated from newer version of qiskit even if both of them are OpenQASM2.0. In order to test if your qasm string works on the eurqia system, please test your qasm strings with correct qiskit version. The following is recommended procedure.

### 1. Generate qasm string with the working code (which presumebaly uses the newer version of qiskit) and save it locally and save the unitary with additional extension `.npy`.
```
qc0.remove_final_measurements()
op0 = qiskit.quantum_info.Operator(qc0).data
with open(path+'/qc0.qasm','w') as f:
    f.write(qc0.qasm())
np.save(path+'/qc0.qasm.npy',op0)

```

### 2. Create a fresh virtual env and install the specific qiskit version use the following:
```
pip install qiskit-terra==0.16.1
```
so that the version result of `qiskit.__qiskit_version__` is: `{'qiskit-terra': '0.16.1', 'qiskit-aer': None, 'qiskit-ignis': None, 'qiskit-ibmq-provider': None, 'qiskit-aqua': None, 'qiskit': None}`.
Also the `numpy` version is `0.18.1`.


### 3. Source the new virtual env and try to load the new qasm circuit and compare the unitaries
Comparing unitaries generated with newer version of qiskit and older version of qiskit can guaratee the circuits are identical.
Depending on your qiksit version, you might use 
`abs(qiskit.quantum_info.Statevector(cq).data)**2`
 or `abs(qiskit.quantum_info.Statevector.from_instruction(cq))**2` for newer and older qiskit version repectively.
    

