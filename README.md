# Practical Implementation for Optimizing Quantum Circuits

## Overview

The Python script provided interprets and converts an IBM OpenQASM code, which is a textual representation of a quantum circuit, into an algebraic expression. This script processes the specified quantum gates on qubits with the possibility of control-target relationships, thereby building and simplifying the corresponding algebraic expression by applying circuit identities representing the operations applied to the qubits over the sequence of gates.

The script follows the algorithm specified in the paper "Optimizing Quantum Circuits using Algebraic Expressions" by Johnson P Thomas, Varun Puram, and Krishnageetha Karuppasamy.

## Input

The input to this script is an OpenQASM file named "input.txt", which must be formatted correctly to represent the quantum circuit. OpenQASM is a hardware-agnostic description language used to write quantum circuits. Below is an example of how such a file might look:

```qasm
qreg q[3];
barrier q[0], q[1], q[2];
h q[2];
barrier q[0], q[1], q[2];
cz q[1], q[2];
barrier q[0], q[1], q[2];
h q[2];
barrier q[0], q[1], q[2];
cz q[1], q[0];
cx q[1], q[2];
barrier q[0], q[1], q[2];
```


### Explanation of the Input Format
The provided OpenQASM code snippet outlines a series of quantum gates and operations applied to a set of three qubits:

1. `qreg q[3];`  
   **Quantum Register Declaration**: Declares a quantum register named `q` containing three qubits. This register is used to store the quantum states of the circuit.

2. `barrier q[0], q[1], q[2];`  
   **Barrier**: Inserts a barrier for all qubits in the register, preventing optimizations by the quantum compiler that could alter the order or structure of operations up to this point.

3. `h q[2];`  
   **Hadamard Gate**: Applies the Hadamard gate to qubit `q[2]`. This gate places the qubit in a superposition state, evenly between 0 and 1 if it was initially in a pure state.

4. `barrier q[0], q[1], q[2];`  
   **Barrier**: Ensures that no transformations or optimizations affect the state immediately following the Hadamard gate and prior to further entangling operations.

5. `cz q[1], q[2];`  
   **Controlled-Z Gate**: A controlled-Z gate between qubit `q[1]` (control) and qubit `q[2]` (target) is applied, introducing a phase shift in the target qubit when both qubits are in the |1> state.

6. `barrier q[0], q[1], q[2];`  
   **Barrier**: Maintains the quantum entanglement created by the controlled-Z gate and prevents any compiler optimization before subsequent gates.

7. `h q[2];`  
   **Hadamard Gate**: A second Hadamard gate on qubit `q[2]` is used to potentially interfere with the states produced by previous gates.

8. `barrier q[0], q[1], q[2];`  
   **Barrier**: Preserves the state post-Hadamard gate application against any optimizations.

9. `cz q[1], q[0];`  
   **Controlled-Z Gate**: Entangles qubit `q[0]` with qubit `q[1]`, affecting the phase of qubit `q[0]`'s state |1> when both involved qubits are also in |1>.

10. `cx q[1], q[2];`  
    **Controlled-X Gate (CNOT)**: Applies a CNOT gate with qubit `q[1]` as control and qubit `q[2]` as target, flipping the state of the target qubit if the control qubit is in the |1> state.

11. `barrier q[0], q[1], q[2];`  
    **Barrier**: A final barrier to secure the quantum state as specified after the last CNOT gate application.

These steps collectively build a quantum circuit that involves quantum interference and entanglement, crucial elements for algorithms that leverage quantum mechanical properties for computation. Each barrier is strategically placed to ensure the sequential operations are executed precisely as intended, which is critical for maintaining intended quantum behavior.

## Output

The output of the script is a string representing the optimized algebraic expression for the quantum circuit. This expression simplifies the circuit by combining or eliminating redundant operations, making it easier to analyze the circuit's behavior.

### Example Output

After processing, the script produces an optimized algebraic expression as follows:

```
final simplified version: (I D0 I + Z D1 I)
```



## Core Components

The script has several core components:

### Parsing the Input
- **Process**: It starts by reading and parsing the `input.txt` file to extract quantum circuit information such as the number of qubits, and the specific gates applied, including their control and target qubits.

