# PA3-Pantheon
# PA3: Pantheon Congestion Control Evaluation

This repository contains the results and analysis of **Programming Assignment 3 (PA3)** for the Computer Networks course. The experiments were conducted using **Pantheon** and **Mahimahi** to evaluate the performance of different congestion control schemes.

---
## Experiments Conducted

I tested the following congestion control algorithms:

- **BBR**
- **Cubic**
- **Vegas**

Under two network scenarios:

1. **Low latency, constrained bandwidth**  
   (50 Mbps, 10ms)
2. **High latency, high bandwidth**  
   (1 Mbps, 200ms)

---

## Key Metrics Analyzed

- **Throughput**
- **Delay (95th percentile)**
- **Packet Loss Rate**
- **Utilization**

Each of these metrics is logged, summarized in CSVs, and visualized via plots.

---
# Pantheon Installation and Setup Guide

This guide outlines the steps followed to install and configure the Pantheon congestion control evaluation framework on a Ubuntu VirtualBox system.

---

## 1. Clone Pantheon Repository

```bash
git clone https://github.com/StanfordSNR/pantheon.git
cd pantheon
git submodule update --init --recursive
sudo ./tools/install_deps.sh
```

---

## 2. Install Python 2 and Pip (Deprecated)

```bash
sudo apt install python2 -y
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
sudo python2 get-pip.py
sudo pip2 install pyyaml
```

---

## 3. Install Dependencies

```bash
sudo apt install -y build-essential cmake python2 python-is-python2 pkg-config \
libboost-all-dev libprotobuf-dev protobuf-compiler libgoogle-glog-dev \
libgflags-dev libpcap-dev libssl-dev iproute2 net-tools
```

---

## 4. Fix Mahimahi PPA Issue and Install Mahimahi

Since the original PPA for Mahimahi is broken, install it manually:

```bash
sudo apt install mahimahi
```

---

## 5. Install Additional Tools

```bash
sudo apt install curl iperf -y
```

---

## 6. Build Pantheon Tunnel

```bash
cd third_party/pantheon-tunnel
make clean
make -j
```

If build fails in `src/packet`, manually go to:
```bash
cd src/packet
make clean
make -j
```

---

## 7. Kernel Module Setup for Congestion Control

```bash
sudo modprobe tcp_bbr
sudo modprobe tcp_vegas
sudo sysctl -w net.ipv4.tcp_allowed_congestion_control="reno cubic bbr vegas"
sudo sysctl -w net.core.default_qdisc=fq
sudo sysctl -w net.ipv4.ip_forward=1
```

---

## 8. Setup and Run Tests

```bash
python2 src/experiments/setup.py --setup --schemes "cubic bbr vegas"
python2 src/experiments/test.py local --schemes "cubic bbr"
```

If you encounter:
```
mm-link: Please run "sudo sysctl -w net.ipv4.ip_forward=1"
```
Run the suggested command and re-run the test.

---

## Notes

- `YAMLLoadWarning` from `utils.py` can be ignored.
- Some submodules like `proto-quic` may fail occasionally due to network; just rerun `git submodule update --init --recursive`.
- Use `Ctrl+C` to stop test runs when needed.
- Mahimahi traces and log files are saved in `src/experiments/data`.

---

## 9. Custom Network Profile Experiments

### Create Custom Trace Files

```bash
seq 1 60 | awk '{print 50}' > src/experiments/50mbps.trace
seq 1 60 | awk '{print 1}' > src/experiments/1mbps.trace
```

### Run Low Latency Experiment (50 Mbps, 5ms Delay)

```bash
python2 src/experiments/test.py local \
  --schemes "vegas cubic bbr" \
  --run-times 1 \
  --runtime 60 \
  --data-dir result/part_low_latency \
  --uplink-trace src/experiments/50mbps.trace \
  --downlink-trace src/experiments/50mbps.trace \
  --prepend-mm-cmds "mm-delay 5"

python2 src/analysis/analyze.py --data-dir result/part_low_latency
```

### Run High Latency Experiment (1 Mbps, 100ms Delay)

```bash
python2 src/experiments/test.py local \
  --schemes "vegas cubic bbr" \
  --run-times 1 \
  --runtime 60 \
  --data-dir result/part_high_latency \
  --uplink-trace src/experiments/1mbps.trace \
  --downlink-trace src/experiments/1mbps.trace \
  --prepend-mm-cmds "mm-delay 100"

python2 src/analysis/analyze.py --data-dir result/part_high_latency
```


*Last updated on: 2025-04-17*


