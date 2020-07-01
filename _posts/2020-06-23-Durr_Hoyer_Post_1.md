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

    The Dürr-Høyer algorithm takes an unsorted table of integers and 
    finds the minima in Big-O(sqrt(N)) N being the # of elements in the table.          
    There is a python host file and a Q# file.  
    The python file follows the Quantum Minimum Searching Algorithm closely.
    The Q# file follows the Quantum Exponential Searching Algorithm implementation for finding a unique solution.
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

## The Dürr–Høyer Algorithm
A brief introduction to the Dürr–Høyer Algorithm can be derived from the Quantum Minimum Searching Algorithm outlined in 'A quantum algorithm for finding the minimum' as summarized below. If you would like to read the paper in all of its glory, there is a link here:[(1)].


1. Choose an integer (y) uniformly at random between 0...N-1, where N = flattened table length   
2. Repeat the following steps until time = 22.5 * sqrt(N) + 1.4 * log^2(N). The time begins once we start step 2(a), if time equals the expression with N proceed to step c   
2(a) Intialize the memory as a uniform superposition of qubits. 
Each qubit represents an index. 
After intializing the memory grab the y-th qubit and entangle the state of your register with this y-th qubit according the the Oracle given by Grover. 
This marks all T[j]<T[y].   
2(b) Apply the quantum exponential searching algorithm [(2)].
It is a generalized Grovers.   
2(c) Measure the first register.
Take this hypothetical minima's index as y'.
If the integer in the y'-th position is less than the integer in the y-th position set y as y'.   
3. Return y.   

The steps 2(a) and 2(b) pose the biggest challenge if someone has no experience with Quantum Computing. We will first observe how to initalize the register. Then we will see how the QESA can be implemented to find a unique solution, in this case the minimum.

In order to intialize the register we pepare a uniform superposition of qubits, the number of qubits is determined by the number of elements in our table. We then grab our y-th index and entangle it with our register using a Controlled Z.

