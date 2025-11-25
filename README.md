# omnetpp-simu5g

### **OMNeT++ · INET · Simu5G · Time-Sensitive Networking (TSN) · 5G NR Standalone**

This repository implements a  network simulation, combining a **deterministic TSN Ethernet backbone** with a **5G NR access network** using **INET 4.5.x + Simu5G 1.4.x**.

It is designed for academic and research use, including:

* Real-time industrial networking
* Deterministic communication experiments
* 5G latency & reliability analysis
* Time-Sensitive Networking (TSN) scheduler configurations
* Load sweeping and congestion testing

---

# 📘 Project Purpose

This project simulates a **hybrid factory network** with mixed critical and non-critical traffic:

* **TSN Ethernet backbone**

  * 802.1Qbv (Time-Aware Shaper – TAS)
  * 802.1Qci (Per-Stream Filtering & Policing – PSFP)
  * 802.1AS (perfect sync assumed if unavailable)

* **5G NR Standalone (Simu5G)**

  * gNodeB + 40 mobile UEs
  * 5G Core / UPF integrated into TSN backbone
  * Realistic NR PHY/MAC stack

* **Traffic Classes**

  * Critical real-time control (1 kHz)
  * Periodic sensors (100 Hz)
  * Best-effort monitoring (Poisson 100 pkt/s)

* **Experiments**

  * TSN ON vs. TSN OFF
  * Load sweep (10–90% best-effort load)
  * Jitter, E2E delay, queue occupancy
  * Deadline miss ratio

---

# 📂 Project Structure

```
test5g/
│
├── topo/
│   └── FactoryTopo5g.ned
│
├── configs/
│   ├── base.ini
│   ├── tsn_on.ini
│   ├── tsn_off.ini
│   └── load_sweep.ini
│
├── tsn/
│   ├── gcl/
│   │   └── gcl_factory_cycle.json
│   └── psfp/
│       └── psfp_rules.json
│
├── src/
│   └── util/
│       ├── MetricsRecorder.cc
│       └── MetricsRecorder.ned
│
├── simulations/
│   └── run_all.sh
│
├── omnetpp.ini
└── test5g        # compiled simulation binary
```

---

# 🖥 System Requirements

* **OMNeT++ 6.x**
* **INET Framework 4.5.x**
* **Simu5G 1.4.x**
* CMake (if Simu5G built manually)
* Clang/Xcode (macOS) or GCC (Linux)

---

# 🍏 macOS Sequoia Setup (IMPORTANT)

macOS Sequoia introduces stricter binary security.

### 1. Install dependencies

```bash
brew install cmake gcc python3 pyenv openssl
```

### 2. Install OMNeT++

```bash
tar xzf omnetpp-6.3.0-src.tgz
cd omnetpp-6.3.0
source setenv
./configure
make -j$(sysctl -n hw.logicalcpu)
```

### 3. Allow unsigned simulation binaries

```bash
chmod +x test5g
xattr -dr com.apple.quarantine test5g
xattr -dr com.apple.quarantine test5g/test5g
```

If blocked again:
System Settings → Privacy & Security → “Allow Anyway”.

---

# 🐧 Ubuntu Setup

### Install dependencies

```bash
sudo apt update
sudo apt install build-essential clang cmake python3 python3-pip qtbase5-dev libqt5opengl5-dev
```

### Build OMNeT++

```bash
cd omnetpp-6.0.3
source setenv
./configure
make -j$(nproc)
```

### Build INET

```bash
cd inet4.5
make -j$(nproc)
```

### Build Simu5G

```bash
cd simu5g-1.4.1
make -j$(nproc)
```

---

# 🐍 Python Virtual Environment (`pyenv`)

Recommended for plotting and post-processing.

### Install pyenv

```bash
curl https://pyenv.run | bash
```

Add to shell config:

```bash
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

### Create environment

```bash
pyenv install 3.10.12
pyenv virtualenv 3.10.12 tsn5g-env
pyenv activate tsn5g-env
```

### Install Python packages

```bash
pip install numpy pandas matplotlib seaborn jupyter
```

---

# 🔧 Compilation Instructions

### 1. Load OMNeT++ environment

```bash
source /path/to/omnetpp-6.0.1/setenv
```

### 2. Compile the simulation

```bash
make -j$(nproc)
```

or on macOS:

```bash
make MODE=debug
```

---

# ▶️ Running the Simulation

### 1. TSN ON experiment

```bash
./test5g/test5g -f configs/tsn_on.ini -c tsn_on
```

### 2. TSN OFF experiment

```bash
./test5g/test5g -f configs/tsn_off.ini -c tsn_off
```

### 3. Load sweep (10–90% BE load)

```bash
./test5g/test5g -f configs/load_sweep.ini -c load_sweep
```

### 4. All experiments

```bash
./simulations/run_all.sh
```

Results appear under:

```
results/
 ├── tsn_on/
 ├── tsn_off/
 └── load_sweep/
```

---

# 📊 Results & Metrics

* End-to-end latency
* Maximum latency
* Delay variation (jitter)
* Packet loss / PDR
* Queue occupancy per TSN queue
* Deadline miss ratio (critical traffic > 500 μs)

Metrics are configured in `base.ini`.

---

# 🧩 Troubleshooting

### macOS blocks binary

```bash
xattr -dr com.apple.quarantine test5g
```

### Missing shared libraries

```bash
export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/path/to/inet/src:/path/to/simu5g/src
```

### "Unassigned parameter"

Most NR init issues are solved by:

* Adding `channelControl`
* Adding `carrierAggregation`
* Adding `binder`
* Setting CC frequencies in `base.ini`

### PPP channel errors

Ensure the PPP link has a channel:

```ned
gnbA.ppp <--> PppLink10G <--> upf.pppg[0];
```

---

# 📝 License & Academic Use

This project is intended for:

* academic coursework
* research in TSN + 5G
* reproducible simulation experiments

Forks, pull requests, and citations are welcome.

