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

> TL;DR
> The Dürr-Høyer algorithm takes an unsorted table of integers of length N and finds the minima in O(sqrt(N)).
> I started a project that implements this algorithm here : [https://github.com/mridulsar/DurrHoyerLibrary](https://github.com/mridulsar/DurrHoyerLibrary), contributions welcome!

## Introduction

Hello all! 
My name is Mridul Sarkar, and I am an undergraduate at University of California Davis studying Mathematics and Scientific Computation.
I wanted to share with the community my experiences in learning about the Dürr–Høyer algorithim and creating a library for Q# implementing it.
This project has pushed me and is still pushing me outside of my comfort zone, but learning Q# while slowly refining my quantum intution has been a rewarding experience. 
I would like to pass on what I have seen and understood in that time, as well as ask the community for feedback. 
I will likely be posting about this algorithm and library more as the library and my understanding continues developing.   
https://github.com/mridulsar/DurrHoyerLibrary.     

## The Dürr–Høyer Algorithm
Say we have a table with N unsorted items where you want o find the minimul value stored in this table. The Dürr–Høyer Algorithm helps us solve this problem. It was originally proposed in "A quantum algorithm for finding the minimum" [(1)], where it was called the 'Quantum Minimum Searching Algorithm'.
I'll summarize the main ideas below in case you don't have time to read the paper in all of its glory.

### QMSA
1. Choose an integer (y) uniformly at random between 0...N-1, where N = flattened table length   
2. Repeat the following steps until time = 22.5 * sqrt(N) + 1.4 * log^2(N). Time can be easily captured in python using time.clock() whcih returns wall clock time since program was started. The time begins once we start step 2(a), if time equals the expression with N proceed to step c. <br>  
   2(a) Initialize the memory as a uniform superposition of qubits.<br>
Each qubit represents an index. 
After intializing the memory grab the y-th qubit and entangle the state of your register with this y-th qubit according the the Oracle given by Grover. 
This marks all T[j]<T[y].<br>
   2(b) Apply the quantum exponential searching algorithm [(2)], which is a generalized Grover's search.<br>
   2(c) Measure the first register, call that outcome y' which is an index into the table.
If the integer in the table at index y' is less than the integer at y, set y = y'.<br>

3. Return y.   
### Implementing QMSA
The steps 2(a) and 2(b) pose the biggest challenge if someone has no experience with Quantum Computing. We will first observe how to initalize the register. Then we will see how the QESA can be implemented to find a unique solution, in this case the minimum.

In order to intialize the register we pepare a uniform superposition of qubits, the number of qubits is determined by the number of elements in our table. We then grab our y-th index and entangle it with our register using a Controlled Z.

```
   // intialize register to number of qubits as there are indices
   using ((Register) = (Qubit[TableLength])) 
   { 
      // Create Uniform Superposition of all indices
      PrepareUniformSuperposition(TableLength,LittleEndian(Register)); 
      // Grab the qubit in the RandomIndex (which is intialized in our python host script using numpy.uniform.random()) and set it aside              
      let Marker = Register[RandomIndex]; 
      // Apply Oracle to flip all states that are T[j]<T[y]
      Controlled Z(Register,Marker); 
   }
```

Step 2(a) has been stasified. Now we must figure out how to apply the QESA algorithm. 
For the following circuits assume our Random Index is 0. 
#### QESA
The QESA algorithm is a generalized Grover's search characterized by the following circuit for an even number of table elements.     
We can visuailze our algorithm with a circuit design. 
Below is the QESA circuit.   

![QESA circuit](/assets/images/QESA.JPG)

Here is the QESA circuit treated as a gate with the intialization on qubits as mentioned above. 

![2 items in a list](/assets/images/2-item-list0.gif)


Lets break down whats going on here.
We utilize the register we were working with earlier and apply a H transform, this is stated to simply be the Hadmard. 

```
   // Apply Hadamard to register
   ApplyToEach(H,Register); 
```

We then apply a conditional phase shift if the qubit is 0.    
```
   // Reflect qubits that are 0s
   ApplyConditionalPhase_0(LittleEndian(Register));  
   
  operation ApplyConditionalPhase_0(register: LittleEndian) : Unit is Adj + Ctl
    {
        using (aux = Qubit()) 
        {
            // If qubit is 0 flip it!
            (ControlledOnInt(0,X))(register!,aux); 
        }
    }

```
From here we apply the inverse of the Hadmard, this is just the conjugate transpose since Hadamard is unitary. 
This can be implemented be calling the Adjunct of Hadamard.  

```
   // Apply Adjunct Hadamard to register
   ApplyToEachA(H,Register); 
```
The last step is another conditional phase shift, though it is applied if the qubit is 1.
This can be done by using the Z gate. 
The Z gate takes our qubit in and checks if it is |0> or |1>. 
If |0> leave it be.
If |1> map to |0>. 

#### QESA Script
Our Q# script will be structured as follows:

```

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
        operation Algorithm_Even(TableLength : Int, RandomIndex : Int) : Int
    {
        // intialize register to number of qubits as there are indices
        using ((Register) = (Qubit[TableLength])) 
        {
            within
                {   
                    // Create Uniform Superposition of all indices
                    PrepareUniformSuperposition(TableLength,LittleEndian(Register)); 
                    // Grab the qubit in the RandomIndex and set it aside
                    let Marker = Register[RandomIndex]; 
                    // Apply Oracle to flip all states that are T[j]<T[y]
                    Controlled Z(Register,Marker); 
                }
            apply 
                {
                    // Apply Hadamard to register
                    ApplyToEach(H,Register); 
                    // Reflect qubits that are 0s
                    ApplyConditionalPhase_0(LittleEndian(Register)); 
                    // Apply Adjunct Hadamard to register
                    ApplyToEachA(H,Register); 
                    //Reflect qubits that are 1s
                    Z(LittleEndian(Register)); 
                }
            Measure(Qubit[0])
        }
    }
    

    operation Algorithm_Odd(TableLength : Int, RandomIndex : Int) : Int
    {
        // intialize register to number of qubits as there are indices
        using ((Register) = (Qubit[TableLength])) 
        {
            within
                {   
                    // Create Uniform Superposition of all indices
                    PrepareUniformSuperposition(TableLength,LittleEndian(Register)); 
                    // Grab the qubit in the RandomIndex and set it aside
                    let Marker = Register[RandomIndex]; 
                    // Apply Oracle to flip all states that are T[j]<T[y]
                    Controlled Z(Register,Marker); 
                }
            apply 
                {
                    // Implemntation for odd number of table entries
                    QFTLE(LittleEndian(Register)); 
                    // Reflect qubits that are 0s
                    ApplyConditionalPhase_0(LittleEndian(Register)); 
                    // Inverse QFT by using BigEndian
                    QFT(BigEndian(Register)); 
                    // Reflect qubits that are 1s
                    Z(LittleEndian(Register)); 
                }
            Measure(Qubit[0])

        }
    }


    operation ApplyConditionalPhase_0(register: LittleEndian) : Unit is Adj + Ctl
    {
        using (aux = Qubit()) 
        {
            // If qubit is 0 flip it!
            (ControlledOnInt(0,X))(register!,aux); 
        }
    }

}

    
```

For futher information on how this was derived take a look at 'Tight bounds on quantum searching' [2].
It is important to note the above algorithm only works for a table with an even number of entries. 

The algorithm breaks down when applying the Hadamard gate as the Hadamard is layed across the diagnoal of a identity matrix which is equal in dimensions to the number of qubits we have. 
With a bit of math, if we try to lay a 2x2 matrix along an odd dimensioned identity matrix the transformation is not retained.
To circumvent this we introduce the following implementation, utilizing QFT.    
#### QESA special case
```
         operation Algorithm_Odd(TableLength : Int, RandomIndex : Int) : Int
    {
        // intialize register to number of qubits as there are indices
        using ((Register) = (Qubit[TableLength])) 
        {
            within
                {   
                    // Create Uniform Superposition of all indices
                    PrepareUniformSuperposition(TableLength,LittleEndian(Register)); 
                    // Grab the qubit in the RandomIndex and set it aside
                    let Marker = Register[RandomIndex]; 
                    // Apply Oracle to flip all states that are T[j]<T[y]
                    Controlled Z(Register,Marker); 
                }
            apply 
                {
                    // Implemntation for odd number of table entries
                    QFTLE(LittleEndian(Register)); 
                    // Reflect qubits that are 0s
                    ApplyConditionalPhase_0(LittleEndian(Register)); 
                    // Inverse QFT by using BigEndian
                    QFT(BigEndian(Register)); 
                    //Reflect qubits that are 1s
                    Z(LittleEndian(Register)); 
                }
            Measure(Qubit[0])

        }
    }
```
Now we refer back to our QMSA outline to observe that the Algorithm(TableLenght,RandomIndex) is iterated on until we find a suitable y' or we simply hit our time limit. The true stars of this algorithm are the time limit, which guarantees O(sqrt(N)), the generalized Grovers algorithm given in QESA, which provides for easy implementation and has O(1) for each iteration, and lastly, the oracle function which marks our states, which along with our intialization of qubits has O(log(n)).

Here is the python host that will apply the conditions of QMSA while using QESA.    
#### QMSA Script
```
    class DH(object):

      def __init__(self,table):
          # Flatten table
          self.table = np.ndarray.flatten(table) 
          # Number of elements in the table
          self.N= len(self.table) 
          # Choose our random index
          self.y = int(np.random.uniform(0,self.N-1)) 

      def QMSA(self,N,y,table):
      
          N = self.N

          y = self.y

          t = self.table

          y_prime = N
          # Time limit as specified in QMSA
          time_limit = (22.5*np.sqrt(N)+1.4*np.log(N)**2) 

          while time.clock() < time_limit:    

              for i in range(N):

                  while t[y] < t[y_prime]:
                      # Implementation of odd and even functionality
                      if N // 2 : 

                          y_prime= Algorithm.simulate(N,y)

                      else: 

                          y_prime=Algorithm_Odd.simulate(N,y)
                  # Check if we have found our minimum
                  if t[y_prime] < t[y]: 
                      # reset loop if found
                      i=0 
              #store index with minimum
              y = y_prime 
          return y                                                      


```

## Motivation
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

## My Background
I began my journey from math to computers through my passion for ML algorithms, where I stumbled upon quantum computing. 
I was instantly pulled in by the elegant algorithms and proposed efficiency.
About a year ago, I started looking into a model that treats nodes of neural networks as qubits. 
I took a deep dive into the research papers and found myself amazed by what was out there.
Realizing I had hunger to learn more I looked for online courses in quantum computing.
I began with a Coursera course from St. Petersburg University on quantum computing with detail on quantum algorithm design and quantum computer architecture. 
From here I tinkered with Q# and developed basic algorithms that I previously learned.
I found Dürr and Høyer's paper and saw it as an fusion of Duestch's, Shor's, and Grover's Algorithm; a perfect next step for me.

## Conclusion

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

1. Implementing QESA for multiple solutions   

2. Fluid QMSA time manipulation

------------------

[(1)]:https://arxiv.org/pdf/quant-ph/9607014.pdf
[(2)]:https://arxiv.org/pdf/quant-ph/9605034.pdf
[(3)]: Kitaev,  A. Yu.,  “Quantum measurements and the Abelian stabilizer problem”, manuscript quant-ph/9511-026 (1995).