### Data Storage Structures
- **control_target_pairs**: A list of dictionaries where each dictionary maps control qubits to their respective target qubits for each segment of the circuit divided by barriers.
- **target_operations**: A list of dictionaries that map each target qubit to its corresponding operation for each segment.
- **gate**: A list of dictionaries to store non-controlled gates for each segment.

### Segmentation
- **Process**: The script divides the circuit into segments using `barrier` as a delimiter, which helps in localized optimization within those segments.

### Expression Construction
- **Process**: For each segment, it generates possible states of control qubits and computes the resulting operations on target qubits. These are then formatted into an algebraic expression.

### Expression Simplification
- **Process**: The script simplifies the constructed expression using algebraic rules specific to quantum operations (e.g., X.X = I where X is the Pauli-X gate and I is the identity operation). This simplification process involves applying distributive laws and reducing expressions using predefined quantum gate properties.

### Output Generation
- **Process**: Finally, the script outputs the fully simplified algebraic expression that represents the quantum circuit.

## Functions

- **is_control_for_any(qubit, control_target_pairs)**:
  - This function checks whether a specific qubit acts as a control qubit in any pair within the provided segment.
  - Parameters:
    - `qubit`: The qubit to be checked.
    - `control_target_pairs`: A list of dictionaries representing control-target pairs for each segment.
  - Returns:
    - `True` if the qubit acts as a control qubit for any target in any pair, otherwise `False`.

- **create_term(state, control_target_pairs, target_operations, not_control, gate)**:
  - This function generates a term in the algebraic expression based on the state of control qubits and the operations defined in the segment.
  - Parameters:
    - `state`: The binary state representing the activation of control qubits.
    - `control_target_pairs`: A dictionary containing control-target pairs for the current segment.
    - `target_operations`: A dictionary containing target operations for each target qubit in the segment.
    - `not_control`: A boolean list indicating whether each qubit is a control qubit.
    - `gate`: A dictionary containing gate operations for each qubit in the segment.
  - Returns:
    - A string representing the term in the algebraic expression.

- **multiply_segments(segment1, segment2)**:
  - This function combines two segments of the algebraic expression.
  - Parameters:
    - `segment1`: The first segment of the expression.
    - `segment2`: The second segment of the expression.
  - Returns:
    - A string representing the combined segments.

- **apply_distributive_law(expression)**:
  - This function applies the distributive law to expand the expression, simplifying it further.
  - Parameters:
    - `expression`: The algebraic expression to which the distributive law will be applied.
  - Returns:
    - A string representing the expanded expression after applying the distributive law.

- **apply_rules(expression, rules)**:
  - This function applies a set of predefined rules to further simplify the expression.
  - Parameters:
    - `expression`: The algebraic expression to which the rules will be applied.
    - `rules`: A dictionary containing predefined rules for simplification.
  - Returns:
    - A string representing the simplified expression after applying the rules.

- **simplify_identity(expression)**:
  - This function reduces expressions involving the identity operation.
  - Parameters:
    - `expression`: The algebraic expression to be simplified.
  - Returns:
    - A string representing the simplified expression.

- **simplify_expressions(expressions)**:
  - This function applies simplifications across a list of expressions.
  - Parameters:
    - `expressions`: A list of algebraic expressions to be simplified.
  - Returns:
    - A list of strings representing the simplified expressions.


## Conclusion

This script is a tool for transforming a quantum circuit described in OpenQASM into a simplified algebraic expression. It facilitates easier analysis and optimization of quantum circuits by abstracting the operations into a mathematical format. The output can be used for further theoretical studies or as input to other optimization tools. The script ensures clarity and accuracy in handling complex quantum computing concepts, making it accessible for users familiar with quantum programming.

## Setup and Usage Guide for `quantumalgosim.py`

### Prerequisites
Before running the script, ensure you have the following installed:
- **Python 3.6 or higher**: Download from [python.org](https://www.python.org/downloads/).
- **Python Libraries**: Install any required libraries using pip (if needed, e.g., numpy for computations)
### Running the Script
Follow these steps to run the script:

#### Open Terminal or Command Prompt:
- Navigate to the directory containing the script and the `input.txt` file.

#### Execute the Script:
- Run the script by typing the following command:
  ```bash
  python quantumalgosim.py

