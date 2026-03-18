---
layout: default
title: "License Management"
nav_order: 4
---
<div class="lang-en" markdown="1">

# License Management

This page describes how to manage your QUBO++ license using the **`qbpp-license`** command-line utility.
The license system is shared between **QUBO++ (C++)** and **PyQBPP (Python)**.

## License Types

QUBO++ offers several license types with different variable count limits and validity periods.

| License Type | Key Required | Validity | CPU Variables | GPU Variables |
|---|---|---|---|---|
| **Anonymous Trial** | No | 7 days | 1,000 | 1,000 |
| **Registered Trial** | Yes | 30 days | 10,000 | 10,000 |
| **Standard** | Yes | Agreement term | 2,147,483,647 | 1,000 |
| **Professional** | Yes | Agreement term | 2,147,483,647 | 2,147,483,647 |
| **Fallback** | N/A | Always | 100 | 100 |

- **Anonymous Trial**: No registration required. Automatically activated on first use.
- **Registered Trial**: Requires a free license key for evaluation purposes.
- **Standard License**: For production use. Supports large-scale CPU optimization.
- **Professional License**: For production use with GPU acceleration (ABS3 Solver, Exhaustive Solver).
- **Fallback Mode**: When no valid license is found or the license has expired, QUBO++ runs with a 100-variable limit.

## Setting and Activating the License Key

If you have a license key, set it using one of the following methods.

### Method 1: Activate with `qbpp-license` (recommended)

Run the following command once per machine:

```bash
qbpp-license -k XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

The license key is encrypted and cached locally. After activation, no environment variable or `-k` option is needed for subsequent runs — QUBO++ programs automatically use the cached key.

To re-activate or change the key, simply run the command again with a new key.

### Method 2: Environment Variable

Set the `QBPP_LICENSE_KEY` environment variable. This is useful for Docker/CI environments where the local cache is not persistent.

```bash
export QBPP_LICENSE_KEY=XXXXXX-XXXXXX-XXXXXX-XXXXXX
```

### Priority

When multiple methods are used, the following priority applies:

1. **`-k` argument** or `qbpp::license_key()` in code (highest)
2. **`QBPP_LICENSE_KEY` environment variable**
3. **Cached key** (lowest)

> **Note**: For Anonymous Trial, no license key is needed. Simply run `qbpp-license -a` without setting a key.

## Checking License Status

To display the current license status without making any changes:

```bash
qbpp-license
```

This shows the license type, expiry date, variable limits, and activation usage. The command contacts the license server to refresh the status.

## Deactivating a License

To move a license to another machine, first deactivate it on the current machine:

```bash
qbpp-license -d
```

- Each license key has a limited number of allowed activations.
- Deactivation frees up one activation slot.
- **Anonymous Trial** licenses cannot be deactivated.
- There is a **24-hour cooldown** between consecutive deactivations to prevent abuse.


## Command Reference

```
Usage: qbpp-license [options]

Options:
  -h, --help         Show help message and exit
  -k, --key KEY      Specify a license key
  -a, --activate     Activate the license on this machine
  -d, --deactivate   Deactivate the license on this machine
  -t, --time-out SEC Set the server communication timeout (default: 20 seconds)
