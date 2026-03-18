---
layout: default
title: "License Management"
nav_order: 4
---
<div class="lang-en" markdown="1">

# License Management

This page describes how to manage your QUBO++ license using the **`qbpp-license`** command-line utility.
The license system is shared between **QUBO++ (C++)** and **PyQBPP (Python)**.

> **Important**: Keep your license key confidential. Do not share it with unauthorized users, post it in public repositories, or include it in unencrypted configuration files. In CI/CD environments, store the key as a secret environment variable.

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

Licenses are available in two forms: **Node-locked** and **Floating**. The license key format determines the type — floating license keys start with `F-`.

## Node-locked Licenses

Node-locked licenses are tied to specific machines. Each license key has a limited number of allowed activations (machines).

### Activating

You must activate and cache the key by running the following command once per machine:

```bash
qbpp-license -k XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

The license key is encrypted and cached locally. After activation, no environment variable or `-k` option is needed for subsequent runs — QUBO++ programs automatically use the cached key.

To re-activate or change the key, simply run the command again with a new key.

> **Note on validity periods**: For licenses with a limited validity period (e.g., 30 days), the countdown begins at the time of activation (`qbpp-license -k KEY -a`), not when the program is first run.

> **Note**: For Anonymous Trial, no license key is needed. Simply run `qbpp-license -a` without setting a key.

### Checking Status

To display the current license status:

```bash
qbpp-license
```

This shows the license type, expiry date, variable limits, and activation usage. The command contacts the license server to refresh the status.

### Deactivating

To move a license to another machine, first deactivate it on the current machine:

```bash
qbpp-license -d
```

- Deactivation frees up one activation slot.
- **Anonymous Trial** licenses cannot be deactivated.
- There is a **24-hour cooldown** between consecutive deactivations to prevent abuse.

## Floating Licenses

Floating licenses allow shared access across multiple machines within an organization. Unlike node-locked licenses that are tied to specific machines, floating licenses use a **lease-based** mechanism — any machine can use the license, but only one at a time. Floating license keys start with `F-` (e.g., `F-QBPP12-ABCDEF-GHIJKL-MNOPQR`).

**No activation or deactivation is needed.** Simply register the key, and QUBO++ programs handle checkout/checkin automatically.

> **Note on validity periods**: For floating licenses with a limited validity period (e.g., 30 days), the countdown begins when a QUBO++ program first uses the license (first checkout) — not when the key is registered with `qbpp-license -k F-KEY -a`.

### Setting Up

Cache the key locally with `-a`:

```bash
qbpp-license -k F-XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

Or use the environment variable (useful for Docker/CI):

```bash
export QBPP_LICENSE_KEY=F-XXXXXX-XXXXXX-XXXXXX-XXXXXX
```

To check the license information without caching:

```bash
qbpp-license -k F-XXXXXX-XXXXXX-XXXXXX-XXXXXX
```

### Checking Session Status

Running `qbpp-license` displays the current floating session status:

- **In Use**: The license is currently checked out by a program. The IP address, hostname, and user of the machine holding the lease are displayed.
- **Idle**: The license is not in use and available for checkout.
- **Cooldown**: The lease has been released but the cooldown period has not yet expired. The remaining time, IP address, hostname, and user of the last session are displayed. The same machine can still use the license during cooldown.

### How It Works

- When a QUBO++ program starts, it automatically acquires a lease from the license server.
- The lease is automatically renewed while the program is running.
- When the program exits, the lease is released, making the slot available for other machines.
- If the program crashes or the network is lost, the lease expires automatically after a timeout period.
- Only one program can use a floating license at a time. If the license is in use, other programs will run in **Fallback Mode** (100-variable limit) until the lease is released or expires.

## Common

### Key Priority

When multiple methods are used, the following priority applies:

1. **`-k` argument** or `qbpp::license_key()` in code (highest)
2. **`QBPP_LICENSE_KEY` environment variable**
3. **Cached key** (lowest)

### Command Reference

```
Usage: qbpp-license [options]

Options:
  -h, --help         Show help message and exit
  -k, --key KEY      Specify a license key (info display only, does not cache)
  -a, --activate     Activate and cache the license key locally
  -d, --deactivate   Deactivate the license on this machine (node-locked only)
  -t, --time-out SEC Set the server communication timeout (default: 20 seconds)
```

#### Examples

| Command | Description |
|---|---|
| `qbpp-license` | Display current license status |
| `qbpp-license -k KEY` | Show license info (no caching) |
| `qbpp-license -a` | Activate Anonymous Trial |
| `qbpp-license -k KEY -a` | Activate and cache a node-locked key |
| `qbpp-license -k F-KEY -a` | Cache a floating key |
| `qbpp-license -d` | Deactivate node-locked license |
| `qbpp-license -t 60` | Check status with a 60-second timeout |

### How License Verification Works

