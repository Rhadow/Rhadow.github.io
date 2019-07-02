---
layout     : post
title      : "Quantum Kata (2) - Superposition"
subtitle   : "My solution to second set of quantum kata - superposition"
date       : 2019-07-01
author     : "Rhadow"
tags       : QuantumComputing
comments   : true
signature  : true
---

Superposition is one of the special properties of quantum computer that makes it powerful. I think of it as having different potential states of qubits happening at the same time which allows calculations of different states in one go. This is what makes quantum computer much faster than classical computer.

In this section, we will be challenged to manipulate qubits in superposition into various qubit states.

### Task 1. Plus state

> Input: a qubit in \\(\left\| 0 \right>\\) state
>
> Goal: create a \\(\left\|+\right>\\) state on this qubit (\\(\left\|+\right> = \frac{\left\| 0 \right> + \left\| 1 \right>}{\sqrt{2}}\\))

We already seen the same question in [last section](https://rhadow.github.io/2019/06/27/quantum-kata-basic-gates/). This can be achieved by Hadamard gate.

```
operation PlusState (qs : Qubit[]) : Unit {
    H(qs[0]);
}
```

### Task 2. Minus state

> Input: a qubit in \\(\left\| 0 \right>\\) state
>
> Goal: create a \\(\left\|-\right>\\) state on this qubit (\\(\left\|-\right> = \frac{\left\| 0 \right> - \left\| 1 \right>}{\sqrt{2}}\\))

This can be either achieved by first apply X gate then Hadamard gate or first apply Hadamard gate then Z gate.

```
operation MinusState (qs : Qubit[]) : Unit {
    X(qs[0]);
    H(qs[0]);
}
```

### Task 3\*. Unequal superposition

> Input: a qubit in \\(\left\| 0 \right>\\) state and angle \\(\alpha\\)
>
> Goal: create a \\(cos(\alpha)\left\| 0 \right> + sin(\alpha)\left\| 1 \right>\\) state on this qubit.


This kata is a duplicate of Kata 1.4 in [last section](https://rhadow.github.io/2019/06/27/quantum-kata-basic-gates/).

```
operation UnequalSuperposition (qs : Qubit[], alpha : Double) : Unit {
    Ry(2.0 * alpha, qs[0]);
}
```

### Task 4. Superposition of all basis vectors on two qubits

> Input: two qubits in \\(\left\| 00 \right>\\) state
>
> Goal: create the following state on these qubits: \\(\frac{\left\| 00 \right> + \left\| 01 \right> + \left\| 10 \right> + \left\| 11 \right>}{\sqrt{2}}\\)

To create all basis vectors, we apply hadamard gate on each \\(\left\| 0 \right>\\) qubit. This technique is not only limited to two qubits, it is applicable for n qubits. The general equation is:

![Hadamard general equation](https://rhadow.github.io/public/quantum_kata/hadamard_general.png)

```
operation AllBasisVectors_TwoQubits (qs : Qubit[]) : Unit {
    H(qs[0]);
    H(qs[1]);
}
```

### Task 5. Superposition of basis vectors with phases

> Input: two qubits in \\(\left\| 00 \right>\\) state
>
> Goal: create the following state on these qubits: \\(\frac{\left\| 00 \right> + i\left\| 01 \right> - \left\| 10 \right> - i\left\| 11 \right>}{2}\\)

As this kata hinted, it will be easier if we separate two qubits and work on each qubit. A general qubit equation is \\(a\left\| 0 \right> + b\left\| 1 \right>\\). We first have two general qubits and solve the amplitude (a, b, c, d) as follows.

$$
(a\left| 0 \right> + b\left| 1 \right>)\otimes(c\left| 0 \right> + d\left| 1 \right>) =
$$

$$
ac\left| 00 \right> + ad\left| 01 \right> + bc\left| 10 \right> + bd\left| 11 \right> = 
$$

$$
\frac{\left| 00 \right> + i\left| 01 \right> - \left| 10 \right> - i\left| 11 \right>}{2}
$$

Here we can see that:

$$
ac = \frac{1}{2}
$$

$$
ad = \frac{i}{2}
$$

$$
bc = \frac{-1}{2}
$$

$$
bd = \frac{-i}{2}
$$

Solve for a, b, c and d we will get:

$$
a = \frac{1}{\sqrt{2}}
$$

$$
b = \frac{-1}{\sqrt{2}}
$$

$$
c = \frac{1}{\sqrt{2}}
$$

$$
d = \frac{i}{\sqrt{2}}
$$

Which means to create the goal state, we need these two qubits:

$$
\frac{\left| 0 \right> - \left| 1 \right>}{\sqrt{2}}
$$

$$
\frac{\left| 0 \right> + i\left| 1 \right>}{\sqrt{2}}
$$

Now this kata can be reduced to transform one \\(\left\| 0 \right>\\) to \\(\frac{\left\| 0 \right> - \left\| 1 \right>}{\sqrt{2}}\\) and another \\(\left\| 0 \right>\\) to \\(\frac{\left\| 0 \right> + i\left\| 1 \right>}{\sqrt{2}}\\).

Let's look at the first one, it can be achieved by first applying Hadamard gate then Z gate. For the second one, a Hadamard gate then S gate will give us the correct state. All we left to do is translate them to code:

```
operation AllBasisVectorsWithPhases_TwoQubits (qs : Qubit[]) : Unit {
    H(qs[0]);
    Z(qs[0]);
    H(qs[1]);
    S(qs[1]);
}
```

### Task 6. Bell state

> Input: two qubits in \\(\left\| 00 \right>\\) state
>
> Goal: create a Bell state \\(\|Φ⁺⟩ = \frac{\left\| 00 \right> + \left\| 11 \right>}{\sqrt{2}}\\)

There is detailed instructions on how to create this bell state in [Quantum Quest 2.2.5](https://www.quantum-quest.nl/quantumquest.pdf). Only Hadamard gate and CNOT gate is needed.

```
operation BellState (qs : Qubit[]) : Unit {
    H(qs[0]);
    CNOT(qs[0], qs[1]);
}
```

### Task 7. Bell state

> Inputs: two qubits in \\(\left\| 00 \right>\\) state and an Integer index
>
> Goal: create one of the Bell states based on the value of index:
>
> 0: \\(\|Φ⁺⟩ = \frac{\left\| 00 \right> + \left\| 11 \right>}{\sqrt{2}}\\)
>
> 1: \\(\|Φ⁻⟩ = \frac{\left\| 00 \right> - \left\| 11 \right>}{\sqrt{2}}\\)
>
> 2: \\(\|Ψ⁺⟩ = \frac{\left\| 01 \right> + \left\| 10 \right>}{\sqrt{2}}\\)
>
> 3: \\(\|Ψ⁻⟩ = \frac{\left\| 01 \right> - \left\| 10 \right>}{\sqrt{2}}\\)

Again, another duplicate kata from [last section](https://rhadow.github.io/2019/06/27/quantum-kata-basic-gates/). There are two ways of creating Bell states, we can either transform the \\(\left\| 00 \right>\\) to \\(\|Φ⁺⟩\\) first and then convert it to the rest of states or change the state to \\(\left\| 01 \right>\\), \\(\left\| 10 \right>\\), \\(\left\| 11 \right>\\) then do the Bell transform.

$$
left\|Φ⁺\right> = BellTransform(\frac{\left\| 00 \right>)
left\|Φ⁻\right> = BellTransform(\frac{\left\| 10 \right>)
left\|Ψ⁺\right> = BellTransform(\frac{\left\| 01 \right>)
left\|Ψ⁻\right> = BellTransform(\frac{\left\| 11 \right>)
$$

```
operation AllBellStates (qs : Qubit[], index : Int) : Unit {
    // Bell Transform first
    H(qs[0]);
    CNOT(qs[0], qs[1]);
    if (index == 1) {
        Z(qs[0]);
    } elif (index == 2) {
        X(qs[0]);
    } elif (index == 3) {
        X(qs[0]);
        Z(qs[0]);
    }

    // Bell Transform last
    // if (index == 1) {
    //     X(qs[0]);
    // } elif (index == 2) {
    //     X(qs[1]);
    // } elif (index == 3) {
    //     X(qs[0]);
    //     X(qs[1]);
    // }
    // H(qs[0]);
    // CNOT(qs[0], qs[1]);

}
```
