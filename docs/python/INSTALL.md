---
layout: default
title: "Installation (PyQBPP)"
nav_order: 3
---
<div class="lang-en" markdown="1">
# Installation

## Supported Environment
- Linux (Ubuntu 20.04 or later)
- x86_64 or arm64 (aarch64) CPUs
- Python 3.8 or later

## Installation

PyQBPP is available on [PyPI](https://pypi.org/project/pyqbpp/).
For detailed package information, see the [PyQBPP PyPI page](https://pypi.org/project/pyqbpp/).

We recommend using a Python virtual environment (venv) to install PyQBPP.
No sudo privileges are required.

```bash
python3 -m venv ~/qbpp-env
source ~/qbpp-env/bin/activate
pip install pyqbpp
```

To upgrade to the latest version:
```bash
pip install --upgrade pyqbpp
```

## License Activation

After installation, activate the license to start using PyQBPP:

```bash
qbpp-license -k XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

If no license key is set, an **Anonymous Trial** (7 days, 1,000 variables) is activated.

For details on license types, deactivation, troubleshooting, and more, see **[License Management](../LICENSE_MANAGEMENT)**.
</div>

<div class="lang-ja" markdown="1">
# インストール

## 対応環境
- Linux (Ubuntu 20.04以降)
- x86_64またはarm64 (aarch64) CPU
- Python 3.8以降

## インストール

PyQBPPは[PyPI](https://pypi.org/project/pyqbpp/)で公開されています。
パッケージの詳細については、[PyQBPP PyPIページ](https://pypi.org/project/pyqbpp/)を参照してください。

PyQBPPのインストールには、Python仮想環境(venv)の使用を推奨します。
sudo権限は不要です。

```bash
python3 -m venv ~/qbpp-env
source ~/qbpp-env/bin/activate
pip install pyqbpp
```

最新バージョンへのアップグレード:
```bash
pip install --upgrade pyqbpp
```

## ライセンスの有効化

インストール後、ライセンスを有効化してPyQBPPの使用を開始します:

```bash
qbpp-license -k XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

ライセンスキーが設定されていない場合、**匿名トライアル**（7日間、1,000変数）が有効化されます。

ライセンスの種類、無効化、トラブルシューティングなどの詳細については、**[ライセンス管理](../LICENSE_MANAGEMENT)**を参照してください。
</div>
