---
layout: default
title: "Installation"
nav_order: 3
---
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

