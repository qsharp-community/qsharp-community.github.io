---
title: "Building a Q# library to find table minima: Part 1"
date: 2020-06-22
categories:
  - Blog
tags:
  - library
  - algorithm
  - in-development
---
## Introduction

Hello all! 
My name is Mridul Sarkar, and I am an undergraduate at University of California Davis studying Scientific and Mathematical Computation.
I wanted to share with the community my experiences in learning about the Dürr–Høyer algorithim and creating a library for Q# implementing it.
This project has pushed me and is still pushing me outside of my comfort zone, but learning Q# while slowly refining my quantum mechanics has been a rewarding experience. 
I would like to pass on what I have seen and understood in that time, as well as ask the community for feedback. 
I will likely be posting about this algorithm and library more as the library and my understanding continues developing.

### My Background
I began my journey from math to computers through my passion for ML algorithms, where I stumbled upon quantum computing. 
I was instantly pulled in by the elegant algorithms and proposed efficiency.
Recently, I started looking into quantum machine learning where you can model nodes of neural networks as qubits. 
I took a deep dive into the research papers and found myself amazed by what was out there.
I began with a Coursera course from St. Petersburg University on quantum computing with detail on quantum algorithm design and quantum computer architecture. 
From here I tinkered with Q# and developed basic algorithms that I previously learned.
I found Dürr and Høyer's paper and saw it as an fusion of Duestch's, Shor's, and Grover's Algorithm; a perfect next step for me.


### Motivation
The motivation behind this project is to provide open source functionality. The efficiency that is proposed by this algorithm is Big-O = square root of the number of items in the table. Durr and Hoyer propose an ensemble of quantum algorithms in order to find the minimum of an unsorted table with a 50% success rate. This success rate is bounded by the Big-O. If we decide to alleviate this Big-O the probability of finding the minimum increases. With this comes some complexity in the inner workings of the library which will be explained when analyzing the algorithm. In order for this library to be used properly it must meet some guidelines. At the moment I am referencing the amazing template given by Dr. Sarah Kaiser https://github.com/crazy4pi314/qsharp-library-template in order to make this library usable in Q#. I am programming the algorithm in Q# with a python shell. The python shell will be provided below. The circuit's gates will be analyzed and some code is given.


### The Algorithm
A brief introduction to the Durr and Hoyer Algorithm can be derived from the Quantum Minimum Searching Algorithm outlined in 'A quantum algorithm for finding the minimum' [(1)]:

Durr and Hoyer propose a computationally efficient routine to find the minimum of a table of distinct unsorted integers. The algorithm is as follows:


