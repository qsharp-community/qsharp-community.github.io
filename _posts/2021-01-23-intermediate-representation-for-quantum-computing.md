---
title: "Intermediate representation for quantum computing"
date: 2021-01-23
categories:
  - blog
---

# Intermediate representation for quantum computing

As a quantum computing physicist, at some point you might have wondered which language to use for programming and testing quantum algorithms, especially taking into account that the wide variety of quantum-programming languages is growing every day. This expansion naturally enriches the quantum programming environment and fosters the understanding and designing of quantum algorithms, but it also raises some questions. For instance, you might consider not only which language is the most appealing to you, but also which is best suited to be translated into instructions that an actual quantum platform can implement. For that, one should consider which compilers and intermediate representations work with these languages. And before we go on, I know, the words compiler or intermediate representation might sound scary if you have a physics background, like me, rather than a computer science one. But at the same time, I believe it is exciting to dive deeper into the programming of a quantum algorithm and come a step closer to what is, at the end, the final goal of any algorithm researcher: to run it on a quantum platform.

An intermediate representation is – as its name indicates – an intermediary step in the workflow between the source code and the hardware itself. It should allow you to represent different kinds of source code before the specific platform is specified. Moreover, at this intermediate level it is also possible to perform some optimization and circuit rearrangement that makes the final implementation more efficient. Some examples of intermediate representations are [OpenQASM](https://github.com/QISKit/openqasm) (used by IBM for Qiskit) or [Quil](https://github.com/rigetti/quil) (used by Rigetti) and of course the topic of this blog entry: the Quantum Intermediate Representation (QIR) by Microsoft.

# What is QIR?

QIR is based on LLVM, which is a classical, well-spread, open-source intermediate representation. QIR is hardware-agnostic, which means that there are no specifications regarding the quantum platform, and even if now it is mainly used for representing Q# programs, it has the potential to represent any other programming language such as Qiskit.
 
The fact that LLVM is the underlying language means that QIR is naturally able to process both classical and quantum logic. This feature is essential for the so-called hybrid algorithms, which have experienced an incredible boost in the last years. This kind of algorithms include both quantum and classical operations that influence each other. This is the case, for instance, of variational algorithms, where a parameter-dependent quantum circuit is run, followed by a classical post-processing and optimization part, which is later used to update the circuit in the quantum device. Part of the reason why these algorithms have generated such a great interest lately is their ability to tackle optimization problems – take for instance the Quantum Approximate Optimization Algorithm (QAOA), which can be used for solving graph problems such as Max-Cut, among other applications – or even quantum chemistry problems – for example, by using the Variational Quantum Eigensolver to approximate a Hamiltonian’s spectrum.

# Why is QIR interesting?
 
We’ve already mentioned that QIR is an essential tool when running quantum algorithms on real hardware. But even if we do not want to go that far and are just developing algorithms at a more theoretical level, intermediate representations can play an important role! 

First, there are optimization steps that can be performed at the intermediate level that can make the overall implementation more efficient. Investigating this optimisation of your input code can help to get a better understanding of where we can make algorithms more efficient and how to improve the quantum programming languages. 

Moreover, you could use the intermediate representation to generate code that is later on given as input into a quantum simulator -- instead of a real device -- which could potentially use a different language than your source one. IThis way, we can easily compare and benchmark different languages and simulators using a common framework.

# References and further reading

Don’t forget to check the [blog entry](https://devblogs.microsoft.com/qsharp/introducing-quantum-intermediate-representation-qir/) where QIR was introduced! It also includes code examples of how QIR code would look like. If you want to deep further you can visit the [QIR branch](https://github.com/microsoft/qsharp-compiler/tree/feature/qir), which belongs to the [Q# compiler repository](https://github.com/microsoft/qsharp-compiler), and the [QIR specifications page](https://github.com/microsoft/qsharp-language/tree/main/Specifications/QIR).

You can see a list of QIR-related projects [here](https://github.com/microsoft/qsharp-language/blob/main/Specifications/QIR/List.md), including [QCOR](https://qcor.ornl.gov/), which provides a nice overview of how the workflow looks like in a quantum program and how its different layers are interconnected.


# About this post and the authors

This blog entry is written in the context of a project carried out under the [QOSF mentorship program](https://qosf.org/qc_mentorship/), where Dr. Sarah Kaiser and me, Esther Cruz Rico, are investigating the potential of QIR. This is an ongoing project and you can check it in [this repository](https://github.com/esthercruz/qosf_mentorship_project). 
