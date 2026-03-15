---
layout: default
title: "Quick Start"
nav_order: 4
---
<div class="lang-en" markdown="1">
# Quick Start

> **Try without installing:** You can try PyQBPP immediately in the browser using the [**Playground**](../PLAYGROUND) — no installation required.

This page provides an overview of the Quick Start procedure for PyQBPP.

## Installation

Install PyQBPP by following the instructions in [**Installation**](INSTALL).

## Create and run a sample program

### Create a PyQBPP sample program
Create a PyQBPP sample program below and save as file **`test.py`**:
```python
from pyqbpp import between, var_int, ExhaustiveSolver

x = between(var_int("x"), 0, 10)
y = between(var_int("y"), 0, 10)

f = x + y == 10
g = 2 * x + 4 * y == 28
h = f + g
h.simplify_as_binary()
print(f"h = {h}")

solver = ExhaustiveSolver(h)
sol = solver.search()
print(f"sol = {sol}")
print(f"x = {sol(x)}, y = {sol(y)}")
```

### Run the program
Run `test.py` as follows:
```bash
python3 test.py
h = 884 -127*x[0] -244*x[1] -448*x[2] -351*x[3] -227*y[0] -420*y[1] -704*y[2] -579*y[3] +20*x[0]*x[1] +40*x[0]*x[2] +30*x[0]*x[3] +18*x[0]*y[0] +36*x[0]*y[1] +72*x[0]*y[2] +54*x[0]*y[3] +80*x[1]*x[2] +60*x[1]*x[3] +36*x[1]*y[0] +72*x[1]*y[1] +144*x[1]*y[2] +108*x[1]*y[3] +120*x[2]*x[3] +72*x[2]*y[0] +144*x[2]*y[1] +288*x[2]*y[2] +216*x[2]*y[3] +54*x[3]*y[0] +108*x[3]*y[1] +216*x[3]*y[2] +162*x[3]*y[3] +68*y[0]*y[1] +136*y[0]*y[2] +102*y[0]*y[3] +272*y[1]*y[2] +204*y[1]*y[3] +408*y[2]*y[3]
sol = Sol(energy=0, x[0]=1, x[1]=1, x[2]=0, x[3]=1, y[0]=0, y[1]=0, y[2]=1, y[3]=0)
x = 6, y = 4
```

## Next steps
1. Activate your license. See [**Installation**](INSTALL) for details.
2. Learn the basics of PyQBPP. Start with **Basics** in [**PyQBPP (Python)**](./).
</div>

<div class="lang-ja" markdown="1">
# クイックスタート

> **インストール不要で試す:** [**Playground**](../PLAYGROUND) を使えば、ブラウザ上ですぐに PyQBPP を試すことができます。

このページでは、PyQBPPのクイックスタート手順の概要を説明します。

## インストール

[**インストール**](INSTALL)の手順に従ってPyQBPPをインストールしてください。

## サンプルプログラムの作成と実行

### PyQBPPサンプルプログラムの作成
以下のPyQBPPサンプルプログラムを作成し、**`test.py`**として保存してください:
```python
from pyqbpp import between, var_int, ExhaustiveSolver

x = between(var_int("x"), 0, 10)
y = between(var_int("y"), 0, 10)

f = x + y == 10
g = 2 * x + 4 * y == 28
h = f + g
h.simplify_as_binary()
print(f"h = {h}")

solver = ExhaustiveSolver(h)
sol = solver.search()
print(f"sol = {sol}")
print(f"x = {sol(x)}, y = {sol(y)}")
```

### プログラムの実行
`test.py`を以下のように実行します:
```bash
python3 test.py
h = 884 -127*x[0] -244*x[1] -448*x[2] -351*x[3] -227*y[0] -420*y[1] -704*y[2] -579*y[3] +20*x[0]*x[1] +40*x[0]*x[2] +30*x[0]*x[3] +18*x[0]*y[0] +36*x[0]*y[1] +72*x[0]*y[2] +54*x[0]*y[3] +80*x[1]*x[2] +60*x[1]*x[3] +36*x[1]*y[0] +72*x[1]*y[1] +144*x[1]*y[2] +108*x[1]*y[3] +120*x[2]*x[3] +72*x[2]*y[0] +144*x[2]*y[1] +288*x[2]*y[2] +216*x[2]*y[3] +54*x[3]*y[0] +108*x[3]*y[1] +216*x[3]*y[2] +162*x[3]*y[3] +68*y[0]*y[1] +136*y[0]*y[2] +102*y[0]*y[3] +272*y[1]*y[2] +204*y[1]*y[3] +408*y[2]*y[3]
sol = Sol(energy=0, x[0]=1, x[1]=1, x[2]=0, x[3]=1, y[0]=0, y[1]=0, y[2]=1, y[3]=0)
x = 6, y = 4
```

## 次のステップ
1. ライセンスを有効化してください。詳細は[**インストール**](INSTALL)を参照してください。
2. PyQBPPの基礎を学びましょう。[**PyQBPP (Python)**](./)の**基礎**から始めてください。
</div>
