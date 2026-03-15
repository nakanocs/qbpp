---
layout: default
title: "Quick Start"
nav_order: 4
---
<div class="lang-en" markdown="1">
# Quick Start

This page provides an overview of the Quick Start procedure for PyQBPP.

## Installation

Install PyQBPP by following the instructions in [**Installation**](INSTALL).

## Create and run a sample program

### Create a PyQBPP sample program
Create a PyQBPP sample program below and save as file **`test.py`**:
```python
from pyqbpp import var_int, ExhaustiveSolver

x = var_int("x", 0, 10)
y = var_int("y", 0, 10)

f = (x + y == 10) + (2 * x + 4 * y == 28)
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
sol = solver.search()

print(f"x = {sol(x)}, y = {sol(y)}")
```

### Run the program
Run `test.py` as follows:
```bash
python3 test.py
x = 6, y = 4
```

## Next steps
1. Activate your license. See [**Installation**](INSTALL) for details.
2. Learn the basics of PyQBPP. Start with **Basics** in [**PyQBPP (Python)**](./).
</div>

<div class="lang-ja" markdown="1">
# クイックスタート

このページでは、PyQBPPのクイックスタート手順の概要を説明します。

## インストール

[**インストール**](INSTALL)の手順に従ってPyQBPPをインストールしてください。

## サンプルプログラムの作成と実行

### PyQBPPサンプルプログラムの作成
以下のPyQBPPサンプルプログラムを作成し、**`test.py`**として保存してください:
```python
from pyqbpp import var_int, ExhaustiveSolver

x = var_int("x", 0, 10)
y = var_int("y", 0, 10)

f = (x + y == 10) + (2 * x + 4 * y == 28)
f.simplify_as_binary()

solver = ExhaustiveSolver(f)
sol = solver.search()

print(f"x = {sol(x)}, y = {sol(y)}")
```

### プログラムの実行
`test.py`を以下のように実行します:
```bash
python3 test.py
x = 6, y = 4
```

## 次のステップ
1. ライセンスを有効化してください。詳細は[**インストール**](INSTALL)を参照してください。
2. PyQBPPの基礎を学びましょう。[**PyQBPP (Python)**](./)の**基礎**から始めてください。
</div>
