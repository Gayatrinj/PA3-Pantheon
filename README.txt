##Pantheon Setup & Experiment Execution Guide (Ubuntu 20.04)
#Prerequisites
Install essential dependencies:
sudo apt update && sudo apt upgrade -y
sudo apt install -y build-essential cmake python2 python-is-python2 curl \
git pkg-config libboost-all-dev libprotobuf-dev protobuf-compiler \
libgoogle-glog-dev libgflags-dev libpcap-dev libssl-dev iproute2 \
net-tools gnuplot mahimahi

Install pip2 and Python dependencies:
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
sudo python2 get-pip.py
sudo pip2 install matplotlib numpy tabulate pyyaml
Clone & Setup Pantheon
git clone https://github.com/StanfordSNR/pantheon.git
cd pantheon
git submodule update --init --recursive
System Config for Congestion Control
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.core.default_qdisc=fq
sudo sysctl -w net.ipv4.tcp_allowed_congestion_control="reno cubic bbr vegas"
# Setup Congestion Control Schemes
python2 src/experiments/setup.py --setup --schemes "cubic bbr vegas"
python2 src/experiments/test.py  local --schemes "cubic bbr vegas"  # for Part A
##Part B: Experiment Execution Steps
#Step 1: Create Traces
seq 1 60 | awk '{print 50}' > src/experiments/50mbps.trace  # Low latency bandwidth
seq 1 60 | awk '{print 1}' > src/experiments/1mbps.trace    # High latency bandwidth
#Step 2: Run Low-Latency Experiment
python2 src/experiments/test.py local \
  --schemes "vegas cubic bbr" \
  --run-times 1 \
  --runtime 60 \
  --data-dir result/part_low_latency \
  --uplink-trace src/experiments/50mbps.trace \
  --downlink-trace src/experiments/50mbps.trace \
  --prepend-mm-cmds "mm-delay 5"
#Step 3: Run High-Latency Experiment
python2 src/experiments/test.py local \
  --schemes "vegas cubic bbr" \
  --run-times 1 \
  --runtime 60 \
  --data-dir result/part_high_latency \
  --uplink-trace src/experiments/1mbps.trace \
  --downlink-trace src/experiments/1mbps.trace \
  --prepend-mm-cmds "mm-delay 100"
#Step 4: Analyze Results
# Analyze low latency results
python2 src/analysis/analyze.py --data-dir result/part_low_latency

# Analyze high latency results
python2 src/analysis/analyze.py --data-dir result/part_high_latency
##Common Errors & Fixes
1. Permission Errors (autom4te.cache):
Fix: Run `sudo chown -R $USER:$USER pantheon/` and delete `autom4te.cache`.


2. Build Failures due to -Werror=stringop-truncation:
Fix: Remove `-Werror` from Makefiles or modify strncpy usage in C++ source files.

3. mm-tunnelserver: needs to be installed setuid root:
Fix:
sudo chown root:root pantheon/third_party/pantheon-tunnel/src/frontend/mm-tunnelserver
sudo chmod u+s pantheon/third_party/pantheon-tunnel/src/frontend/mm-tunnelserver

4. Authentication failed during Git push:
Fix: Set up GitHub with a personal access token (PAT) or use SSH instead of HTTPS.

5. plot.py error - missing metadata file:
Fix: Ensure `pantheon_metadata.json` exists in the result directory before running `plot.py`.
