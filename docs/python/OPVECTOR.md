---
layout: default
nav_exclude: true
title: "Vector Operations"
nav_order: 12
---
<div class="lang-en" markdown="1">
# Basic Operators and Functions for Vectors
Operators and functions for vectors operate element-wise.

## Basic operators for vectors
The basic operators **`+`**, **`-`**, **`*`**, and **`/`** work for vectors of variables and expressions.
These operators are applied element-wise.

### Vector–Scalar Operations
When you combine a vector and a scalar, the scalar is applied to each element of the vector.

The following program illustrates this behavior:
```python
import pyqbpp as qbpp

x = qbpp.var("x", 3)
f = 2 * x + 1

print("f =", f)
for i in range(len(f)):
    print(f"f[{i}] =", f[i])
```
This program produces the following output:
```
f = {1 +2*x[0],1 +2*x[1],1 +2*x[2]}
f[0] = 1 +2*x[0]
f[1] = 1 +2*x[1]
f[2] = 1 +2*x[2]
```

### Vector–Vector Operations
When you combine two vectors of the same size, the operation is performed element-wise.

```python
import pyqbpp as qbpp

x = qbpp.var("x", 3)
y = qbpp.var("y", 3)
f = 2 * x + 3 * y + 1

print("f =", f)
for i in range(len(f)):
    print(f"f[{i}] =", f[i])
```
This program produces the following output:
```
f = {1 +2*x[0] +3*y[0],1 +2*x[1] +3*y[1],1 +2*x[2] +3*y[2]}
f[0] = 1 +2*x[0] +3*y[0]
f[1] = 1 +2*x[1] +3*y[1]
f[2] = 1 +2*x[2] +3*y[2]
```

## Compound operators for vectors
The compound operators **`+=`**, **`-=`**, **`*=`**, and **`/=`** also work element-wise for vectors:
```python
import pyqbpp as qbpp

x = qbpp.var("x", 3)
y = qbpp.var("y", 3)
f = 6 * x + 4

f += 3 * y
print("f =", f)
f -= 12
print("f =", f)
f *= 2 * y
print("f =", f)
f /= 2
print("f =", f)
```
This program produces the following output:
```
f = {4 +6*x[0] +3*y[0],4 +6*x[1] +3*y[1],4 +6*x[2] +3*y[2]}
f = {-8 +6*x[0] +3*y[0],-8 +6*x[1] +3*y[1],-8 +6*x[2] +3*y[2]}
f = {12*x[0]*y[0] +6*y[0]*y[0] -16*y[0],12*x[1]*y[1] +6*y[1]*y[1] -16*y[1],12*x[2]*y[2] +6*y[2]*y[2] -16*y[2]}
f = {6*x[0]*y[0] +3*y[0]*y[0] -8*y[0],6*x[1]*y[1] +3*y[1]*y[1] -8*y[1],6*x[2]*y[2] +3*y[2]*y[2] -8*y[2]}
```

## Square function for vectors
The square function also works element-wise for vectors:
```python
import pyqbpp as qbpp

x = qbpp.var("x", 3)
f = qbpp.sqr(x + 1)

print("f =", f)
```
This program produces the following output:
```
f = {1 +x[0]*x[0] +x[0] +x[0],1 +x[1]*x[1] +x[1] +x[1],1 +x[2]*x[2] +x[2] +x[2]}
```

## Simplify functions for vectors
Simplify functions also work element-wise for vectors:
```python
import pyqbpp as qbpp

x = qbpp.var("x", 3)
f = qbpp.sqr(x - 1)
print("f =", f)
print("qbpp.simplify(f) =", qbpp.simplify(f))
print("qbpp.simplify_as_binary(f) =", qbpp.simplify_as_binary(f))
print("qbpp.simplify_as_spin(f) =", qbpp.simplify_as_spin(f))
```
This program produces the following output:
```
f = {1 +x[0]*x[0] -x[0] -x[0],1 +x[1]*x[1] -x[1] -x[1],1 +x[2]*x[2] -x[2] -x[2]}
simplify(f) = {1 -2*x[0] +x[0]*x[0],1 -2*x[1] +x[1]*x[1],1 -2*x[2] +x[2]*x[2]}
simplify_as_binary(f) = {1 -x[0],1 -x[1],1 -x[2]}
simplify_as_spin(f) = {2 -2*x[0],2 -2*x[1],2 -2*x[2]}
```

> **NOTE**
> These operators and functions also work for **multi-dimensional arrays**.
</div>

<div class="lang-ja" markdown="1">
# ベクトルの基本演算子と関数
ベクトルの演算子と関数は要素ごとに適用されます。

