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

## TL;DR
---

    The Dürr-Høyer algorithm takes an unsorted table of integers    
    and finds the minima in Big-O(sqrt(N)) N being the # of elements in the table.         
    Open Source Implementation of the algorithm is easily done, understanding the Dürr and Høyer paper [(1)]    
    was and is the most difficult challenge.   
    There is a python host file and two Q# circuits.   
    One circuit applies the Quantum Minimum Searching Algorithm and
    another circuit intializes our register as described in 'A quantum algorithm for finding the minimum'[(1)]    
    The most intersting part of the algorithm is the initializing of the register   
    and applying the quantum exponential searching algorithm [(2)] to the register.

    If you would like to contribute to this project or take a look at some code head over to:   
    https://github.com/mridulsar/DurrHoyerLibrary 

---

## Introduction

Hello all! 
My name is Mridul Sarkar, and I am an undergraduate at University of California Davis studying Scientific and Mathematical Computation.
I wanted to share with the community my experiences in learning about the Dürr–Høyer algorithim and creating a library for Q# implementing it.
This project has pushed me and is still pushing me outside of my comfort zone, but learning Q# while slowly refining my quantum mechanics has been a rewarding experience. 
I would like to pass on what I have seen and understood in that time, as well as ask the community for feedback. 
I will likely be posting about this algorithm and library more as the library and my understanding continues developing.   
https://github.com/mridulsar/DurrHoyerLibrary.     

### My Background
I began my journey from math to computers through my passion for ML algorithms, where I stumbled upon quantum computing. 
I was instantly pulled in by the elegant algorithms and proposed efficiency.
Recently, I started looking into quantum machine learning where you can model nodes of neural networks as qubits. 
I took a deep dive into the research papers and found myself amazed by what was out there.
I began with a Coursera course from St. Petersburg University on quantum computing with detail on quantum algorithm design and quantum computer architecture. 
From here I tinkered with Q# and developed basic algorithms that I previously learned.
I found Dürr and Høyer's paper and saw it as an fusion of Duestch's, Shor's, and Grover's Algorithm; a perfect next step for me.


### Motivation
The motivation behind this project is to provide open source functionality. 
The efficiency that is proposed by this algorithm is much better than a typical algorithm for finding the minimum of an unsorted table.
On a classical computer it will take as many time steps as there are items to find a minimum. 
This means the Big-O is N.
The Dürr-Høyer algorithm takes this problem and solves it in sqrt(N) time steps.
For example assume there are 9 items in a table, on a classical comptuer this would take 9 time steps.
The Dürr-Høyer algorithm would find the minimum in 3 steps.
We can understand this effecieny as the Big-O. 
The Big-O essentially means the run time of the program will increase as the table size grows. 
Dürr and Høyer propose an ensemble of quantum algorithms in order to find the minimum of an unsorted table with a 50% success rate. 
This success rate is bounded by the Big-O.
If we decide to alleviate this Big-O the probability of finding the minimum increases. 
Essentialy if we let the algorithm run for longer than the time it takes to guarantee a 50% success rate, we will get a success rate greater than 50 %. 
With this comes some complexity in the inner workings of the library which will be explained when analyzing the algorithm. 
In order for this library to be used properly it must meet some guidelines. 
At the moment I am referencing the amazing template given by Dr. Sarah Kaiser https://github.com/crazy4pi314/qsharp-library-template in order to make this library usable in Q#. 
I am programming the algorithm in Q# with a python host script.
The circuit's gates will be analyzed and some code is given.  
https://github.com/mridulsar/DurrHoyerLibrary   


## The Dürr–Høyer Algorithm
A brief introduction to the Dürr–Høyer Algorithm can be derived from the Quantum Minimum Searching Algorithm outlined in 'A quantum algorithm for finding the minimum' as summarized below. If you would like to read the paper in all of its glory, there is a link here:[(1)].


1. Choose an integer (y) uniformly at random between 0...N-1, where N = flattened table length   
2. Repeat the following steps until time = 22.5 * sqrt(N) + 1.4 * log^2(N). Once time equals the expression with N proceed to step c   
2(a) Intialize the memory as a uniform superposition of qubits. 
Each qubit represents an integer in the table. 
After Intializing the memory apply the qubit representing y-th index of the flattened table to your register. 
Additionally, mark all elements of the table which are less than than the y-th integer of the table.   
2(b) Apply the quantum exponential searching algorithm [(2)].   
2(c) Measure the first qubit through use of Shor's Algorithm.
Take this hypothetical minima's index as y'.
If the integer in the y'-th position is less than the integer in the y-th position set y as y'.   
3. Return y.

It is important to note steps a and b:   


2(a) Intialize the memory as a uniform superposition of qubits.
Each qubit represents an integer in the table. 
After Intializing the memory apply the qubit representing y-th index of the flattened table to your register. 
Additionally, mark all elements of the table which are less than than the y-th integer of the table.   