---

    using ((Register) = (Qubit[TableLength])) // intialize register to number of qubits as there are indices
    {
      within
      {   
        PrepareUniformSuperposition(TableLength,LittleEndian(Register)); // Create Uniform Superposition of all indices
                    
        let Marker = Register[RandomIndex]; // Grab the qubit in the RandomIndex and set it aside

        Controlled Z(Register,Marker); // Apply Oracle to flip all states that are T[j]<T[y]
      }
---

Step 2(a) has been stasified. Now we must figure out how to apply the QESA algorithm. It is stated as follows:  



![implement](/assets/images/DurrHoyer-Implementation.JPG)    



We utilize the register we were working with earlier and apply a T' transform, this is stated to simply be the Hadmard. 

---
    ApplyToEach(H,Register); // Apply Hadamard to register
    
---
We then apply a conditional phase shift if the qubit is 0.    

    ApplyConditionalPhase_0(LittleEndian(Register)); // Reflect qubits that are 0s
    
---

    operation ApplyConditionalPhase_0(register: LittleEndian) : Unit is Adj + Ctl
    {
        using (aux = Qubit()) 
        {
            (ControlledOnInt(0,Z))(register!,aux); // If qubit is 0 flip it!
        }
    }
---

From here we apply the inverse of the Hadmard, this is just the conjugate transpose since Hadamard is unitary. 
This can be implemented be calling the Adjunct of Hadamard.  

---

    ApplyToEachA(H,Register); // Apply Adjunct Hadamard to register
    
---

The last step is another conditional phase shift, though it is applied if the qubit is 1.

---

    ApplyConditionalPhase(LittleEndian(Register)); // Reflect qubits that are 1s
    ---
    operation ApplyConditionalPhase(register : LittleEndian) : Unit is Adj + Ctl 
    {
        using (aux = Qubit()) 
        {
            (ControlledOnInt(1,Z))(register!,aux); // If qubit is 1 flip it!
        }
    }

---

Our Q# script will be structured as follows:

---

    namespace QESA {
        open Microsoft.Quantum.Intrinsic;
        open Microsoft.Quantum.Diagnostics;
        open Microsoft.Quantum.Arrays; 
        open Microsoft.Quantum.Preparation;
        open Microsoft.Quantum.Canon;
        open Microsoft.Quantum.Arithmetic;
        open Microsoft.Quantum.Math;
        open Microsoft.Quantum.Core;
        open Microsoft.Quantum.Convert;
        operation Algorithm_Even(TableLength : Int, RandomIndex : Int) : Unit
        {
            using ((Register) = (Qubit[TableLength])) // intialize register to number of qubits as there are indices
            {
                within
                    {   
                        let Marker = Register[RandomIndex]; // Grab the qubit in the RandomIndex and set it aside

                        PrepareUniformSuperposition(TableLength,LittleEndian(Register)); // Create Uniform Superposition of all indices

                        Controlled Z(Register,Marker); // Apply Oracle to flip all states that are T[j]<T[y]
                    }
                apply 
                    {
                        ApplyToEach(H,Register); // Apply Hadamard to register

                        ApplyConditionalPhase_0(LittleEndian(Register)); // Reflect qubits that are 0s

                        ApplyToEachA(H,Register); // Apply Adjunct Hadamard to register

                        ApplyConditionalPhase(LittleEndian(Register)); //Reflect qubits that are 1s
                    }
            }
        }
        
        operation ApplyConditionalPhase_0(register: LittleEndian) : Unit is Adj + Ctl
        {
            using (aux = Qubit()) 
            {
                (ControlledOnInt(0,Z))(register!,aux); // If qubit is 0 flip it!
            }
        }
        operation ApplyConditionalPhase(register : LittleEndian) : Unit is Adj + Ctl 
        {
            using (aux = Qubit()) 
            {
                (ControlledOnInt(1,Z))(register!,aux); // If qubit is 1 flip it!
            }
        }
    }

    
---

For futher information on how this was derived take a look at 'Tight bounds on quantum searching' [2].
It is important to not the above algorithm only works for a table with even entries. 
The algorithm breaks down when applying the Hadamard gate as the Hadamard is layed across the diagnoal of a identity matrix which is equal in dimensions to the number of qubits we have. 
With a bit of math, if we try to lay a 2x2 matrix along an odd dimensioned identity matrix the transformation is not retained.
To circumvent this we introduce the following implementation, utilizing QFT.

        operation Algorithm_Odd(TableLength : Int, RandomIndex : Int) : Unit
        {
            using ((Register) = (Qubit[TableLength])) // intialize register to number of qubits as there are indices
            {
                within
                    {   
                        let Marker = Register[RandomIndex]; // Grab the qubit in the RandomIndex and set it aside

                        PrepareUniformSuperposition(TableLength,LittleEndian(Register)); // Create Uniform Superposition of all indices

                        Controlled Z(Register,Marker); // Apply Oracle to flip all states that are T[j]<T[y]
                    }
                apply 
                    {
                        QFTLE(LittleEndian(register)); // Implemntation for odd number of table entries

                        ApplyConditionalPhase_0(LittleEndian(Register)); // Reflect qubits that are 0s

                        QFT(BigEndian(register)); // Inverse QFT by using BigEndian

                        ApplyConditionalPhase(LittleEndian(Register)); //Reflect qubits that are 1s
                    }
            }
        }

Now we refer back to our QMSA outline to observe that the Algorithm(TableLenght,RandomIndex) is iterated on until we find a suitable y' or we simply hit our time limit. The true stars of this algorithm are the time limit, which guarantees O(sqrt(N)), the generalized Grovers algorithm given in QESA, which provides for easy implementation and has O(1) for each iteration, and lastly, the oracle function which marks our states, which along with our intialization of qubits has O(log(n)).

Here is the python host that will apply the conditions of QMSA while using QESA.

---

    class DH(object):

      def __init__(self,table):

          self.table = np.ndarray.flatten(table) # Flatten table

          self.N= len(self.table) # Number of elements in the table

          self.y = int(np.random.uniform(0,self.N-1)) # Choose our random index

      def QMSA(self,N,y,table):
      
          N = self.N

          y = self.y

          t = self.table

          y_prime = N

          time_limit = (22.5*np.sqrt(N)+1.4*np.log(N)**2) # Time limit as specified in QMSA

          while time.clock() < time_limit:    

              for i in range(N):

                  while t[y] < t[y_prime]:

                      if N // 2 : # Implementation of odd and even functionality

                          y_prime= Algorithm.simulate(N,y)

                      else: 

                          y_prime=Algorithm_Odd.simulate(N,y)

                  if t[y_prime] < t[y]: # Check if we have found our minimum

                      i=0 # reset loop if found

              y = y_prime #store index with minimum

          return y                                                      


------------------

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
Dürr and Høyer propose a quantum algorithms in order to find the minimum of an unsorted table with a 50% success rate. 
This success rate is bounded by the Big-O.
If we decide to alleviate this Big-O the probability of finding the minimum increases. 
Essentialy if we let the algorithm run for longer than the time it takes to guarantee a 50% success rate, we will get a success rate greater than 50 %. 
With this comes some complexity in the inner workings of the library which will be explained when analyzing the algorithm. 
In order for this library to be used properly it must meet some guidelines. 
At the moment I am referencing the amazing template given by Dr. Sarah Kaiser https://github.com/crazy4pi314/qsharp-library-template in order to make this library usable in Q#. 
I am programming the algorithm in Q# with a python host script.
The circuit's gates will be analyzed and some code is given.  
https://github.com/mridulsar/DurrHoyerLibrary   

### My Background
I began my journey from math to computers through my passion for ML algorithms, where I stumbled upon quantum computing. 
I was instantly pulled in by the elegant algorithms and proposed efficiency.
Recently, I started looking into quantum machine learning where you can model nodes of neural networks as qubits. 
I took a deep dive into the research papers and found myself amazed by what was out there.
I began with a Coursera course from St. Petersburg University on quantum computing with detail on quantum algorithm design and quantum computer architecture. 
From here I tinkered with Q# and developed basic algorithms that I previously learned.
I found Dürr and Høyer's paper and saw it as an fusion of Duestch's, Shor's, and Grover's Algorithm; a perfect next step for me.

### Conclusion

So far I have been greatly enjoying throwing myself into this new and exciting world of quantum computing. 
I have seen some awesome principles translate across quantum algorithms that are keeping me engaged.
In specific, I mentioned I felt Dürr and Høyer used principles from Duestch's, Grover's, and Shor's algorithm.
Duestsch's Algorithm famously simplifies a classical problem into an non intuitive oracle function as done in Dürr and Høyer's algorithm to mark all states that satisfy T[j]<T[y]. 
Grover's Algorithm is quite literally applied in this algorithm, though a generalized version is used.
Lastly, Shor's Algorithm shows off the power of combining classical and quantum systems to achieve outstanding results, which is shown in Dürr and Høyer's algorithm by bounding our time, a classical step in the algorithm.
I love seeing such fundamental concepts continue to push boundaries.  

This algorithm should be available to use within Q# within the next few weeks once testing is done to ensure usability. 
QESA is really the algorithm of interest, to get a running version of QMSA is just to show that this algorithm does indeed work. 
What is more interesting is creating variations of QMSA to fit your needs while utilizing QESA. 
It is important to note that currently QESA can only solve for one unique solution. 
In the future, QESA will be more robust and multiple solutions can be found.
In the mean time you can implement this code for yourself, I would love to see what this community can do with an algorithm like this.
If you had any suggestions or questions feel free to send me an email.

https://github.com/mridulsar/DurrHoyerLibrary   

------------------

## My current challenges in this project:   

1. Implementing search for multiple solutions.    

2. Developing alleviation for Big-O in a fluid manner.   

------------------

[(1)]:https://arxiv.org/pdf/quant-ph/9607014.pdf
[(2)]:https://arxiv.org/pdf/quant-ph/9605034.pdf