## ベクトルの基本演算子
基本演算子 **`+`**、**`-`**、**`*`**、**`/`** は変数や式のベクトルに対して使用できます。
これらの演算子は要素ごとに適用されます。

### ベクトルとスカラーの演算
ベクトルとスカラーを組み合わせると、スカラーがベクトルの各要素に適用されます。

以下のプログラムはこの動作を示しています。
```python
import pyqbpp as qbpp

x = qbpp.var("x", 3)
f = 2 * x + 1

print("f =", f)
for i in range(len(f)):
    print(f"f[{i}] =", f[i])
```
このプログラムの出力は以下の通りです。
```
f = {1 +2*x[0],1 +2*x[1],1 +2*x[2]}
f[0] = 1 +2*x[0]
f[1] = 1 +2*x[1]
f[2] = 1 +2*x[2]
```

### ベクトル同士の演算
同じサイズの2つのベクトルを組み合わせると、演算は要素ごとに行われます。

```python
import pyqbpp as qbpp

x = qbpp.var("x", 3)
y = qbpp.var("y", 3)
f = 2 * x + 3 * y + 1

print("f =", f)
for i in range(len(f)):
    print(f"f[{i}] =", f[i])
```
このプログラムの出力は以下の通りです。
```
f = {1 +2*x[0] +3*y[0],1 +2*x[1] +3*y[1],1 +2*x[2] +3*y[2]}
f[0] = 1 +2*x[0] +3*y[0]
f[1] = 1 +2*x[1] +3*y[1]
f[2] = 1 +2*x[2] +3*y[2]
```

## ベクトルの複合演算子
複合演算子 **`+=`**、**`-=`**、**`*=`**、**`/=`** もベクトルに対して要素ごとに適用されます。
```python
import pyqbpp as qbpp

x = qbpp.var("x", 3)
y = qbpp.var("y", 3)
f = 6 * x + 4

f += 3 * y
print("f =", f)
f -= 12
print("f =", f)
f *= 2 * y
print("f =", f)
f /= 2
print("f =", f)
```
このプログラムの出力は以下の通りです。
```
f = {4 +6*x[0] +3*y[0],4 +6*x[1] +3*y[1],4 +6*x[2] +3*y[2]}
f = {-8 +6*x[0] +3*y[0],-8 +6*x[1] +3*y[1],-8 +6*x[2] +3*y[2]}
f = {12*x[0]*y[0] +6*y[0]*y[0] -16*y[0],12*x[1]*y[1] +6*y[1]*y[1] -16*y[1],12*x[2]*y[2] +6*y[2]*y[2] -16*y[2]}
f = {6*x[0]*y[0] +3*y[0]*y[0] -8*y[0],6*x[1]*y[1] +3*y[1]*y[1] -8*y[1],6*x[2]*y[2] +3*y[2]*y[2] -8*y[2]}
```

## ベクトルの二乗関数
二乗関数もベクトルに対して要素ごとに適用されます。
```python
import pyqbpp as qbpp

x = qbpp.var("x", 3)
f = qbpp.sqr(x + 1)

print("f =", f)
```
このプログラムの出力は以下の通りです。
```
f = {1 +x[0]*x[0] +x[0] +x[0],1 +x[1]*x[1] +x[1] +x[1],1 +x[2]*x[2] +x[2] +x[2]}
```

## ベクトルの簡約化関数
簡約化関数もベクトルに対して要素ごとに適用されます。
```python
import pyqbpp as qbpp

x = qbpp.var("x", 3)
f = qbpp.sqr(x - 1)
print("f =", f)
print("qbpp.simplify(f) =", qbpp.simplify(f))
print("qbpp.simplify_as_binary(f) =", qbpp.simplify_as_binary(f))
print("qbpp.simplify_as_spin(f) =", qbpp.simplify_as_spin(f))
```
このプログラムの出力は以下の通りです。
```
f = {1 +x[0]*x[0] -x[0] -x[0],1 +x[1]*x[1] -x[1] -x[1],1 +x[2]*x[2] -x[2] -x[2]}
simplify(f) = {1 -2*x[0] +x[0]*x[0],1 -2*x[1] +x[1]*x[1],1 -2*x[2] +x[2]*x[2]}
simplify_as_binary(f) = {1 -x[0],1 -x[1],1 -x[2]}
simplify_as_spin(f) = {2 -2*x[0],2 -2*x[1],2 -2*x[2]}
```

> **NOTE**
> これらの演算子と関数は**多次元配列**に対しても使用できます。
</div>