```

### Examples

| Command | Description |
|---|---|
| `qbpp-license` | Display current license status |
| `qbpp-license -a` | Activate (Anonymous Trial if no key is set) |
| `qbpp-license -k KEY -a` | Activate with a specific key |
| `qbpp-license -d` | Deactivate the license on this machine |
| `qbpp-license -t 60` | Check status with a 60-second timeout |
| `qbpp-license -k KEY -t 60 -a` | Activate with a key and extended timeout |

## How License Verification Works

- **`qbpp-license` command**: Always contacts the license server to get the latest status. This may take a few seconds depending on network conditions.
- **QUBO++ programs**: Verify the license using the **local cache** and do not block on server communication. The server is contacted only when the cache needs to be refreshed (e.g., after a long period without synchronization).
- **License key storage**: When a license is activated, the key is encrypted and cached locally. This allows subsequent runs without setting the key again.


## Network and Timeout

If your network is slow or behind a firewall/proxy, the default 20-second timeout may not be enough.

To increase the timeout:

```bash
qbpp-license -t 60 -a
```

If the server is unreachable, QUBO++ falls back to the cached license status. If no cache exists, QUBO++ runs in **Fallback Mode** (100-variable limit).

### Programs work but with limited variables

- QUBO++ may be running in Fallback Mode. Check the license status:
  ```bash
  $ qbpp-license
  ```
- Ensure the license key is set correctly via `qbpp-license -k KEY -a` or `QBPP_LICENSE_KEY`.
- Re-activate if necessary: `qbpp-license -a`

### "Deactivation cooldown"

- There is a 24-hour waiting period between consecutive deactivations.
- Wait and try again after the cooldown period has elapsed.

## Floating Licenses

Floating licenses allow shared access across multiple machines within an organization. Instead of being permanently locked to a machine, a floating license uses a **lease-based** mechanism.

- When a QUBO++ program starts, it acquires a lease from the license server.
- The lease is automatically renewed while the program is running.
- When the program exits, the lease is released, making the slot available for other machines.
- If the program crashes or the network is lost, the lease expires automatically after a timeout period.

Usage is the same as node-locked licenses:

```bash
qbpp-license -k F-XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

The floating license key is also cached locally, so subsequent runs do not require setting the key again.

</div>

<div class="lang-ja" markdown="1">

# ライセンス管理

このページでは、**`qbpp-license`** コマンドラインユーティリティを使用した QUBO++ ライセンスの管理方法を説明します。
ライセンスシステムは **QUBO++ (C++)** と **PyQBPP (Python)** で共通です。

## ライセンスの種類

QUBO++ では、変数数の上限と有効期間が異なる複数のライセンスタイプが用意されています。

| ライセンスタイプ | キー必要 | 有効期間 | CPU 変数数 | GPU 変数数 |
|---|---|---|---|---|
| **Anonymous Trial** | 不要 | 7日間 | 1,000 | 1,000 |
| **Registered Trial** | 必要 | 30日間 | 10,000 | 10,000 |
| **Standard** | 必要 | 契約期間 | 2,147,483,647 | 1,000 |
| **Professional** | 必要 | 契約期間 | 2,147,483,647 | 2,147,483,647 |
| **Fallback** | N/A | 常時 | 100 | 100 |

- **Anonymous Trial**: 登録不要。初回使用時に自動的にアクティベートされます。
- **Registered Trial**: 評価目的の無料ライセンスキーが必要です。
- **Standard License**: 本番利用向け。大規模な CPU 最適化をサポートします。
- **Professional License**: GPU アクセラレーション（ABS3 Solver、Exhaustive Solver）を使用した本番利用向けです。
- **Fallback Mode**: 有効なライセンスが見つからない場合やライセンスが期限切れの場合、QUBO++ は100変数の制限で動作します。

## ライセンスキーの設定とアクティベーション

ライセンスキーをお持ちの場合は、以下のいずれかの方法で設定してください。

### 方法1: `qbpp-license` でアクティベート（推奨）

マシンごとに以下のコマンドを一度実行します：

```bash
qbpp-license -k XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

ライセンスキーは暗号化されてローカルにキャッシュされます。アクティベーション後、以降の実行では環境変数や `-k` オプションは不要です — QUBO++ プログラムはキャッシュされたキーを自動的に使用します。

再アクティベーションやキーの変更は、新しいキーでコマンドを再度実行するだけです。

### 方法2: 環境変数

`QBPP_LICENSE_KEY` 環境変数を設定します。ローカルキャッシュが永続的でない Docker/CI 環境で便利です。

```bash
export QBPP_LICENSE_KEY=XXXXXX-XXXXXX-XXXXXX-XXXXXX
```

### 優先順位

複数の方法が使用されている場合、以下の優先順位が適用されます：

1. **`-k` 引数** またはコード内の `qbpp::license_key()`（最高優先）
2. **`QBPP_LICENSE_KEY` 環境変数**
3. **キャッシュされたキー**（最低優先）

> **注意**: Anonymous Trial の場合、ライセンスキーは不要です。キーを設定せずに `qbpp-license -a` を実行するだけです。

## ライセンス状態の確認

変更を加えずに現在のライセンス状態を表示するには：

```bash
qbpp-license
```

ライセンスタイプ、有効期限、変数数の上限、アクティベーション使用状況が表示されます。このコマンドはライセンスサーバーに接続して状態を更新します。

## ライセンスのディアクティベーション

ライセンスを別のマシンに移動するには、まず現在のマシンでディアクティベートします：

```bash
qbpp-license -d
```

- 各ライセンスキーには許可されたアクティベーション数の上限があります。
- ディアクティベーションによりアクティベーション枠が1つ解放されます。
- **Anonymous Trial** ライセンスはディアクティベートできません。
- 悪用防止のため、連続するディアクティベーション間には **24時間のクールダウン** があります。


## コマンドリファレンス

```
Usage: qbpp-license [options]

