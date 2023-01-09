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


### 3. Source the new virtual env and try to load the new qasm circuit and compare the unitaries
Comparing unitaries generated with newer version of qiskit and older version of qiskit can guaratee the circuits are identical.
A script snippet is like this:
```
qasm_path = path+'/qc0.qasm'
with open(qasm_path,'r') as f:
    c0_str  = f.readlines()
c0_qasm = ''.join(c0_str)
cq = qiskit.QuantumCircuit.from_qasm_str(c0_qasm)
unitary_path = qasm_path + '.npy'

try:
    # By default unitary is named as qasm file plus .npy
    with open(unitary_path,'rb') as f:
        m0=np.load(f)
    loaded = True
except:
    warnings.warn(f'No unitary file loaded/found from {unitary_path} so I did not comare unitarties between provided unitary.')
    loaded = False

if loaded:
    threashold = 1e-6
    op = qiskit.quantum_info.Operator(cq)
    diff = np.linalg.norm(op.data/op.data[0,2]-m0/m0[0,2])

    if diff > threashold:
    #     print(f'Unitary differences ({diff}) greater than threashold:{threashold}.')
        raise ValueError(f'Unitary differences ({diff}) greater than threashold:{threashold}.')
```

The idea is that two unitaries should be identical (within threshold) update to a global phase.  The division in `op.data/op.data[0,2]-m0/m0[0,2]` is to take care of phase issue and might not work with your case if element `[0,2]` is too small or not exist. Please modify accordingly if necessary.


