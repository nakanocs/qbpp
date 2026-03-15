---
layout: default
title: "Installation"
nav_order: 3
---
<div class="lang-en" markdown="1">

# Installation

## Supported Environment

QUBO++ runs on Linux-based systems with the following CPUs:
* **amd64**: 64-bit Intel and AMD processors
* **arm64**: 64-bit ARM processors

Ubuntu 20.04 or later is recommended.
For Windows users, QUBO++ can be used through [WSL (Windows Subsystem for Linux)](WSL).

## Installation

There are two ways to install QUBO++:
- **Method 1: apt (recommended)** — Simple installation with automatic path configuration
- **Method 2: tar.gz** — Manual installation without requiring apt repository setup

## Method 1: Install via apt (recommended)

First, add the QUBO++ apt repository:
```bash
curl -fsSL https://nakanocs.github.io/qbpp-apt/KEY.gpg | sudo gpg --dearmor -o /usr/share/keyrings/qbpp.gpg
echo "deb [signed-by=/usr/share/keyrings/qbpp.gpg] https://nakanocs.github.io/qbpp-apt stable main" | sudo tee /etc/apt/sources.list.d/qbpp.list
```

Then install QUBO++:
```bash
sudo apt update
sudo apt install qbpp
```

This automatically installs headers to `/usr/local/include/qbpp/`, shared libraries to `/usr/local/lib/`, and the `qbpp-license` command to `/usr/local/bin/`.
No environment variable configuration is needed.

The sample programs are installed to `/usr/local/share/qbpp/samples/`.
To compile and run them:
```bash
cp -r /usr/local/share/qbpp/samples ~/qbpp_samples
cd ~/qbpp_samples
make nqueen_easy
```

To upgrade to a new version:
```bash
sudo apt update
sudo apt install --only-upgrade qbpp
```

To uninstall:
```bash
sudo apt remove qbpp
```

## Method 2: Install via tar.gz

Download the `.tar.gz` file of the latest QUBO++ release from the [**Latest Releases**](https://github.com/nakanocs/qbpp/releases/latest) page.
Extract the archive as follows:
```bash
tar xf qbpp_<arch>_<version>.tar.gz
```

### Setting Environment Variables
To compile and run QUBO++ programs, set the following environment variables.
Add these lines to the end of your **`~/.bashrc`** so that they are automatically set when a shell starts:
```bash
export QBPP_PATH=[QUBO++ install directory]

export CPLUS_INCLUDE_PATH=$QBPP_PATH/include:$CPLUS_INCLUDE_PATH
export LIBRARY_PATH=$QBPP_PATH/lib:$LIBRARY_PATH
export LD_LIBRARY_PATH=$QBPP_PATH/lib:$LD_LIBRARY_PATH
export PATH=$QBPP_PATH/bin:$PATH

export QBPP_LICENSE_KEY=[Your QUBO++ license key]
```

## License Activation

After installation, activate the license to start using QUBO++:

```bash
qbpp-license -a
```

If you have a license key, set it before activation:
```bash
export QBPP_LICENSE_KEY=XXXXXX-XXXXXX-XXXXXX-XXXXXX
qbpp-license -a
```

If no license key is set, an **Anonymous Trial** (7 days, 1,000 variables) is activated.

For details on license types, deactivation, troubleshooting, and more, see **[License Management](LICENSE_MANAGEMENT)**.

</div>

<div class="lang-ja" markdown="1">

# インストール

## 対応環境

QUBO++ は以下の CPU を搭載した Linux ベースのシステムで動作します：
* **amd64**: 64ビット Intel および AMD プロセッサ
* **arm64**: 64ビット ARM プロセッサ

Ubuntu 20.04 以降を推奨します。
Windows ユーザーは [WSL (Windows Subsystem for Linux)](WSL) を通じて QUBO++ を使用できます。

## インストール

QUBO++ のインストール方法は2つあります：
- **方法1: apt（推奨）** — パスの自動設定を含むシンプルなインストール
- **方法2: tar.gz** — apt リポジトリの設定が不要な手動インストール

## 方法1: apt によるインストール（推奨）

まず、QUBO++ の apt リポジトリを追加します：
```bash
curl -fsSL https://nakanocs.github.io/qbpp-apt/KEY.gpg | sudo gpg --dearmor -o /usr/share/keyrings/qbpp.gpg
echo "deb [signed-by=/usr/share/keyrings/qbpp.gpg] https://nakanocs.github.io/qbpp-apt stable main" | sudo tee /etc/apt/sources.list.d/qbpp.list
```

次に QUBO++ をインストールします：
```bash
sudo apt update
sudo apt install qbpp
```

これにより、ヘッダファイルが `/usr/local/include/qbpp/` に、共有ライブラリが `/usr/local/lib/` に、`qbpp-license` コマンドが `/usr/local/bin/` に自動的にインストールされます。
環境変数の設定は不要です。

サンプルプログラムは `/usr/local/share/qbpp/samples/` にインストールされます。
コンパイルして実行するには：
```bash
cp -r /usr/local/share/qbpp/samples ~/qbpp_samples
cd ~/qbpp_samples
make nqueen_easy
```

新しいバージョンにアップグレードするには：
```bash
sudo apt update
sudo apt install --only-upgrade qbpp
```

アンインストールするには：
```bash
sudo apt remove qbpp
```

## 方法2: tar.gz によるインストール

[**Latest Releases**](https://github.com/nakanocs/qbpp/releases/latest) ページから最新の QUBO++ リリースの `.tar.gz` ファイルをダウンロードします。
以下のようにアーカイブを展開します：
```bash
tar xf qbpp_<arch>_<version>.tar.gz
```

### 環境変数の設定
QUBO++ プログラムをコンパイル・実行するには、以下の環境変数を設定します。
シェル起動時に自動的に設定されるよう、**`~/.bashrc`** の末尾に以下の行を追加してください：
```bash
export QBPP_PATH=[QUBO++ インストールディレクトリ]

export CPLUS_INCLUDE_PATH=$QBPP_PATH/include:$CPLUS_INCLUDE_PATH
export LIBRARY_PATH=$QBPP_PATH/lib:$LIBRARY_PATH
export LD_LIBRARY_PATH=$QBPP_PATH/lib:$LD_LIBRARY_PATH
export PATH=$QBPP_PATH/bin:$PATH

export QBPP_LICENSE_KEY=[QUBO++ ライセンスキー]
```

## ライセンスのアクティベーション

インストール後、ライセンスをアクティベートして QUBO++ の使用を開始します：

```bash
qbpp-license -a
```

ライセンスキーをお持ちの場合は、アクティベーション前に設定してください：
```bash
export QBPP_LICENSE_KEY=XXXXXX-XXXXXX-XXXXXX-XXXXXX
qbpp-license -a
```

ライセンスキーが設定されていない場合、**Anonymous Trial**（7日間、1,000変数）がアクティベートされます。

ライセンスの種類、ディアクティベーション、トラブルシューティングなどの詳細は **[ライセンス管理](LICENSE_MANAGEMENT)** をご覧ください。

</div>
