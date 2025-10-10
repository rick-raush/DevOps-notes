Architecture

| Category | Architecture Name | Common Label(s) | Hardware Vendor / CPU Type | 64-bit or 32-bit | Package Type Used | Typical Systems |
| --- | --- | --- | --- | --- | --- | --- |
| **Intel / AMD (x86 family)** | **x86_64** | amd64, x64 | Intel Core / Xeon, AMD Ryzen / EPYC | 64-bit | `.deb`, `.rpm` | Ubuntu, Debian, RHEL, CentOS, Fedora, SUSE |
|     | **x86** | i386, i686 | Legacy Intel / AMD 32-bit CPUs | 32-bit | `.deb`, `.rpm` | Very old Linux or embedded systems |
| **ARM (AArch)** | **arm64** | aarch64 | ARMv8+ (Graviton, Apple M-series, Ampere, etc.) | 64-bit | `.deb`, `.rpm` | AWS Graviton, Raspberry Pi 4+, Apple Silicon |
|     | **armhf** | armv7l | ARMv7 (older 32-bit ARM CPUs) | 32-bit | `.deb` (mostly) | Raspberry Pi 2/3 (32-bit OS) |
| **Package format** | —   | —   | —   | —   | `.deb` (Debian package) | Ubuntu, Debian |
|     | —   | —   | —   | —   | `.rpm` (Red Hat package) | RHEL, CentOS, Fedora, SUSE |

| Package Tool | Type | OS Family | Handles Dependencies? | File Type |
| --- | --- | --- | --- | --- |
| dpkg | Low-level | Debian/Ubuntu | ❌ No | `.deb` |
| apt/apt-get | High-level | Debian/Ubuntu | ✅ Yes | `.deb` |
| rpm | Low-level | RHEL/CentOS/Fedora | ❌ No | `.rpm` |
| yum/dnf | High-level | RHEL/CentOS/Fedora | ✅ Yes | `.rpm` |