- **`qbpp-license` command**: Always contacts the license server to get the latest status. This may take a few seconds depending on network conditions.
- **QUBO++ programs (node-locked)**: Verify the license using the **local cache** and do not block on server communication. The server is contacted only when the cache needs to be refreshed.
- **QUBO++ programs (floating)**: Contact the license server on every run to acquire/release a lease. A stable network connection is required.
- **License key storage**: When a license is activated with `-a` (node-locked or floating), the key is encrypted and cached locally. The `-k` option alone only displays information without caching.

### Network and Timeout

If your network is slow or behind a firewall/proxy, the default 20-second timeout may not be enough.

To increase the timeout:

```bash
qbpp-license -t 60 -a
```

If the server is unreachable, QUBO++ falls back to the cached license status. If no cache exists, QUBO++ runs in **Fallback Mode** (100-variable limit).

### Troubleshooting

#### Programs work but with limited variables

- QUBO++ may be running in Fallback Mode. Check the license status:
  ```bash
  $ qbpp-license
  ```
- **Node-locked**: Ensure the license key is set correctly via `qbpp-license -k KEY -a` or `QBPP_LICENSE_KEY`. Re-activate if necessary: `qbpp-license -a`
- **Floating**: Ensure the license key is cached via `qbpp-license -k F-KEY -a` or `QBPP_LICENSE_KEY`. Check that the license is not in use by another program.

#### "Deactivation cooldown" (node-locked only)

- There is a 24-hour waiting period between consecutive deactivations.
- Wait and try again after the cooldown period has elapsed.

</div>

<div class="lang-ja" markdown="1">

# ライセンス管理

このページでは、**`qbpp-license`** コマンドラインユーティリティを使用した QUBO++ ライセンスの管理方法を説明します。
ライセンスシステムは **QUBO++ (C++)** と **PyQBPP (Python)** で共通です。

> **重要**: ライセンスキーは厳重に管理してください。許可されていないユーザーとの共有、公開リポジトリへの掲載、暗号化されていない設定ファイルへの記載は行わないでください。CI/CD 環境ではシークレット環境変数として設定してください。

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

ライセンスには**ノードロック**と**フローティング**の2つの形態があります。ライセンスキーの形式で種類が決まります — フローティングライセンスキーは `F-` で始まります。

## ノードロックライセンス

ノードロックライセンスは特定のマシンに紐付けられます。各ライセンスキーには許可されたアクティベーション数（マシン数）の上限があります。

### アクティベーション

マシンごとに以下のコマンドを一度実行して、アクティベートとキーのキャッシュを行う必要があります：

```bash
qbpp-license -k XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

ライセンスキーは暗号化されてローカルにキャッシュされます。アクティベーション後、以降の実行では環境変数や `-k` オプションは不要です — QUBO++ プログラムはキャッシュされたキーを自動的に使用します。

再アクティベーションやキーの変更は、新しいキーでコマンドを再度実行するだけです。

> **有効期間について**: 有効期間が設定されたライセンス（例: 30日間）のカウントダウンは、アクティベーション（`qbpp-license -k KEY -a`）の実行時点から開始されます。プログラムの初回実行時点ではありません。

> **注意**: Anonymous Trial の場合、ライセンスキーは不要です。キーを設定せずに `qbpp-license -a` を実行するだけです。

### ライセンス状態の確認

現在のライセンス状態を表示するには：

```bash
qbpp-license
```

ライセンスタイプ、有効期限、変数数の上限、アクティベーション使用状況が表示されます。このコマンドはライセンスサーバーに接続して状態を更新します。

### ディアクティベーション

ライセンスを別のマシンに移動するには、まず現在のマシンでディアクティベートします：

```bash
qbpp-license -d
```

- ディアクティベーションによりアクティベーション枠が1つ解放されます。
- **Anonymous Trial** ライセンスはディアクティベートできません。
- 悪用防止のため、連続するディアクティベーション間には **24時間のクールダウン** があります。

## フローティングライセンス

フローティングライセンスは、組織内の複数のマシン間での共有アクセスを可能にします。ノードロックライセンスが特定のマシンに紐付けられるのに対し、フローティングライセンスは**リースベース**の仕組みを使用します — どのマシンからでも使用できますが、同時に使用できるのは1台のみです。フローティングライセンスキーは `F-` で始まります（例: `F-QBPP12-ABCDEF-GHIJKL-MNOPQR`）。

**アクティベーションやディアクティベーションは不要です。** キーを登録するだけで、QUBO++ プログラムが自動的にチェックアウト/チェックインを行います。

> **有効期間について**: 有効期間が設定されたフローティングライセンス（例: 30日間）のカウントダウンは、プログラムで QUBO++ の機能を最初に使用した時点（初回チェックアウト）から開始されます。`qbpp-license -k F-KEY -a` でキーを登録した時点ではありません。

### セットアップ

`-a` オプションを付けてキーをローカルにキャッシュします：

```bash
qbpp-license -k F-XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

環境変数を使用することもできます（Docker/CI 環境で便利）：

```bash
export QBPP_LICENSE_KEY=F-XXXXXX-XXXXXX-XXXXXX-XXXXXX
```

