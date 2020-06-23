---
title: "In Progress Durr Hoyer Algorithm"
author "mridulsar"
categories:
  - Blog
tags:
  - qsharp
  - algorithm
  - open source
---
# Durr Hoyer

Hello all! My name is Mridul Sarkar. I am an undergraduate at University of California Davis studying Scientific and Mathematical Computation. I began my journey from math to computers through my passion for ML algorithms. Eventually I stumbled upon quantum computing. I was instantly pulled in by the elegant algorithms and proposed effeciency. 
------------------


## Background
Recently I began working on an open source Durr Hoyer library. This project has pushed me and is still pushing me outside of my comfort zone. Learning Q# while slowly refining my quantum mechanics has been a rewarding experience. I would like to pass on what I have learnt in that time while looking for feedback. I will likely be posting about this algorithm more as it continues developing.
------------------

## Motivation
The motivation behind this project is to provide open source functionality. The effeciency that is proposed by this algorithm is Big-O = square root of the number of items in the table. Durr and Hoyer proposes an ensemble of quantum algorithms in order to find the minimum of an unsorted table with a 50% success rate. This success rate is bounded by the Big-O. If we decide to alleviate this Big-O, which I will explain in detail later, the probability of finding the minimum increases. 
------------------

## The Algorithm
A brief introduction to the Durr and Hoyer Algorithm can be derived from the Quantum Minimum Searching Algorithm outlined in 'A quantum algorithm for finding the minimum' [(1)]: 

Durr and Hoyer propose a computationally effecient routine to find the minimum of a table of distinct unsorted integers. The algorithm is as follow:

![alt text](QMSA.JPG)
------------------

### Psuedocode

```
    t = table.flatten()
    N = len(t)
    y = randomuniform[1..N] #N is a integer
    time_limit = (22.5*sqrt(N)=1.4*log(N)^2)
    qq = preprocess.simulate(t,y')

    while time.clock() < time_limit:
        for i in range(N):
            while t[i] < t[y]:
                j = N-i
                if N is even
                    Grovers Algorithm for one step across [0...j]
                    q=Algorithm with T[0...j] defined operator
                    y'=q[0]
                else
                    Grovers Algorithm for one step across [0...j]
                    t=Algorithm with QFT instead of T[0...j]
                    y'=q[0]
        if t[y'] < t[y]:
            y = y'
            return
    return y
```
------------------

It is easy to undestand the conceptual implementation in python. It is important to note steps:
## (a..b)

------------------

### (a)
Part a is probably the most interesting part of this algorithm from my perspective. After a brief chat with Dr. Hoyer I realized this aspect of the algorithm is a field of research in itself. The creation of qubits from a table of unique integers is an intersting question. An even more interesting question is if we create a function to map values onto integers and must ponder how to create a state of qubits to capture this classical data. 

For now, the more prevelant question is what sort of intializiation we can use to verify the integrity and effeciency of this algorithm. My most immediate thought is entangle all the qubits and set them as unique basis. The classical value of the integer is not represented here, only the index of the integer is caputred at a quantum level on the table assuming an input dataytpe of BigEndian. If the goal is to test the algorithm's probablistic integrity this plan of action will suffice.

We do know that the register must be initiated as j divided by the square root of N (length of flattened table) along with a qubit for each integer and a redudant qubit, specifically being the randomly chosen y.

This still needs to be properely implemented as I have been wrestling with which route to take. Currently the place holder for this eventual quantum information algorithm is #preprocess.simulate() Suggestions are welcome and appreciated.

In order to mark the indeces as the algorithm states we must run the intialized qubits through grovers algorithm once. 

------------------

### (b)
Part b must be understood through another paper 'Tight bounds on quantum searching' [(2)]. It details the use of the quantum exponential searching algorithm in three cases. Those three cases being: finding one solution, multiple solutions, or an unknown number of solutions. For each case there is different prepration of qubits and algorithms used in unison to find the solution(s). The Durr Hoyer Algorithm is supposed to push the computational effeciency versus computational accuracy. The statement earlier mentioning the 50% success rate for finding the minimum is bounded by the Big-O can be explored further now. [3]

It is intuituve to see that in the case for finding a minimum of an unsorted table is finding one unique solution. The details are mentioned under "Implementation Considerations".

![alt text](implementation.JPG)

A thorough read of this will supply any calirfication to the python code posted above. It is important to note that if the table has odd elements a different form of the algorithm must be used. Being new to quantum computing I have found myself wading through darkness at times. Reading this even now leaves me a bit unsure and skeptical of my own implementation. After research and conversations with quantum computing affecionados I have reached the following conclusions:

1. T is just a Hadamard Gate
2. S_A is just a CNOT Gate
3. S_0 will will change the sign of the qubit if it is in null space, otherwise it will give the same result back

Generalizations are a mathematicians greatest friend, but a downfall if not implemented correctly. If anyone has any input for the generalizations I have made I will take them with open arms!

------------------

# Conclusion

So far I have been greatly enjoying throwing myself into this new and exciting world of quantum computing. I am constantly in search for collaboration among those who are equally as passionate about quantum computing. If you had any suggestions or questions feel free to send me an email at msarkar at ucdavis dot edu. 

------------------

### My current challenges in this project:

1. Creating a C# test script/any test script for my circuit

2. Validating generalizations I made of 'Implementation Considerations'

3. Containerizing this project

------------------

[(1)]:https://arxiv.org/pdf/quant-ph/9607014.pdf
[(2)]:https://arxiv.org/pdf/quant-ph/9605034.pdf
[3]:    Essentialy, the number of times we call upon the algorithm in step (b) the greater our Big-0 and probabilistic success rate. Durr Hoyer answers to a 'sweet spot' of sorts. The algoirthm will run for the alloted while loop in the python code above to achieve a 50% success rate. This proof is done in detail in [(1)]. An important feature of this open source library will be the user's manipulation of how many times step (b) is applied. In addition, future implementation can include manipulation of the algorithm in order to implement the different 3 cases mentioned for the quantum exponential searching algorithm. Though this means the algorithm is technically no longer Durr Hoyer it will be an exciting feature.


