---
title: "Eigen库的使用技巧"
tags: ["C/C++", "SLAM" ]
date: "2018-09-17"
---
这几天是疯狂开坑啊，开了又不填，不能怪我，毕竟没人看嘛，哈哈~  
今天这个坑是 Eigen的使用技巧~  
大部分来自于Eigen官网，这里并不会完全的罗列，一些我认为没必要讲的，就略过了~

<!--more-->

## <strong>前言</strong>
Eigen作为常见的矩阵运算库，其速度和可靠性是得到保证的。与其他的矩阵运算库（intel mkl、ACML、GOTO BLAS、 ATLAS）相比，从官网给出的数据来看，优化已经逼近MKL [Benchmark](http://eigen.tuxfamily.org/index.php?title=Benchmark)  
另外SLAM中常用的ceres，深度学习常用的tensorflow 都是基于eigen开发的， 足见其性能.

当然，如果实在需要高性能的计算，在编译是可加入Intel MKL的，就能既有Intel MKL的速度，又能拥有Eigen的优雅~
[Eigen+MKL教程](http://eigen.tuxfamily.org/dox/TopicUsingIntelMKL.html)

## 一、Dense matrix and array manipulation
### 1. 基础矩阵类  

#### <strong>构造方法</strong>
matrix类6个模板参数，其中前3个是必选项，后3个为可选项。
```cpp
Matrix<typename Scalar,
       int RowsAtCompileTime,
       int ColsAtCompileTime,
       int Options = 0,
       int MaxRowsAtCompileTime = RowsAtCompileTime,
       int MaxColsAtCompileTime = ColsAtCompileTime>
```  
6个参数分别为: 数据类型、行数、列数、数据的存储顺序、最大的行数、最大的列数。  
<strong>注:</strong>后面三个参数一般不用指定，数据的存储顺序基本上也没什么用，因为Eigen主要是使用Column-major实现的，意味着即使Eigen支持 row-major，性能上也是column-major更好； 除非其他外部接口库是使用row-major，否则不建议使用。

Vector是特殊的矩阵，区别只是列数默认为1（RowVecor是行数默认为1).   
矩阵的行列数可以一开始就指定，也可以设置为动态长度，其<strong>构造方法</strong>如下:
```cpp
Matrix3f a;
MatrixXf b; // 不指定长度，则其默认长度为0x0

// dynamic长度也可以在给构造是就指定
MatrixXf a(10,15);
VectorXf b(30);

//当然，为了提供统一的API for fixed-size 和 dynamic-size的矩阵，使用这种指定方法去构造 fixed-size matrixes也是合法的:
Matrix3f a(3,3);

//而对固定长度Vector来说，可以用类似的结构去初始化其内部元素
Vector2d a(5.0, 6.0);
Vector3d b(5.0, 6.0, 7.0);
Vector4d c(5.0, 6.0, 7.0, 8.0);
```

#### <strong>赋值方法</strong>   
赋值方法主要有两种，一种是对应元素标号进行指定，一种是comma-initializer syntax。
```cpp

int main()
{
    //对应元素赋值
    MatrixXd m(2,2);
    m(0,0) = 3;
    m(1,0) = 2.5;
    m(0,1) = -1;
    m(1,1) = m(1,0) + m(0,1);
    std::cout << "Here is the matrix m:\n" << m <<std::endl;
    VectorXd v(2);
    v(0) = 4;
    v(1) = v(0) - 1;
    std::cout << "Here is the vector v:\n" << v <<std::endl;
    //Comma-initialization
    m<< 1,2,3,4;
    std::cout << "Here is the matrix m with Comma-initialization :\n" << m << std::endl;
}
```

#### <strong>Dynamic vs Fixed</strong>
既然可以指定长度和动态长度，那两者有何区别呢？ 用哪种比较好呢？ 
Matrix4f mymatrix 等价于 float mymatrix[16]
而动态长度永远是等价于使用 heap 去 allocate;  
MatrixXf mymatrix(rows,cols) 等价于 float *mymatrix = new float[rows*cols];  
另外， MatrixXf 对象还需要保存其 size。

固定长度的限制在于，必须在编译的时候就知道其size； 另外，在矩阵比较大的情况下，例如32，其相对于动态长度的matrix，性能上的提升微乎其微。
更糟糕的是，创建一个超级大的matrix,可能会导致 stack overflow.
最后，动态size时使用vectorize( use SIMD) , 肯能会更加 aggressive.  
<strong>总而言之，在元素size小于等于16的情况下，使用固定长度较好，在较大size的情况下， 使用动态长度较好。</strong>  

### 2. 矩阵和向量运算  
Eigen中是重载了 +、-、*等基本运算符的，基本的加减乘，只要矩阵维数合乎数理，是可以直接进行操作的  
<strong>矩阵的加减和数乘:</strong>
```cpp
int main()
{
  //Addition and substraction
  Matrix2d a;
  a << 1, 2,
       3, 4;
  MatrixXd b(2,2);
  b << 2, 3,
       1, 4;
  std::cout << "a + b =\n" << a + b << std::endl;
  std::cout << "a - b =\n" << a - b << std::endl;
  std::cout << "Doing a += b;" << std::endl;
  a += b;
  std::cout << "Now a =\n" << a << std::endl;
  Vector3d v(1,2,3);
  Vector3d w(1,0,0);
  std::cout << "-v + w - v =\n" << -v + w - v << std::endl;

  //Scalar multiplication and division
  a << 1, 2,
       3, 4;
  std::cout << "a * 2.5 =\n" << a * 2.5 << std::endl;
  std::cout << "0.1 * v =\n" << 0.1 * v << std::endl;
  std::cout << "Doing v *= 2;" << std::endl;
  v *= 2;
  std::cout << "Now v =\n" << v << std::endl;
}
```
<strong>注意:</strong> 无需顾虑使用Eigen计算较大的矩阵，其内部会进行循环展开，只遍历一次(当然使用SIMD可以更加优化)。请放心使用Eigen进行计算~



<strong>矩阵转置和共轭:</strong>
没什么特别的，直接调用函数 transpose(), conjugate(),adjoint()即可。  
但需要注意的是，<strong>不要</strong>使用如下展开方法:
```cpp
Matrix2i a; a << 1, 2, 3, 4;
cout << "Here is the matrix a:\n" << a << endl;
a = a.transpose(); // !!! do NOT do this !!!
cout << "and the result of the aliasing effect:\n" << a << endl;

//结果会因为转置没有完成就赋值而发生错误
```


<strong>矩阵乘法:</strong>  
乘法已经被重载，直接按照数学表达进行相乘即可。
```cpp
int main()
{
  Matrix2d mat;
  mat << 1, 2,
         3, 4;
  Vector2d u(-1,1), v(2,0);
  std::cout << "Here is mat*mat:\n" << mat*mat << std::endl;
  std::cout << "Here is mat*u:\n" << mat*u << std::endl;
  std::cout << "Here is u^T*mat:\n" << u.transpose()*mat << std::endl;
  std::cout << "Here is u^T*v:\n" << u.transpose()*v << std::endl;
  std::cout << "Here is u*v^T:\n" << u*v.transpose() << std::endl;
  std::cout << "Let's multiply mat by itself" << std::endl;
  mat = mat*mat;
  std::cout << "Now mat is mat:\n" << mat << std::endl;
}
```
<strong>注:</strong>乘法操作m=m*m 不会带来如转置一样的赋值错误，因为在eigen中乘法是被特殊对待的，它会对其进行展开:
```cpp
// m = m*m 等价于
tmp =m * m;
m =tmp;
```

<strong>点乘和叉乘</strong>
基本操作，如下:
```cpp
int main()
{
  Vector3d v(1,2,3);
  Vector3d w(0,1,2);
  cout << "Dot product: " << v.dot(w) << endl;
  double dp = v.adjoint()*w; // automatic conversion of the inner product to a scalar
  cout << "Dot product via a matrix product: " << dp << endl;
  cout << "Cross product:\n" << v.cross(w) << endl;
}
```

<strong>计算一些统计量</strong>  
eigen也支持计算一些统计量，包括 sum(),mean(),prod()等等:
```cpp
int main()
{
  Eigen::Matrix2d mat;
  mat << 1, 2,
         3, 4;
  cout << "Here is mat.sum():       " << mat.sum()       << endl;
  cout << "Here is mat.prod():      " << mat.prod()      << endl;
  cout << "Here is mat.mean():      " << mat.mean()      << endl;
  cout << "Here is mat.minCoeff():  " << mat.minCoeff()  << endl;
  cout << "Here is mat.maxCoeff():  " << mat.maxCoeff()  << endl;
  cout << "Here is mat.trace():     " << mat.trace()     << endl;
}
```