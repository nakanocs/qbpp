---
layout: default
title: "PyQBPP Installation"
nav_order: 3
---
# PyQBPP Installation

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
qbpp-license -a
```

If you have a license key, set it before activation:
```bash
export QBPP_LICENSE_KEY=XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX-XXXXXX
qbpp-license -a
```

If no license key is set, an **Anonymous Trial** (7 days, 1,000 variables) is activated.

For details on license types, deactivation, troubleshooting, and more, see **[License Management](../LICENSE_MANAGEMENT)**.
