---
layout: default
title: "License Management"
nav_order: 9
---
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

## Setting the License Key

If you have a license key, set it using one of the following methods.

### Method 1: Environment Variable (recommended)

Add the following line to your `~/.bashrc`:

```bash
export QBPP_LICENSE_KEY=XXXXXX-XXXXXX-XXXXXX-XXXXXX
```

Then reload the shell:
```bash
$ source ~/.bashrc
```

### Method 2: Command-Line Option

Pass the key directly to `qbpp-license`:

```bash
$ qbpp-license -k XXXXXX-XXXXXX-XXXXXX-XXXXXX -a
```

### Method 3: In C++ Code

Set the key programmatically before creating variables:

```cpp
#include <qbpp/qbpp.hpp>

int main() {
    qbpp::license_key("XXXXXX-XXXXXX-XXXXXX-XXXXXX");
    // ... your QUBO++ code ...
}
```

> **Note**: For Anonymous Trial, no license key is needed. Simply run `qbpp-license -a` without setting a key.

## Activating a License

A license must be activated on each machine before QUBO++ programs can use it. Activation is **node-locked** -- the license is tied to the specific machine.

```bash
$ qbpp-license -a
```

- This contacts the license server and registers the current machine.
- Each activation consumes one slot from the license's activation limit.
- If the license is already activated on this machine, running `-a` again does **not** consume an additional slot.

## Checking License Status

To display the current license status without making any changes:

```bash
$ qbpp-license
```

This shows the license type, expiry date, variable limits, and activation usage. The command contacts the license server to refresh the status.

## Deactivating a License

To move a license to another machine, first deactivate it on the current machine:

```bash
$ qbpp-license -d
```

- Deactivation frees up one activation slot.
- **Anonymous Trial** licenses cannot be deactivated.
- There is a **24-hour cooldown** between consecutive deactivations to prevent abuse.

> **Warning**: Each license key has a limited number of allowed activations and deactivations. Once the total reaches this limit, you will no longer be able to activate or deactivate that license. Plan your activations carefully.

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
- **QUBO++ programs**: Verify the license using the **local cache** (`~/.qbpp/license_cache`) and do not block on server communication. The server is contacted only when the cache needs to be refreshed (e.g., after a long period without synchronization).
- **Offline use**: Once activated, the license works offline using the cached credentials. No persistent internet connection is required for running QUBO++ programs.

## Network and Timeout

If your network is slow or behind a firewall/proxy, the default 20-second timeout may not be enough.

To increase the timeout:

```bash
$ qbpp-license -t 60 -a
```

If the server is unreachable, QUBO++ falls back to the cached license status. If no cache exists, QUBO++ runs in **Fallback Mode** (100-variable limit).

## Troubleshooting

### "License server timed out"

- Check your internet connection.
- Increase the timeout: `qbpp-license -t 60`
- If you are behind a corporate firewall or proxy, ensure outbound HTTPS traffic is allowed.

### "Activation limit reached"

- Each license has a limited number of activation/deactivation slots.
- Deactivate on machines you no longer use: `qbpp-license -d`
- Contact the license administrator to increase the activation limit if needed.

### "License expired"

- Trial licenses have a fixed validity period (7 days for Anonymous, 30 days for Registered).
- Contact the distributor to renew or upgrade your license.
- QUBO++ continues to work in Fallback Mode (100-variable limit) after expiry.

### Programs work but with limited variables

- QUBO++ may be running in Fallback Mode. Check the license status:
  ```bash
  $ qbpp-license
  ```
- Ensure the license key is set correctly via `QBPP_LICENSE_KEY` or `qbpp::license_key()`.
- Re-activate if necessary: `qbpp-license -a`

### "Deactivation cooldown"

- There is a 24-hour waiting period between consecutive deactivations.
- Wait and try again after the cooldown period has elapsed.

## Floating Licenses

Floating licenses allow shared access across multiple machines within an organization. Instead of being permanently locked to a machine, a floating license uses a **lease-based** mechanism.

- A floating license key is prefixed with `F-` (e.g., `F-XXXXXX-XXXXXX-XXXXXX-XXXXXX`).
- When a QUBO++ program starts, it acquires a lease from the license server.
- The lease is automatically renewed while the program is running.
- When the program exits, the lease is released, making the slot available for other machines.
- If the program crashes or the network is lost, the lease expires automatically after a timeout period.

Usage is the same as node-locked licenses:

```bash
$ export QBPP_LICENSE_KEY=F-XXXXXX-XXXXXX-XXXXXX-XXXXXX
$ qbpp-license -a
```