![QMSA](/assets/images//DurrHoyer-QMSA.JPG "QMSA")


#### Psuedocode

```
    t = table.flatten()
    N = len(t)
    y = randomuniform[1..N] #N is a integer
    time_limit = (22.5*sqrt(N)=1.4*log(N)^2)
    q = preprocess.simulate(t,y)

    while time.clock() < time_limit:
        for i in range(N):
            while t[i] < t[y]:
                i = N-i
                if N is even
                    q=Algorithm with Hadamard gates[0...j]
                    y'=q[0]
                else
                    t=Algorithm with QFT instead of H[0...j]
                    y'=q[0]
        if t[y'] < t[y]:
            y = y'
            return
    return y
```
------------------

It is easy to understand the conceptual implementation in python. It is important to note steps a and b:

------------------

![a](/assets/images//a.JPG)   

Part a is probably the most interesting part of this algorithm from my perspective. After a brief chat with Dr. Hoyer I realized this aspect of the algorithm is a field of research in itself. The creation of qubits from a table of unique integers is an interesting question.

For now, the more prevalent question is what sort of initialization we can use to verify the integrity and efficiency of this algorithm. My most immediate thought is to entangle all the qubits and superimpose them. The classical value of the integer is not represented here, only the index of the integer is captured at a quantum level on the table assuming an input datatype of BigEndian. If the goal is to test the algorithm's probabilistic integrity this plan of action will suffice.

The register can be initiated as follows:

![latex_imp](/assets/images//latex_information.JPG)

Obviously this is unachievable on current quantum computer architecture. We only have a qubit, the above example implies we have harnessed the tritbit and so on till n bit. In reality we will register all integers as unique binary strings which will equate to their qubits.
 
Luckily PrepareUniformSuperposition() in Q# does this for us easily. I have implemented it as follows:

![intialize](/assets/images//intialize.JPG)

Note: make sure to use LittleEndian() on the register of qubits before applying PrepareUniformSuperposition(). From here you will have to use LittleEndianAsBigEndian() to set the right data type after using PrepareUniformSuperposition() as it spits out LE datatype. I didn't include this so it looked neat.

In order to mark items we simply mark all indices that come before i.

------------------

![b](/assets/images//b.JPG)   

Part b must be understood through another paper 'Tight bounds on quantum searching' [(2)]. It details the use of the quantum exponential searching algorithm in three cases. Those three cases being: finding one solution, multiple solutions, or an unknown number of solutions. For each case there is different preparation of qubits and algorithms used in unison to find the solution(s). The Durr Hoyer Algorithm is supposed to push computational efficiency versus computational accuracy. The statement earlier mentioning the 50% success rate for finding the minimum is bounded by the Big-O can be explored further now. [3]

It is intuitive to see that in the case for finding a minimum of an unsorted table is finding one unique solution. The details are mentioned under "Implementation Considerations".

![Implementation](/assets/images//DurrHoyer-Implementation.JPG "Implementation")

A thorough read of this will supply any clarification to the python code posted above. 'G' mentioned above is the 'Algorithm' in the python psuedo code. It will take in the random value, the table in quantum information, and the lenght of the table. It is important to note that if the table has odd elements a different form of the algorithm must be used. Being new to quantum computing I have found myself wading through darkness at times. Reading this even now leaves me a bit unsure and skeptical of my own implementation, redoing my calculations. After research and conversations with quantum computing aficionados I have reached the following conclusions:

1. T is just a Hadamard Gate
2. Conditional Phase Shift on A is a variation of the CNOT Gate
3. Conditional Phase Shift on 0 will will change the sign of the qubit if it is in null space, otherwise it will give the same result back

Here is a small snippet of my current circuit:

![circuit](/assets/images//Algorithm_Even.JPG)

Generalizations are a mathematicians greatest friend, but a downfall if not implemented correctly. If anyone has any input for the generalizations I have made I will take them with open arms!

------------------

### Conclusion

So far I have been greatly enjoying throwing myself into this new and exciting world of quantum computing. I have seen some awesome principles translate across quantum algorithms that are keeping me engaged. In specific, I mentioned I felt Durr and Hoyer used principles from Duestch's, Grover's, and Shor's algorithm. Duestsch's Algorithm famously simplifies a classical problem into an non intuitive oracle function as 'G' does. Grover's Algorithm is quite literally applied in this algorithm. Lastly, Shor's Algorithm shows off the power of combining classical and quantum systems to achieve outstanding results. I love seeing such fundamental concepts continue to push boundaries. I am constantly in search for collaboration among those who are equally as passionate about quantum computing. If you had any suggestions or questions feel free to send me an email.

https://github.com/mridulsar/DurrHoyerLibrary   
------------------

## My current challenges in this project:   

1. Validating generalizations I made of 'Implementation Considerations'    

2. Developing alleviation for Big-O in a fluid manner.   

------------------

[(1)]:https://arxiv.org/pdf/quant-ph/9607014.pdf
[(2)]:https://arxiv.org/pdf/quant-ph/9605034.pdf
[3]:    Essentially, the number of times we call upon the algorithm in step (b) the greater our Big-0 and probabilistic success rate. Durr Hoyer answers to a 'sweet spot' of sorts. The algorithm will run for the allotted while loop in the python code above to achieve a 50% success rate. This proof is done in detail in [(1)]. An important feature of this open source library will be the user's manipulation of how many times step (b) is applied. In addition, future implementation can include manipulation of the algorithm in order to implement the different 3 cases mentioned for the quantum exponential searching algorithm. Though this means the algorithm is technically no longer Durr Hoyer it will be an exciting feature.