Options:
  -h, --help         ヘルプメッセージを表示して終了
  -k, --key KEY      ライセンスキーを指定
  -a, --activate     このマシンでライセンスをアクティベート
  -d, --deactivate   このマシンでライセンスをディアクティベート
  -t, --time-out SEC サーバー通信のタイムアウトを設定（デフォルト: 20秒）
```

### 使用例

| コマンド | 説明 |
|---|---|
| `qbpp-license` | 現在のライセンス状態を表示 |
| `qbpp-license -a` | アクティベート（キー未設定時は Anonymous Trial） |
| `qbpp-license -k KEY -a` | 指定したキーでアクティベート |
| `qbpp-license -d` | このマシンでライセンスをディアクティベート |
| `qbpp-license -t 60` | 60秒のタイムアウトで状態を確認 |
| `qbpp-license -k KEY -t 60 -a` | キーと延長タイムアウトでアクティベート |

## ライセンス認証の仕組み

- **`qbpp-license` コマンド**: 常にライセンスサーバーに接続して最新の状態を取得します。ネットワーク状況によっては数秒かかることがあります。
- **QUBO++ プログラム**: **ローカルキャッシュ**を使用してライセンスを検証し、サーバー通信でブロックしません。サーバーへの接続はキャッシュの更新が必要な場合（例：長期間同期されていない場合）のみ行われます。
- **ライセンスキーの保存**: ライセンスがアクティベートされると、キーは暗号化されてローカルにキャッシュされます。これにより、キーを再設定せずに以降の実行が可能になります。


## ネットワークとタイムアウト

ネットワークが遅い場合やファイアウォール/プロキシの背後にある場合、デフォルトの20秒のタイムアウトでは不十分な場合があります。

タイムアウトを延長するには：

```bash
qbpp-license -t 60 -a
```

サーバーに到達できない場合、QUBO++ はキャッシュされたライセンス状態にフォールバックします。キャッシュが存在しない場合、QUBO++ は **Fallback Mode**（100変数制限）で動作します。

### プログラムは動作するが変数数が制限される

- QUBO++ が Fallback Mode で動作している可能性があります。ライセンス状態を確認してください：
  ```bash
  $ qbpp-license
  ```
- `qbpp-license -k KEY -a` または `QBPP_LICENSE_KEY` でライセンスキーが正しく設定されているか確認してください。
- 必要に応じて再アクティベートしてください：`qbpp-license -a`

### 「ディアクティベーションのクールダウン」

- 連続するディアクティベーション間には24時間の待機期間があります。
- クールダウン期間が経過してから再試行してください。

## フローティングライセンス

フローティングライセンスは、組織内の複数のマシン間での共有アクセスを可能にします。マシンに永続的にロックされる代わりに、フローティングライセンスは**リースベース**の仕組みを使用します。

- QUBO++ プログラムの起動時に、ライセンスサーバーからリースを取得します。
- プログラムの実行中、リースは自動的に更新されます。
- プログラムの終了時にリースが解放され、他のマシンがそのスロットを使用できるようになります。
- プログラムがクラッシュしたりネットワークが切断された場合、リースはタイムアウト期間後に自動的に失効します。

使い方はノードロックライセンスと同じです：

```bash
qbpp-license -k F-XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

フローティングライセンスキーもローカルにキャッシュされるため、以降の実行ではキーの再設定は不要です。

</div>
