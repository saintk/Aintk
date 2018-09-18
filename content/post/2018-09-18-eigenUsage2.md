---
title: "Eigen库的使用技巧(二)"
tags: ["C/C++", "SLAM" ]
date: "2018-09-17"
---

一个博客写不完，分开写吧，后面再统一整理顺序.
Part I   为 eigen的基本数据结构及其操作
Part II  为 eigen求解线性代数问题
Part III 为 eigen求解几何问题

本篇为Part II: eigen求解线性代数问题
<!--more-->
## 二、Linear Problem
### 1. 线性代数和矩阵分解
基本问题就是，如何求解下面线性代数问题:
$$ Ax = b$$  
其中A和b是矩阵( 特殊情况下，b可能为vector), 求解x。  
#### <strong>线性方程组求解:</strong>
可以使用不同的矩阵分解方法求解这个问题，其速度或精度需要自己权衡.
基本用例如下:
```cpp
int main()
{
   Matrix3f A;
   Vector3f b;
   A << 1,2,3,  4,5,6,  7,8,10;
   b << 3, 3, 4;
   cout << "Here is the matrix A:\n" << A << endl;
   cout << "Here is the vector b:\n" << b << endl;
   Vector3f x = A.colPivHouseholderQr().solve(b);
   cout << "The solution is:\n" << x << endl;
}
```  
不同的分解方法，其精度和速度如下:
![精度和速度权衡](/media/posts/2018-09-18-eigen/different_decompose.png)   
即，如果矩阵是<strong>正定</strong>的，使用LLT或LDLT是很好的选择.

#### <strong>矩阵的特征值与特征向量:</strong>
Eigen 提供了多种方法去求解矩阵的特征值和特征向量，对于自伴随矩阵，使用 SelfAdjointEigenSolver最佳，对于一般的实数矩阵和复数矩阵，使用 EigenSolver或 ComplexEigenSolver.  
example:
```cpp
EigenSolver<MatrixXf> es;
MatrixXf A = MatrixXf::Random(4,4);
es.compute(A, /* computeEigenvectors = */ false);
cout << "The eigenvalues of A are: " << es.eigenvalues().transpose() << endl;
es.compute(A + MatrixXf::Identity(4,4), false); // re-use es to compute eigenvalues of A+I
cout << "The eigenvalues of A+I are: " << es.eigenvalues().transpose() << endl;
```

#### <strong>最小二乘解:</strong>
求解最小二乘解的最精确的方法是SVD分解，eigen提供了两种实现: BDCSVD & JacobianSVD，推荐使用BDCSVD。  
example:
```cpp
int main()
{
   MatrixXf A = MatrixXf::Random(3, 2);
   cout << "Here is the matrix A:\n" << A << endl;
   VectorXf b = VectorXf::Random(3);
   cout << "Here is the right hand side b:\n" << b << endl;
   cout << "The least-squares solution is:\n"
        << A.bdcSvd(ComputeThinU | ComputeThinV).solve(b) << endl;
}
```
除了SVD分解，还可以使用QR分解和Cholesky分解，more faster but less reliable.  

<strong>应用中，Cholesky分解可能更加多见，如下:</strong>
```cpp
// 求解 Ax = b的最小二乘解，等价于求解 A'Ax = A'b的解.
MatrixXf A = MatrixXf::Random(3, 2);
VectorXf b = VectorXf::Random(3);
cout << "The solution using normal equations is:\n"
     << (A.transpose() * A).ldlt().solve(A.transpose() * b) << endl;
```

其他一些，分解和构造函数分离，矩阵秩的获得等不再赘述。  
一些意见:
1. LLT 是最快的solver.
2. 对于大型的超定方程，Cholesky/LU分解的cost主要在计算 对称协方差矩阵.
3. 对于超大规模的问题， 只有实现 cache-friendly blocking strategy的方法，才能很好的扩展，所以在 4k x 4k的矩阵中， HouseholderQR 会比 LDLT更快; 目前 LLT, PartialPivLU, HouseholderQR, and BDCSVD 支持该策略。  

### 2. 稀疏矩阵的使用  
对于很多应用，经常会遇到一个超大的矩阵，只有很少一部分系数非0； 在这种情况，使用稀疏矩阵的结构可以有效的减少内存占用、提高性能。  
稀疏矩阵实现了 Compressed Column(or Row) Storage, 精简的保存矩阵数据。一般的稀疏矩阵，元素之间会预留空的空间，方便插入新元素； 当元素之间不预留任何空间，即为压缩模式。  

Sparse矩阵相对于Dense矩阵，少了很多灵活性，其中一个很大的限制就是<strong>运算时，两个稀疏矩阵的存储方法必须一致</strong>, 这就导致了计算 A'+A 必须将A'存在一个临时变量中，如下:
```cpp
SparseMatrix<double> A, B;
B = SparseMatrix<double>(A.transpose()) + A;
```
其他的块操作与dense矩阵基本一致，并且两者可以很方便的进行转换。  

eigen提供了丰富的 built-in solvers，具体参考[官网](http://eigen.tuxfamily.org/dox/group__TopicSparseSystems.html)  
所有的sovler都是相同的用法:
```cpp
#include <Eigen/RequiredModuleName>
// ...
SparseMatrix<double> A;
// fill A
VectorXd b, x;
// fill b
// solve Ax = b
SolverClassName<SparseMatrix<double> > solver;
solver.compute(A);
if(solver.info()!=Success) {
  // decomposition failed
  return;
}
x = solver.solve(b);
if(solver.info()!=Success) {
  // solving failed
  return;
}
// solve for another right hand side:
x1 = solver.solve(b1);
```  
在某些问题，求解可以<strong>使用同一个pattern</strong>，可以用到下述优化方法，将求解给拆解开来:
```cpp
SolverClassName<SparseMatrix<double> > solver;
solver.analyzePattern(A);   // for this step the numerical values of A are not used
solver.factorize(A);
x1 = solver.solve(b1);
x2 = solver.solve(b2);
...
A = ...;                    // modify the values of the nonzeros of A, the nonzeros pattern must stay unchanged
solver.factorize(A);
x1 = solver.solve(b1);
x2 = solver.solve(b2);
...

// compute() 等价于 使用 analyzePattern() & factorize()
```
不同的sovler提供了不同的功能，细节需要到对应的class文档中查阅。  

<strong>Matrix-free solvers</strong>  
Matrix-free 是求解线性方程的一种特殊解法，不保存系数矩阵，仅通过 matrix-vector products来获得；这种方法是在矩阵超级大，即使是稀疏矩阵来要消耗大量运算资源的情况下，进行使用: [基本用法](http://eigen.tuxfamily.org/dox/group__MatrixfreeSolverExample.html)  
