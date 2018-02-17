---
title: "Performance Analysis of TensorFlow on System z"
author: [Jan Hofmeier]
date: 2018-02-08
subject: "TensorFlow"
tags: [TensorFlow, ]
subtitle: "Report"
titlepage: true
titlepage-color: 06386E
titlepage-text-color: FFFFFF
titlepage-rule-color: FFFFFF
titlepage-rule-height: 1
...

# Introduction

Mainframes are commonly in the finance and insurance industries. Insurance companies have to deal with a massive number of requests and banks are interested in detecting fraudulent in real time to prevent them. Machine learning offers new opportunities for solving these problems.

# Basics

## Cache

CPUs have become faster over time. In fact, their performance has increased much quicker than that of the memory. As a result, the CPU capacity cannot be utilized to its full extent since the speed of the main memory (RAM) restricts it {Tanenbaum 1999 #34S: 264}. To achieve better utilization of CPUs, copies of frequently used data are stored in cache memory. Caches are much faster and can be therefore accessed more quickly, but are of a much smaller size than regular RAM. They are managed by a dedicated controller which transfers data between the cache memory and the RAM in cache blocks or cache lines. The cache controller always reads and writes entire cache lines. When a CPU sends a request for a word which is located in the cache, the request can be processed directly from the cache, which is called "cache hit". If it is not in the cache, the cache controller has to read it from RAM ("cache miss") {Rauber 2012 #35S: 75-77}. In this case, the CPU has to wait for the data to be loaded.
When the cache memory is full, certain data chunks will be written back to the main memory. Therefore bad requests will result in excessive writing-back and re-reading of data, significantly affecting the performance.

## Intel x86

x86 denotes a family of processors starting with the 8086 of chips originally developed and produced by Intel. Its instruction set has dramatically increased since then, and its word width has doubled in size, which resulted in a new but backward compatible architecture. The first change was introduced with i386 from 16bit register size and 20bit address bus to 32bit registers and a 32bit address bus. The 32bit architecture was later known as IA-32. The 64-bit architecture was originally introduced by AMD under the name AMD64. Intel adopted this architecture under the name Intel 64 which is also known as x86_64. All further references to x86 in this paper refer to x86_64 because this is the latest and currently most widespread Intel architecture.

With the success of the IBM PC which utilized the Intel 8088 processor, a variant of the Intel 8086, with an 8-bit external data bus instead of 16-bit, the successors of this architecture have dominated the desktop computer landscape by near 100% to this day. They are also commonly used in servers and clusters for all sorts of applications including machine learning.

The experiments described in this paper were carried out on an Intel(R) Xeon(R) Gold 6140.



## IBM System z

z Systems is the Mainframe product line of IBM. The latest model is called z14 and can be fitted with up to 196 individual z CPUs clocked at 5,2GHz and 32TB  of main memory. The CPUs are organized on chips and the chips up to 4 so-called drawers. SMT2 (Simultaneous Multithreading) enables every Processor to execute two Hardware Threads ~\cite[S.~34-36]{Lascu.2017}.

### Cache System

z14 has a 4 Level Cache System. All Caches are inclusive and have a cache line size of  256 Bytes.

| Level | Data  | Instruction | Data + Instruction | shared |
| ----- | ----- | ----------- | ------------------ | ------ |
| 1     | 128KB | 128KB       |                    | no     |
| 2     | 4MB   | 2MB         |                    | no     |
| 3     |       |             | 128MB              | chip   |
| 4     |       |             | 672MB              | drawer |

~\cite[S.~71]{Lascu.2017}

![under construction](./tbd.png)\

Pipeline



# Machine Learning

Machine learning (ML) systems have existed since the 1950s, but it is only a decade ago that it gained its momentum and became very popular. Three main reasons explain this development: 

- enormously increasing the amount of accumulated **data**, 
- **know-how** in building ML models and 
- powerful **technology** {https://hbr.org/2017/07/whats-driving-the-machine-learning-explosion}. 

**Data**: The past decade has seen the information creation process exploded due to cheap storage and new opportunities to collect and distribute data. This includes not only publicly available information from social media and different web platforms, but also information related to the fast development of the Internet of Things (IoT) technologies, which allow business to collect data from all sorts of sensors built in industrial equipment and digital devices. 

At the same time, the large amounts of structured data (like transaction, customer information, etc.) have been cumulated by companies for years and could not be used until recently due to the lack of know-how and suitable technologies or capacities. 

**Know-How**: The growing interest in artificial intelligence and machine learning (and fundings as a result) along with some important achievements made by a few big companies gave an impulse and encouraged more scientist to do research and develop better algorithms and models. As a result, many new approaches to deep learning or reinforcement learning have emerged opening new opportunities for data analysis and problem solution.  

**Technology**: Cloud technologies allow businesses to use computational power without having to buy and maintain on-premises hardware. GPUs that were initially developed to display graphics have been proven effective for ML purposes. Besides, with ML becoming more important, there are specialized chips developed for this kind of application (like Google's tensor processing unit, TPU). {https://hbr.org/2017/07/whats-driving-the-machine-learning-explosion}

Another factor that has boosted the development of ML is the memory.  Cheap RAM allows to keep and process data sets in-memory, which accelerates the learning process. Cache size is even more important for ML performance. CPUs usually have a hierarchy of caches, from small fast (L1, L2) to large slow caches (L3, L4). Efficient architectures and caching procedures are crucial for CPU performance. 

With all these factors opened new possibilities for analyzing data. Instead of using hardcoded formulas and conditions to make predictions computers are now able to detect new patterns in data and solve complex problems for which there are no efficient traditional programming solutions (like computer vision or speech processing).

The following chapters describe different types of machine learning approaches and algorithms as well as benchmarks which can be used to measure the performance of ML models.

## Types of Machine Learning Approaches

Machine learning problems can be broadly split into different categories, i.e., depending on:

- Whether the target results for training data are known or level of human supervision (supervised, unsupervised, semi-supervised and reinforcement learning). Since semisupervised systems are rarely used, and reinforcement learning is not a typical use case for System z, this paper is focused on supervised and unsupervised algorithms;
- Whether the ML system can learn "on the fly" (online vs. batch learning) {978-1-491-96229-9, Géron, Aurélien. Hands-On Machine Learning with Scikit-Learn and TensorFlow: Concepts, Tools, and Techniques to Build Intelligent Systems. O'Reilly Media. Kindle Edition., p.7 }

###Batch vs. online learning

In batch ML systems the model cannot be trained incrementally, so all available data is used for the system to be trained. Since a large amount of data are usually required to get good results, which is often very time-consuming. Therefor batch systems are usually trained offline. If a batch system becomes out of date, it needs to be retrained from scratch. The retraining means the systems must be stopped to relaunch the new version, although the process can be automated to reduce the interruption.

### Supervised vs. unsupervised learning

In supervised machine learning like regression or neuronal networks, the training samples contain besides the input data, typically a  feature vector, also the correct output value, called label. A model tries to predict the labels from the input features. It contains a set of parameters, called weights. These weights are derived by minimizing a cost function. The cost function is based on the error of the prediction. {Pattanayak 2017 #9I: 56}

Unsupervised problems like clustering, try to detect pattern without having a known correct output value.

Below is the description of the most commonly used ML models for supervised and unsupervised approaches.

##Supervised Learning

###Linear Regression

####Model

Linear Regression is one of the simplest algorithms for prediction metric data and forms the basis of many machine learning algorithms. The "model" used in linear regression is a function of the form:

$h_\theta(x)=x_1 * \theta_1 + x_2 * \theta_2 + .. + x_3 * \theta_3 + b$
$h_\theta(x)=\sum_{j=1}^{n} (\theta_jx_j) +b$

where $x$ is the input feature vector and $\theta$ is the vector of weights.  {Pattanayak 2017 #9: 56}  The variable $b$ is called bias and acts like a weight the corresponding feature of which always equals 1. Therefore the formula can be simplified by adding a feature $x_0$ to $x$ and treating $b$ as an normal weight $\theta_0$:

$h_\theta(x)=\sum_{j=0}^{n} (\theta_jx_j)$

The formula can be modified (assuming row vectors) using a vector multiplication.

$h_\theta(x)= \theta^Tx$

This formula would also work with vector-matrix multiplication for a $m \times n$ matrix $X$ where each column represents one sample:

$h_\theta(X)= \theta^TX$

####Cost Function

For a label vector $y$ a simple cost function is the squared norm of the error vector  {Pattanayak 2017 #9: 58}:

$C_X(\theta) = \frac {1}{2m}||h_\theta(X) - y ||^2$

By minimizing this function the value of $h_\theta(X)$ approaches the desired values in $y$ 

The closed-form solution for this minimization problem would be {Pattanayak 2017 #9: 59}:

$\hat{\theta}=(X^TX)^{-1}X^T*y$ 

This requires calculation of the inverse of the matrix $(X^TX)$ which has the dimensions of $m \times m$ resulting in a complexity of the order $O(n^3)$ {Courieu 2005 #14I: 27}. This means it scales very badly with the increasing number of samples and is not applicable in practice, where the iterative calculation is used instead.

#### Gradient Decent

If the cost function is differentiable (like the cost function for the linear regression), the standard approach is to use some variant of a gradient descent described below, which is a popular choice for a simple optimization problem {Andrychowicz 2016 #40}. 
In the first step, the weights are initialized with random values. Then the weights are optimized in an iterative fashion by calculation the gradient for every weight and changing the value of the weight in the opposite direction of the gradient and therefore descending along the gradients. The update rule is: \cite[66]{Pattanayak.2017} 

$\theta^{(t+1)}=\theta^{(t)}-\alpha * \Delta C_X(\theta^{(t)})$

where $\theta^{(t)}$ is the vector of the weights of the current iteration and $\theta^{(t+1)}$ the weights of the next iteration.   $\Delta C_X(\theta^{(t)})$ is the vector of gradients for the cost function $C_X(\theta)$. $\alpha$ is a scalar called learning rate. It determines how big the steps should be. If $\alpha$ is chosen to low, the learning will take too long, if $\alpha$ is chosen too big, it might overshoot the minimum.

One should note that gradient descent does not guarantee to find the global minimum. It can fall into a local minimum, but in most machine learning problems this is not a serious problem. Because of the rounding and other limitations, its unlikely to find the exact minimum. The iteration process is usually stopped if the magnitude of the updates falls below a certain value.~\cite[S.~66]{Pattanayak.2017}

The equation for the linear regression cost function stated above can be rearranged as follows: 

$C_X(\theta) = \frac {1}{2m} ||\theta^TX - y ||^2$ 

$= \frac {1}{2m} \sum_{i=0}^{m}(\sum_{j=0}^{n} (\theta_j x_j^{(i)}) - y^{(i)} )^2$ 

$= \frac {1}{2m} \sum_{i=0}^{m}((\sum_{j=0}^{n} (\theta_j x_j^{(i)}) - y^{(i)} ) * \sum_{j=0}^{n} (\theta_j x_j^{(i)}) - y^{(i)})$


The partial derivative with respect to $\theta_0$ is then:

$\frac {\partial} {\partial \theta_0} = \frac {1}{2m} \sum_{i=0}^{m}((x_0^{(i)} *( \sum_{j=0}^{n} (\theta_j x_j^{(i)}) - y^{(i)})) +  \sum_{i=0}^{m}((\sum_{j=0}^{n} (\theta_j x_j^{(i)}) - y^{(i)} ) * x_0^{(i)}))$

$=\frac {1}{m} (\sum_{i=0}^{m}((x_0^{(i)} * \sum_{j=0}^{n} (\theta_j x_j^{(i)}) - y^{(i)})$

$=\frac {1}{m} \sum_{i=0}^{m}(h_\theta(x^{(i)}) - y^{(i)}) * x_0^{(i)}$

And the partial derivative with respect to all other $\theta_j$ are calculated analogously: 

$\frac {\partial} {\partial \theta_j} = \frac {1}{m} \sum_{i=0}^{m}(h_\theta(x^{(i)}) - y^{(i)}) * x_j^{(i)}$

###Classification

Instead of a continuous variable, classification problems try to predict a discrete class label for a given input feature vector.~\cite[S.~61]{Pattanayak.2017}

##### Logistic Regression

The simplest form of classification is binary classification, meaning there are two classes with the assigned values $0$ and $1$. Linear regression can be modified to do this. The label vector $y$ 

####Neural Networks

![under construction](./tbd.png)\



##Unsupervised Learning

![under construction](./tbd.png)\



# Linear Algebra

## Scalar

A simple number is a scalar. It has only magnitude and no direction. {Pattanayak 2017 #9S: 3}

$x=1$ is a scalar

## Vector

A vector is a one-dimensional array of numbers. A vector exists in a vector space, and the number of the elements of the array is the vector space's dimensions.  A vector has a direction inside the vector space. A vector with only one dimension is a scalar. {Pattanayak 2017 #9S: 4}

$\vec {x} = \begin{bmatrix} x_{1} ,  x_{2} , ..  , x_{m}  \end{bmatrix}$            $\vec {y} = \begin{bmatrix} y_{1} \\ y_{2} \\ .. \\  y_{m}  \end{bmatrix}$

$\vec{x}$ is a row vector and $\vec {y}$ a column vector with its components $x_1$ to  $x_m$ and $y_1$ to $y_m$ respectively.  

## Matrix

A matrix is a two-dimensional array of numbers organized in rows and columns. Their number determines the size of the matrix. A matrix can be broken down into it rows or columns as vectors {Pattanayak 2017 #9S: 4}. A vector is, therefore, a special case of a matrix, where either the number of rows or the number of columns equals one. For example $A$ is a $m \times n$ matrix (m rows and n columns) with elements $a_{ij}$.

 $A = \begin{bmatrix} a_{11}  & a_{12}  & .. &a_{1n} \\ a_{21}  & a_{22}  & .. &a_{2n} \\ .. & .. & .. & .. \\  a_{m1}  & a_{m2}  & .. &a_{mn}  \end{bmatrix}$

Another important metric of a matrix is its rank which is the number of linearly independent column or row vectors. In fact, the number of independent columns vectors always equal to the number of independent row vectors {Pattanayak 2017 #9S: 10}.

###Representation in Memory

The computer systems described in this thesis are using a flat memory model. That means that all memory locations are addressed via a linear address space. The whole memory can be seen as one big single-dimensional array, where each address is represented by a single integer. A matrix uses two indices to address it is elements. This requires a mapping of the two indices into the linear address space of the memory. If the memory for the matrix is allocated as one continuous block, the mapping for a $m \times n$ matrix $A$ would look like this: $address(i,j):= b + e * (i*n +j)$
Where $b$ is the base address and $e$ the size of the element. This would be a row order matrix because the elements of a row are stored in adjacent memory locations.

The alternative is to store each row or column vector as a separate array and use an additional array which holds the base addresses of the vector arrays. This has the advantage that the addressing is more flexible, but comes at the cost of twice as much memory accesses and additional memory needed for the address array.

## Tensor

A tensor is an n-dimensional array of numbers. It is the generalization of matrices and vectors so that a vector can be described as a 1-D tensor and a matrix as a 2-D tensor. A color image, for example, can be expressed as three matrices, one matrix for every color and every number in a matrix is a pixel. Combining the 3 Matrices into one structure would result in a 3-D tensor. ~\cite[S.~5]{Pattanayak.2017} 

## Transpose

The transpose to a vector $V^T$ changes a row rector into a column vector and vice versa. When applied to a matrix $A \in \mathbb{R}^{m \times n}$ the transpose $A^T \in \mathbb{R}^{n \times m}$ is defined by transposing its row vectors into column vectors  ~\cite[S.~7]{Pattanayak.2017}. 

## Matrix Addition

Matrix addition $C=A+B$ is defined for two matrices with the same size by performing an element-wise scalar addition.

$\begin{bmatrix} a_{11}  & a_{12}  & .. &a_{1n} \\ a_{21}  & a_{22}  & .. &a_{2n} \\ .. & .. & .. & .. \\  a_{m1}  & a_{m2}  & .. &a_{mn}  \end{bmatrix} + \begin{bmatrix} b_{11}  & b_{12}  & .. &b_{1n} \\ b_{21}  & b_{22}  & .. &b_{2n} \\ .. & .. & .. & .. \\  b_{m1}  & b_{m2}  & .. &b_{mn}  \end{bmatrix} = \begin{bmatrix} a_{11} + b_{11}  & a_{12} + b_{12} & .. &a_{1n} +b_{1n} \\ a_{21} + b_{21} & a_{22} + b_{22} & .. &a_{2n} + b_{2n}\\ .. & .. & .. & .. \\  a_{m1} + b_{m1} & a_{m2} + b_{m2} & .. &a_{mn} + b_{mn} \end{bmatrix}$ 

## Matrix Multiplication

For two matrices $A \in \mathbb{K}^{l\times m}$ and $B \in \mathbb{K}^{m\times n}$ the matrix multiplication $C=A\cdot B$ is defined as:

$C = (c_{i,j})_{1\leq i \leq n \atop 1 \leq j \leq l}$ with the components:

$c_{i,j} := \sum\limits_{k=1}^m a_{i,k} \cdot b_{k,j}$  for all $i \in \{1,\cdots,l\}, j \in\{1,\cdots,n\}$

 {Stroetmann 19.12.2017 #15: 154}

The naive implementation in C (assuming the destination array is zeroed) would be:

```c
for(int i=0; i<l; i++)
    for(int j=0; j<n; j++)
        for(int k=0; k<m; k++)
            C[i][j]+=A[i][k] * B[k][j]
```

That shows the complexity is linear to each of the dimensions $l$, $m$ and $n$. For square matrices, the complexity would, therefore, be in $\mathcal{O} (n^3)$.

###Optimizing the Memory Access Pattern

The memory access pattern is not optimal for architectures with a cache memory. There are two problems if $B$ is too big to fit entirely into the cache: 

1. $B$ is not sequentially accessed in the inner loop. The most inner loop iterates over $k$ which is used as the first index of $B$, so every $n$-th element with the offset of $j$ is accessed. That means if $n$ > size of a cache-line for every access to $B$ an entire cache-line needs to be loaded which wastes both memory bandwidth and cache.
   This can be fixed by transposing $B$ so that $k$ is the last index  {Goto 6.5.2006 #17}:

   ```c
   for(int i=0; i<l; i++)
       for(int j=0; j<n; j++)
           for(int k=0; k<m; k++)
               C[i][j]+=A[i][k] * B[j][k]
   ```

2. Every element of $B$ is read $l$ times (outer loop), and between two accesses to the same element, all other elements are read (both inner loops). By the time the same element is reaccessed, it is probably evicted from the cache. There is no trivial solution for this problem. You could swap both outer loops, but this would only shift the problem to the matrix $A$.
   The problem could be moved away from $B$ by making the most inner loop the outer loop:

   ```c
   for(int k=0; k<m; k++)
       for(int i=0; i<l; i++)
           for(int j=0; j<n; j++)
               C[i][j]+=A[i][k] * B[j][k]
   ```

   In this case, the problem affects the matrix $C$. However, this can be handled by splitting the $k$ loop into parts (for simplicity reasons assume that $k$ is a multiple of the blocking size $bs$):

   ```c
   for(int bk=0; k<m; bk+=bs)
       for(int i=0; i<l; i++)
           for(int j=0; j<n; j++)
               for(int k=0; k<bs; k++)
                   C[i][j]+=A[i][bk+k] * B[j][bk+k]
   ```

   If $bs$ is chosen in such a way that $bs$ elements from $B$ fit into cache, they will not be evicted from the cache, when they are needed again. The count of the iterations over $C$ is reduced from $m$ by a factor of $bs$. 
   This is called a General block-times-panel multiply (GEBP) kernel.  {Goto 6.5.2006 #17}
   Because $bs$ roughly grows linear with the cache size, the number of iterations over $C$ is anti-proportional it is also roughly anti-proportional to the cache size, which means that it works more effectively with bigger caches.  The $bs$ iterations in the most inner loop give time to prefetch the next position of $C$.

   Mathematically speaking $A$ and $B$ get blocked into $M=m/bs$ sub-matrices (done by the outer loop in the C example), called panels along the common $m$ dimension, each containing $bs$ columns or rows respectively:

   $A \rightarrow (A_1 \mid A_2 \mid \dots \mid A_M)$               $B \rightarrow \begin{pmatrix} \underline{B_1} \\ \underline{B_2} \\ \underline{\dots} \\B_M\end{pmatrix}$

   $C$ can then be calculated by matrix multiplying every panel $A_p$ from $A$ with the corresponding panel $B_p$ from $B$  (done by the three inner loops) and adding the results together:

   $C= A_1B_1 + A_2B_2 + \dots +A_MB_M$  {Goto 6.5.2006 #17}

   Because GEBP uses a generic matrix multiplication on the panels, it can also utilize a more efficient algorithm than the naive matrix multiplication.


###Reducing loop overhead

In this example, only one multiplication and addition is carried out per iteration of the inner loop. This adds much overhead through the additional executed instructions for controlling the loop. Especially on pipelined processors, the jumps can mess up the pipeline. The effect of this problems can be minimized by a technique called loop unrolling. Which essentially means that the operation inside the loop is repeated several times in one iteration and the number of iterations is divided by this number. Loop unwinding also helps with fine-grained parallelism. ~\cite[S.~294]{Nicolau.1988} 
 Applied to the inner loop of the matrix multiplication example with an unwinding factor of 2:

```c
for(int k=0; k<bs; k+=2){
    C[i][j]+=A[i][bk+k] * B[j][bk+k];
    C[i][j]+=A[i][bk+1] * B[j][bk+1];
}
```

However, this causes a problem for $bs$ that are not a multiple of two or the unwinding factor respectively. To counter this, extra case handling is required.

```c
if(bs%2!=0)
    C[i][j]+=A[i][bk+bs-1] * B[j][bk+bs-1];   
```

GCC is capable of doing loop unrolling more or less transparently for the programmer in some cases. ~\cite{GNU.13.02.2018} ~\cite{GNU.15.01.2018}
Unwinding too large loops can result in trashing the instruction cache ~\cite[S.~279]{Bachir.2010}. 

### Other Matrix Multiplication Algorithms

There are other algorithms like Strassen's Strategy or Winograd's  variant, that need noticeably fewer operations. In terms of Bachmann–Landau notation Strassen needs only $\mathcal{O} (n^{7 log_2}) =\mathcal{O} (n^{2.86})$ operations and Coppersmith-Winograd only $\mathcal{O} (n^{2.38})$ operations {Anderson 2009 #19D: 1} compared to $\mathcal{O} (n^3)$ operations of the naive algorithm. There usage results in a significant potential performance benefit for bigger matrices. However, with the current technology, the matrix multiplication execution time is dominated by data access rather than by calculations. {Labarta 2008 #16: 156}



## Vector Multiplication

The definition of a vector-vector or matrix-vector multiplication is the same as the matrix multiplication with one or both matrices having one column or row respectively.

There is not much room to improve the memory access pattern over the naive implementation. In case of vector-vector multiplication, every memory location is only accessed once. In case of a matrix-vector multiplication, only the vector's memory locations are accessed multiple times. In use cases relevant for this thesis it is to assume that the vector fits into the L1 cache so that the multiple access is already as efficient as possible.
So the vector-matrix multiplication is most probably limited by the main memory or higher level cache bandwidth.

##Scalar Multiplication

Scalar Multiplication $B= \alpha * A$ is defined as the element-wise multiplication of a scalar $\alpha$ with a matrix or vector $A$. ~\cite[S.~153]{Stroetmann.19.12.2017} ~\cite[S.~131]{Stroetmann.19.12.2017}

$\alpha * \begin{bmatrix} a_{11}  & a_{12}  & .. &a_{1n} \\ a_{21}  & a_{22}  & .. &a_{2n} \\ .. & .. & .. & .. \\  a_{m1}  & a_{m2}  & .. &a_{mn}  \end{bmatrix} = \begin{bmatrix} \alpha * a_{11}  & \alpha * a_{12}  & .. &\alpha * a_{1n} \\ \alpha * a_{21}  & \alpha * a_{22}  & .. &\alpha * a_{2n} \\ .. & .. & .. & .. \\  \alpha * a_{m1}  & \alpha * a_{m2}  & .. &\alpha * a_{mn}  \end{bmatrix}$



# Tensorflow

Tensorflow is an open source library for machine learning originally developed by the Google Brain Team within Google's Machine Intelligence research organization mainly for deep neural networks research /cite{https://www.tensorflow.org/}. The library provides a framework for numerical computation using a directed data flow graph where nodes represent mathematical operations and edges represent the communication in the form of multidimensional tensors of data flowing between these operations. 

![under construction](./tbd.png)\



## Setup

Tensorflow is not available as a pre-compiled package for Linux on System z. For Linux on x86_64 exist pre-compiled tensorflow packages are available, but to support older processor generations, these binaries are compiled without the support for the latest instruction set extensions. So for best performance, a compilation for the specific processor generation is also required on x86.

### Bazel

The build of tensorflow is coordinated by bazel.  Bazel itself calls installed compilers, like gcc for the actual compilation and linking. All builds described in this thesis where used bazel 0.8.0 which was compiled from scratch for both architectures. https://github.com/bazelbuild/bazel/releases/tag/0.8.0

### GCC

####z14

Because GCC 7.2.0 had problems with the vector facility of z14 gcc 8.0.1 (experimental) was compiled from source and used instead. 

```../gcc/configure --prefix="/opt/gcc" --enable-shared --with-system-zlib --enable-threads=posix --enable-__cxa_atexit --enable-checking --enable-gnu-indirect-function --enable-languages="c,c++" --disable-bootstrap --disable-multilib```

```bash
make  
sudo make install  
export PATH=/opt/gcc/bin:$PATH  

sudo ln -sf /opt/gcc/bin/gcc /usr/bin/gcc
export C_INCLUDE_PATH=/opt/gcc/lib/gcc/s390x-ibm-linux-gnu/8.0.1/include  
export CPLUS_INCLUDE_PATH=/opt/gcc/lib/gcc/s390x-ibm-linux-gnu/8.0.1/include  
sudo ln -sf /opt/gcc/lib64/libstdc++.so.6 /usr/lib/s390x-linux-gnu/libstdc++.so.6  
```

https://github.com/linux-on-ibm-z/docs/wiki/Building-TensorFlow

#### x86

On skylake gcc version 7.2.0 from the official Ubuntu repositories was used as it supports all features of the processor.

### Tensorflow

Tensorflow 1.4.1 was compiled from source as it was the most recent version at the time. https://www.tensorflow.org/install/install_sources

#### z14

Compiling tensorflow on z14 requires some modification.

gcc 8.0.0 trunc
https://www.mail-archive.com/gcc-bugs@gcc.gnu.org/msg547216.html

Tensorflow 1.4.1
commit 139dff640c87733cab90a5a1b4f1876ba6c061fc

configure: jmalloc on; copts: -march=arch12 -mzvector

https://lengerrong.blogspot.co.uk/2017/09/fix-up-configurable-attribute-copts.html

aktuelle eigen version nach ~/.cache/bazel/_bazel_root/{hash}/external/eigen_archive kopieren

remove const

#### x86

Using AVX512 resulted whether in crashes, because of bad alignment. AVX512 instructions need memory addresses to be the multiple of certain values. Trying to fix the alignment resulted in wrong calculations. As there are no sources stating that tensorflow supports AVX512 it is assumed that it is not implemented yet. AVX512 was disabled by compiling tensorflow on the level of broadwell.

## Eigen Library

The execution of machine learning relies heavily on linear algebra. One of the most important and time-consuming tasks for example is matrix multiplication. For these purposes TensorFlow uses an external library for linear algebra - the BLAS-Library (basic linear algebra subroutines) called "Eigen". {Google 2018b}

commit: f3a22f35b044 https://bitbucket.org/eigen/eigen/

### Cache Optimizations

Eigen implements the general block panel kernel algorithm for matrix multiplication. https://bitbucket.org/eigen/eigen/src/034b6c3e101792a3cc3ccabd9bfaddcabe85bb58/Eigen/src/Core/products/GeneralBlockPanelKernel.h?at=default&fileviewer=file-view-default To work most efficiently it needs to know the size of the caches to adjust the blocking size accordingly. On some architectures it can be determined automatically, but not on z14. The default cache sizes are defined as: 

```c
const std::ptrdiff_t defaultL1CacheSize = 16*1024;
const std::ptrdiff_t defaultL2CacheSize = 512*1024;
const std::ptrdiff_t defaultL3CacheSize = 512*1024;
```

For the optimization purposes within the scope of this thesis the cache sizes were adjusted to utilize the correct values for z14:

```c
const std::ptrdiff_t defaultL1CacheSize = 128*1024;
const std::ptrdiff_t defaultL2CacheSize = 4096*1024;
const std::ptrdiff_t defaultL3CacheSize = 128*1024*1024;
```

### Benchmark

Eigen comes with the benchmark suit based on BTL (Benchmark Template Library) for benchmarking different BLAS libraries including Eigen itself. This also gives the possibility to compare the raw performance of the LA operations behind ML. However, it is limited to only use one thread. 

The benchmark measures the number of (effective) MFLOPS (million floating point operations per second) for different size square matrices. The higher the number of MFLOPS the better. For small matrices, the number of FLOPS is expected to be lower because the benchmark uses dynamic-size matrices, which are inefficient for small matrices. ~\cite{Eigen.07.12.2016}

The single most time-consuming operation for most machine learning algorithms is matrix multiplication.

This test is conducted with 32bit (Single Precision) floating point numbers.

![MatrixMultiplication](C:\Users\IBM_ADMIN\Documents\PE5\Praxisarbeit5\MatrixMultiplication.png)

All configurations start with a poor performance for very small matrices as expected.  For very small matrices until the size of $3 \times 3$ the performance on all configurations is about the same. The configurations with SP vectorization experience a local peak for $4 \times 4$ matrices, while z14 without vectorization grows much slower in a steady way. For $5 \times 5$ matrices the performance of the vectorized configuration decreased nearly back to the level of the non-vectorized implementation until the size of 8. This pattern is repeated in regular intervals of 4 for z14 with vectorization. This is probably caused by the fact that a 128bit vector register has enough space for four 32bit numbers. So when a matrix size is a multiple of 4, the vectorized configurations can work most effectively and every other size needs additional handling of the remaining numbers. z14 is much more affected by this performance hits. On Skylake, this pattern is much weaker and sometimes disappears completely. The vectorized configuration of both z14 and Skylake have a steep upward curve which starts to flatten out at the size of 100 and almost entirely converges at the size of 1000. z14 with cache optimizations reaches a record just above 25000 MFLOPS and z14 with the standard cache configurations just slightly below 24000 MFLOPS. Skylake reaches just over 16000 MFLOPS, although in some cases it beats z14 for very small matrices.  z14 without vectorization converges at a maximum just below 5000 MFLOPS. This is $1/5$ of the performance of z14 with vectorization. Nearly a fourfold of the performance would be expected, since vectorization allows four arithmetic operations to be carried out in parallel with a constant time for other instructions that cannot be vectorized. One explanation of the additional gain of performance would be that the overhead of control instructions like loops is decreased.
There seems to be no penalty for exceeding cache sizes in neither of the configurations. Together with the observation that the cache optimization for z14 has only a very small effect and the fact that z14 without vectorization performs noticeably worse, lets to the conclusion that the performance of the matrix multiplication is most likely limited by the raw computational power of the processor and not the memory bandwidth.   

![Vector](C:\Users\IBM_ADMIN\Documents\PE5\Praxisarbeit5\Vector.png)

This test measures the performance of vector operations. Two vectors $X$ and $Y$ are scalar-multiplied with their coefficients and then in place added together: $Y= \alpha * X + \beta * Y$
The performance grows very steep with the vector size until it peaks around a size of 137 for z14 elements at ca. 1700 MFLOPS and for Skylake at 182 with 17655 MFLOPS. Skylake has very significant local peaks over the upward curve. After this global peak, the performance of z14 falls gradually until it levels out at around 1500 MFLOPS, whereas Skylake drops abruptly to around 16000 MFLOPS where it levels out. Around the vector size of 1000, the performance of Skylake drops again to just under 10000 MFLOPS. z14 experiences a similar drop at twice the vector size. In the end, Skylake seems to drop again for the third time, while z14 follows a slight upward trend.
The performance of z14 with or without vectorization is nearly identical, which indicates that memory/cache bandwidth is dominant, while the speed of the mathematical operations seems to be negligible.
The drops in performance could be attributed to the capacity of the the caches being exhausted. z14 has much bigger caches than Skylake which would explain why the drop occurs first on Skylake.   



# Glibc

For some basic mathematical functions both TensorFlow and Eigen rely on the implementation of the standard C-Math-Library libm of the operating system. In the case of the Ubuntu GNU/Linux (and most other distributions), this is contained in the glibc.
The installed version of glibc is 2.24.

```CC="gcc -fno-pie -no-pie" CXX="g++ -fno-pie -no-pie" ../glibc/configure --prefix=/usr```

###Exponential Function

Stand: https://sourceware.org/git/?p=glibc.git;a=commit;h=09e56b9e18f987105e39768f907db800e9330930

For efficient calculation of the single precision exponential function, the glibc implementation uses a table of precomputed and a polynomial of third order to approximate between this values. Ignoring such factors as data stored in the cache (in a non-uniform memory architecture), this results in a stable runtime for all values in the domain of the function.

The formula for the exponential function can be rearranged, so that:  

$exp(x) = 2^{k/N} * 2^{r/N}$ 

whereby $k$ and $r$ must satisfy the following condition: 

$k+r=z$ where $z=x*N/Ln(2)$ 

$k$ is obtained by rounding $z$ to the nearest integer and $r$ is calculated to satisfy $k+r=z$ this results in a value of $r$ between $-0.5$ and $0.5$.

Below is a code snippet from the glibc implementation of the exponential function:

```c
  t = T[ki % N];
  t += ki << (52 - EXP2F_TABLE_BITS);
  s = asdouble (t);
  z = C[0] * r + C[1];
  r2 = r * r;
  y = C[2] * r + 1;
  y = z * r2 + y;
  y = y * s;
```

$2^{k/N}$ is calculated by splitting $k/N$ into a an integer and a fraction and calculating the power of two separately by applying  $2^{a+b} = 2^{a} * 2^{b}$. 

$\Rightarrow a+b = \frac {k} {N} , \text{if $a=k \setminus N$  and $b=\frac {k \mod N} {N}$}$

$\Rightarrow 2^{\frac {k} {N}} = 2^{k \setminus N} * 2^{\frac {k \mod N} {N}}$

Because  $N$ is defined as a power of two $N=2^{EXP2F\_TABLE\_BITS}$ a division by $N$ moves the fraction point in binary representation $EXP2F\_TABLE\_BITS$ digits to the left. This can be implemented by a right shift by $EXP2F\_TABLE\_BITS$.

The calculation of $2^{k \setminus N}$ can be done very efficiently by exploiting how the IEEE754 binary64 type (also known as double or double precision floating point) is represented in memory:

| sign | exponent | significand |
| ---- | -------- | ----------- |
| 1    | 11       | 52          |

~\cite[S.~9]{IEEE.29.08.2008}.  

The components of a binary floating point number are interpreted as:

$(-1)^{sign} * significand * 2^{exponent}$  

~\cite[S.~4]{IEEE.29.08.2008}

The exponent in the representation in memory just needs to be set to $k\setminus N$. Counting the bits from right to left in the IEEE754 double representation in memory the exponent starts at the bit $53$. So a left shift by $52$  is necessary to move $k$ from its integer representation to the exponent in the IEEE binary64 representation {IEEE 29.08.2008 #41: 13} .

Combining the left shift by $52$ and the right shift by $EXP2F\_TABLE\_BITS $:

```c
ki << (52 - EXP2F_TABLE_BITS);
```

This leaves the rightmost $EXP2F\_TABLE\_BITS$ bits of $k$ in the most significant bits of the significand, which needs to be addressed later. 

There is no efficient way to calculate $2^n$ for a non-integer $n$. But in the expression $2^{\frac {k \mod N} {N}}$ the exponent only has $N$ distinct values. In this implementation $EXP2F\_TABLE\_BITS$ euqals $5$ so $N$=32, which is fairly small, so the values can be precomputed and saved in the table $T$. 

```t = T[ki % N];``` takes the precomputed value for  $2^{\frac {k \mod N} {N}}$ from the table. Setting the significand of the double from the previously computed $2^{k/N}$. 

Because the exponent part and the significand part are merged by adding their integer representation, the table has to account for the ```EXP2F_TABLE_BITS``` that are not shifted to the exponent and stay in the higher bits of the significand. 

After computing the bit representation of $s$ in the uint64 ```t``` it simply has to be reinterpreted as a double:

```s = asdouble (t);```

Since $r$ ranges from $- \frac {1} {2}$ to $\frac {1} {2}$ and is divided by $N=32$, the range of $2^{\frac {k} {N}}$ is also very small. For this narrow window $2^{\frac {k} {N}}$ can be approximated by a third order polynomial. $2^{\frac {k} {N}} = \cong C_0*r^3 + C_1*r^2 + C_2*r + 1$

```c
z = C[0] * r + C[1];
r2 = r * r;
y = C[2] * r + 1;
y = z * r2 + y;
```

Now the ```s``` and ```y``` just need to be multiplied to get the result:

````
y = y * s;
````

# Hardware Support

## Vectorization

In machine learning (ML) the same mathematical operation needs to be carried out over many elements. To perform one operation on multiple elements more efficiently many processors have special instructions, so-called Single Instruction Multiple Data (SIMD) instructions. .x86 has had a few SIMD extensions to the original instruction set. The first was MMX, followed by Streaming SIMD Extensions (SSE) 1 to 4 and Advanced Vector Extensions (AVX). Intel increased the size of the vector registers from 128bit to 256bit on AVX2 and doubled the size again with AV512 to 512bit.

The z/Architecture first added the so-called "Vector Facility" containing SIMD instructions on z13 supporting integers and Double Precision (DP) (64bit) Floating Point (FP). In z14 the support of Single Precision (SP) (32bit) Floating Point was added. The 32 128-bit vector registers are overlaying the 16 64bit floating point registers.

In machine learning most algorithms do not need high precision, so they work with 32bit floats. That means that four elements fit in a 128bit vector registers and up to 16 elements in a 512bit register. So the processor only needs to execute a fraction of the original number of arithmetic instructions.



## Fused Operations

The core of most ML algorithms multiplies a set of features with a corresponding set of weights and adds them together. So most calculations are multiplying two values and adding the product to a third value. To speed this up modern processors have so-called fused multiply and add operations.

Both x86 and z14 support the fused operations for vectorization. z supports vectorized and not-vectorized (scalar) and has special instructions like "vector FP multiply and add".



# Measurements

The measurements have been carried with different number of CPUs (scaling behavior). Single core performance has the most problems and the most optimization potential. 



## Text Classification

The text classification benchmark is based on the text classification example provided by tensorflow. https://github.com/tensorflow/tensorflow/blob/master/tensorflow/examples/learn/text_classification.py It uses a recurrent neural network (RNN) to predict a class for a given text. The RNN constructed out of GRUCells (Gated Recurrent Unit cell) using the $tanh$ activation function. As datatype _tf.float32_ (single precession floting point) is chosen. https://www.tensorflow.org/api_docs/python/tf/contrib/rnn/GRUCell
The samples for training and prediction are taken from dbpedia: https://github.com/le-scientifique/torchDatasets/raw/master/dbpedia_csv.tar.gz.
It contains 559,998 labeled training samples and 7000 labeled test samples. The time to predict 7000 samples is too small and therefore not indicative. For that reason the test size is doubled 7 times by appending it to itself. 

To measure the time taken by the different steps in the script _printTime(label)_ function and calls to it are added after each step, which prints the time difference since its last call together with the name of the step. This thesis only considers the time needed for training and the time for making predictions. It focuses predominately on the prediction performance. This is because in productive environments a model is often only trained once and than used to make predictions on a huge amount of data, which means that the prediction time becomes more relevant. 

## Scaling

To gain an overview over the performance scaling, the tests have been carried out with different amounts of cores. This process has been automated using a script. Since every core has two hardware threads with SMT2 on z14 and Hyper Threading on Skylake, the number of threads increases by a factor of two. The tests where conducted on z14 and Skylake both with full SIMD support enabled except AVX512 for Skylake disabled.

------- Graphs here ----

While Skylake's speedup flattens out very fast during the training, z14s speedup increases steadily, although far below the ideal speedup.
During the prediction both systems experience a extremely bad speedup. On Skylake the speedup even decreases below 1 with 6 threads and flattens out at the speedup of 0.6, meaning the performance even decreased adding more threads. The performance decrease on Skylake needs further investigation, but is not subject of this thesis, as it focuses mainly on investigating z14 performance.
One explanation for this bad scaling behavior is, that the model is too small to be split over multiple processors and this simple example incorporates no algorithms to distribute the training into multiple workers that can work semi separately. This is supported by the observation, that the CPU utilization is very low for many cores.
The problem of bad scaling during the prediction process could be avoided by splitting the set of samples upfront and feeding them into separate TensorFlow processes, one for each hardware thread.

---- Graphs here ----

Although z14 has a better scaling behavior, its performance on this example is worse than Skylakes in absolute numbers, even with several cores. This is especially true for one core.

The rest of this thesis will therefore investigate the single core performance.

## Single Core Performance

All tests in this chapter are conducted with only one hardware thread enabled.

### Profiling

On skylake performance data, inclusive the call stack can be collected via the linux perf tool. perf is also available on zLinux, but requires at least Linux Kernel 4.15 to collect call stack information. Additionally hardware instrumentation (HWI) is available on z14. HWI has the advantage, that it gives very detailed information per machine instruction, but doesn't support call-graphs.

For predictions 22,56% of the time is spend in Eigen::internal::gebp_kernel, according to the HWI data. gebp_kernel is the general block kernel implementation of the Eigen library, which does matrix multiplication. It is expected that much time is spend on matrix multiplication. 38% is spend in libm, the standard C math library. Inside libm most time is spend in the floating point exponential function and it's helper functions. 

libm is part of the glibc. Since the version of the system glibc 2.24, there are several improvements including improvements to expf. To test if this improvements make a difference, the latest version of glibc was compiled from commit 09e56b9e18f987105e39768f907db800e9330930 

The newer glibc version decreased the prediction time from a total of 500 seconds to 370.
This is also reflected in HWI data, the portion of libm decreased to 20%. Inside libm the portion is split between _\_expf with 13% of the total time and _\_expf_compat with 6%. _\_expf_compat is a compatibility wrapper containing error handling around _\_expf, that is used because tensorflow was linked against the older version of glibc. Because configuring tensorflow to be linked against another glibc than the system glibc would be to costly of working hours, it was instead chosen to remove everything from the wrapper but the call to _\_expf. 

With the now empty wrapper the prediction time was reduced to 337. That is a saving of nearly 9% which is greater than expected considering that _\_expf_compat accounted for only 6% according to the HWI data. The additional savings could be caused by compiler optimizations like inlining of the wrapper into the caller, which basically saves a function call. 

![under construction](./tbd.png)\

# Conclusion

z14 has proven that its Hardware is capable of executing the basic operations required by machine learning algorithms competitively or even faster in some cases. To archive this performance by exploiting the new vector instructions, added with the latest generation, recent software is needed. But at the application of this basic Operations in tensorflow, z14 stayed behind the expectations and the competition. To exploit the full potential of the z/Architecture for machine learning further adoption of tensorflow and its libraries to the z platform are necessary. To stay competitive it should be considered expanding z's vector facility in the future generations to 256 or even 512bits. 

## Notizen

####Test Classification

Training:

Prediction/Inference:

- Skalierung (kleine Daten)
- Skalierung (große Daten)
- Skalierung (neue glibc)

Optimierungen:

Exponentialfunktion (glibc -> aktuelle Version)

auf z Unterschied

auf x86 kaum merkbar

bei Verktorization prediction hat was gebracht, 

aber nur ohne Verctorization von z13



compat auskommentieren



Da bei ML viel auf MM basiert und viel Zeit drauf geht, wurden die Messungen für MM gemacht.

Matrix multiplication performance

Cache sizes angepasst 