Part a is probably the most interesting part of this algorithm from my perspective. 
After a brief chat with Dr. Hoyer I realized this aspect of the algorithm is a field of research in itself. 
The creation of qubits from a table of unique integers is an interesting question.   

For now, the more prevalent question is what sort of initialization we can use to verify the integrity and efficiency of this algorithm. 
My most immediate thought is to entangle all the qubits and superimpose them. 
The classical value of the integer is not represented here, only the index of the integer is captured at a quantum level on the table assuming an input datatype of BigEndian. 
If the goal is to test the algorithm's probabilistic integrity this plan of action will suffice.   

The register can be initiated as follows:   

![latex_imp](/assets/images//latex_information.JPG)   

Obviously this is unachievable on current quantum computer architecture. 
We only have a qubit, the above example implies we have harnessed the tritbit and so on till n bit.
In reality we will register all integers as unique binary strings which will equate to their qubits.   
 
Luckily PrepareUniformSuperposition() in Q# does this for us easily. 
I have implemented it as follows:

---

    operation CreateQuantumInformation( TableLength : Int, RandomIndex : Int ) : Qubit[]
        {
            using (register = Qubit[TableLength])
            {
                PrepareUniformSuperposition(N,LittleEndian(register));
                let register = LittleEndianasBigEndian(register);
                ApplytoAll(i*register[RandomIndex], register!);
            }   
            return aux;
        }
        
---

In order to mark items we run a for loop and compare all values to each other, storing the values that are smaller than the y-th index of our flattened table.

------------------

2(b) Apply the quantum exponential searching algorithm [(2)].  

Part b must be understood through another paper 'Tight bounds on quantum searching' [(2)].
It details the use of the quantum exponential searching algorithm.
The Dürr–Høyer Algorithm is supposed to push computational efficiency versus computational accuracy.
The statement earlier mentioning the 50% success rate for finding the minimum is bounded by the Big-O depends on the number of times the QESA is applied to our quantum information.
Essentially, the number of times we call upon the algorithm in step (b) the greater our Big-0 and probabilistic success rate. The Dürr–Høyer algorithm answers to a 'sweet spot' of sorts. 
The algorithm will run for the allotted time based T = 22.5 * sqrt(N) + 1.4 * log^2(N) to achieve a 50% success rate.
If we do some math and manipulate the expression with N we can achieve better accuracy with a longer run time.
This proof is done in detail in 'A quantum algorithm to find  [(1)]. 
An important feature of this open source library will be the user's manipulation of how many times step 2(b) is applied.


The details of step 2(b) are mentioned under "Implementation Considerations".

![Implementation](/assets/images//DurrHoyer-Implementation.JPG "Implementation")

It is important to note that if the table has odd elements a different form of the algorithm must be used.
Being new to quantum computing I have found myself wading through darkness at times.
Reading this even now leaves me a bit unsure and skeptical of my own implementation, redoing my calculations.
After research and conversations with quantum computing aficionados I have reached the following conclusions:

1. T is just a Hadamard Gate
2. Conditional Phase Shift on A is a variation of the CNOT Gate
3. Conditional Phase Shift on 0 will will change the sign of the qubit if it is in null space, otherwise it will give the same result back

Here is a small snippet of my current circuit:

---

    operation ApplyDürrHøyer( register : BigEndian[] ) : Unit 
    {
        ApplyToEach(H,register!); 
        ApplyConditionalPhase_A(register);
        ApplyToEach(H,register!);
        ApplyConditionalPhase_0(register);
    }
    
---

Generalizations are a mathematicians greatest friend, but a downfall if not implemented correctly.
If anyone has any input for the generalizations I have made I will take them with open arms!

------------------

### Conclusion

So far I have been greatly enjoying throwing myself into this new and exciting world of quantum computing. I have seen some awesome principles translate across quantum algorithms that are keeping me engaged.
In specific, I mentioned I felt Dürr and Høyer used principles from Duestch's, Grover's, and Shor's algorithm.
Duestsch's Algorithm famously simplifies a classical problem into an non intuitive oracle function as 'G' does. Grover's Algorithm is quite literally applied in this algorithm.
Lastly, Shor's Algorithm shows off the power of combining classical and quantum systems to achieve outstanding results.
I love seeing such fundamental concepts continue to push boundaries.
I am constantly in search for collaboration among those who are equally as passionate about quantum computing.
If you had any suggestions or questions feel free to send me an email.

https://github.com/mridulsar/DurrHoyerLibrary   

------------------

## My current challenges in this project:   

1. Validating generalizations I made of 'Implementation Considerations'    

2. Developing alleviation for Big-O in a fluid manner.   

------------------

[(1)]:https://arxiv.org/pdf/quant-ph/9607014.pdf
[(2)]:https://arxiv.org/pdf/quant-ph/9605034.pdf