キーをキャッシュせずに情報のみ表示するには：

```bash
qbpp-license -k F-XXXXXX-XXXXXX-XXXXXX-XXXXXX
```

### セッション状態の確認

`qbpp-license` を実行すると、フローティングライセンスの現在のセッション状態が表示されます：

- **In Use**: ライセンスは現在プログラムによってチェックアウトされています。リースを保持しているマシンの IP アドレス、ホスト名、ユーザー名が表示されます。
- **Idle**: ライセンスは使用されておらず、チェックアウト可能な状態です。
- **Cooldown**: リースは解放されましたが、クールダウン期間がまだ終了していません。残り時間、最後のセッションの IP アドレス、ホスト名、ユーザー名が表示されます。クールダウン中でも同じマシンからはライセンスを使用できます。

### 仕組み

- QUBO++ プログラムの起動時に、ライセンスサーバーから自動的にリースを取得します。
- プログラムの実行中、リースは自動的に更新されます。
- プログラムの終了時にリースが解放され、他のマシンがそのスロットを使用できるようになります。
- プログラムがクラッシュしたりネットワークが切断された場合、リースはタイムアウト期間後に自動的に失効します。
- フローティングライセンスを同時に使用できるのは1つのプログラムのみです。ライセンスが使用中の場合、他のプログラムはリースが解放または失効するまで **Fallback Mode**（100変数制限）で動作します。

## 共通事項

### キーの優先順位

複数の方法が使用されている場合、以下の優先順位が適用されます：

1. **`-k` 引数** またはコード内の `qbpp::license_key()`（最高優先）
2. **`QBPP_LICENSE_KEY` 環境変数**
3. **キャッシュされたキー**（最低優先）

### コマンドリファレンス

```
Usage: qbpp-license [options]

Options:
  -h, --help         ヘルプメッセージを表示して終了
  -k, --key KEY      ライセンスキーを指定（情報表示のみ、キャッシュしない）
  -a, --activate     ライセンスキーをアクティベートしてローカルにキャッシュ
  -d, --deactivate   このマシンでライセンスをディアクティベート（ノードロックのみ）
  -t, --time-out SEC サーバー通信のタイムアウトを設定（デフォルト: 20秒）
```

#### 使用例

| コマンド | 説明 |
|---|---|
| `qbpp-license` | 現在のライセンス状態を表示 |
| `qbpp-license -k KEY` | ライセンス情報を表示（キャッシュしない） |
| `qbpp-license -a` | Anonymous Trial をアクティベート |
| `qbpp-license -k KEY -a` | ノードロックキーをアクティベートしてキャッシュ |
| `qbpp-license -k F-KEY -a` | フローティングキーをキャッシュ |
| `qbpp-license -d` | ノードロックライセンスをディアクティベート |
| `qbpp-license -t 60` | 60秒のタイムアウトで状態を確認 |

### ライセンス認証の仕組み

- **`qbpp-license` コマンド**: 常にライセンスサーバーに接続して最新の状態を取得します。ネットワーク状況によっては数秒かかることがあります。
- **QUBO++ プログラム（ノードロック）**: **ローカルキャッシュ**を使用してライセンスを検証し、サーバー通信でブロックしません。サーバーへの接続はキャッシュの更新が必要な場合のみ行われます。
- **QUBO++ プログラム（フローティング）**: 毎回ライセンスサーバーに接続してリースの取得/解放を行います。安定したネットワーク接続が必要です。
- **ライセンスキーの保存**: `-a` オプション付きでライセンスがアクティベート（ノードロックまたはフローティング）されると、キーは暗号化されてローカルにキャッシュされます。`-k` オプションのみでは情報の表示のみでキャッシュは行われません。

### ネットワークとタイムアウト

ネットワークが遅い場合やファイアウォール/プロキシの背後にある場合、デフォルトの20秒のタイムアウトでは不十分な場合があります。

タイムアウトを延長するには：

```bash
qbpp-license -t 60 -a
```

サーバーに到達できない場合、QUBO++ はキャッシュされたライセンス状態にフォールバックします。キャッシュが存在しない場合、QUBO++ は **Fallback Mode**（100変数制限）で動作します。

### トラブルシューティング

#### プログラムは動作するが変数数が制限される

- QUBO++ が Fallback Mode で動作している可能性があります。ライセンス状態を確認してください：
  ```bash
  $ qbpp-license
  ```
- **ノードロック**: `qbpp-license -k KEY -a` または `QBPP_LICENSE_KEY` でライセンスキーが正しく設定されているか確認してください。必要に応じて再アクティベート：`qbpp-license -a`
- **フローティング**: `qbpp-license -k F-KEY -a` または `QBPP_LICENSE_KEY` でライセンスキーが登録されているか確認してください。ライセンスが他のプログラムで使用中でないか確認してください。

#### 「ディアクティベーションのクールダウン」（ノードロックのみ）

- 連続するディアクティベーション間には24時間の待機期間があります。
- クールダウン期間が経過してから再試行してください。

</div>
