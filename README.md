Below is your **complete, polished, professional, GitHub-ready README.md** — fully formatted, clear, and detailed.
You can copy–paste directly into GitHub.

---

# ⚡ **Energy-Aware System Monitor**

A comprehensive and efficient system monitoring framework that tracks **CPU usage**, **network packets**, **system calls**, **context switches**, and provides a **software-based energy consumption estimation**.

This project includes **two implementations**:

* **eBPF-based monitor** — high-accuracy kernel-level monitoring
* **Traditional baseline monitor** — user-space monitoring using standard Linux tools

---

## 📘 **Overview**

Modern systems generate millions of events per second. Traditional monitoring tools rely on heavy user-space polling, which introduces **latency**, **CPU overhead**, and **extra energy consumption**.

This project demonstrates how **eBPF** can drastically improve:

* Monitoring accuracy
* Latency
* System efficiency
* Energy consumption

Both monitors produce CSV logs that can be used for further research, visualization, or performance comparison.

---

# 📋 **What This Tool Monitors**

This system monitors:

### 🔹 **CPU usage**

Measured once every second using psutil.

### 🔹 **Network packets**

Tracks both received & transmitted packets.

### 🔹 **System calls**

Captures how frequently user applications request services from the kernel.

### 🔹 **Context switches**

Measures how often the CPU switches between processes or threads.

### 🔹 **Software-based energy estimation**

A formula calculates approximate energy consumption:

```
Energy(J) = (CPU% * 0.4) + (Syscalls * 0.00005) + (CtxSwitch * 0.0002)
```

---

# 🔧 **Project Components**

## 1️⃣ **eBPF Monitor (`ebpf_monitor.py`)**

A powerful monitoring script using **Extended Berkeley Packet Filter (eBPF)**.

### ✔ Features

* Real-time kernel-level monitoring
* Tracks system calls, context switches, and packets via kernel tracepoints
* Automatic traffic control (HTB + SFQ) based on energy usage
* Status indicators:

  * 🟢 **NORMAL** — Energy < 6J
  * 🟡 **ELEVATED** — 6–8J
  * 🟠 **HIGH** — 8–10J
  * 🔴 **CRITICAL** — >10J
* CSV logging (`system_monitor_log.csv`)
* Computes **average syscalls** before exit
* Extremely low latency (microseconds)

---

## 2️⃣ **Traditional Baseline Monitor (`traditional_monitor.py`)**

A simpler fallback version that does **not require eBPF**.

### ✔ Features

* Portable — works on almost any Linux system
* Reads average syscalls from eBPF logs for fair comparison
* Uses `/proc/stat` + psutil
* CSV logging (`baseline_monitor_log.csv`)
* Easy to understand for beginners
* Higher latency compared to eBPF

---

# 🧪 **Latency Comparison**

### ⚡ **eBPF Monitoring**

* Runs **inside kernel**, not user-space
* Captures events instantly
* **Microsecond-level latency**
* Ideal for high-speed networks & real-time systems

### 🐌 **Traditional Monitoring**

* User-space polling (1 second interval)
* Requires kernel → user transitions
* Cannot directly measure syscalls
* **Millisecond to second-level latency**
* Slower response to rapid system changes

---

## For Traditional Monitor

* Any Linux system
* Python **3.x**
* Install:

```bash
pip3 install psutil
```

---

# ▶️ **How to Run the eBPF Monitor**

### 1. Make executable:

```bash
chmod +x ebpf_monitor.py
```

### 2. Run with sudo:

```bash
sudo ./ebpf_monitor.py
```

### 3. Stop monitoring:

Press **Ctrl + C**
The script will:

* Stop monitoring
* Compute average syscalls
* Update CSV
* Clean up traffic control settings

CSV saved as:

```
system_monitor_log.csv
```

---

# ▶️ **How to Run the Traditional Monitor**

Run:

```bash
python3 traditional_monitor.py
```

It will:

* Load average syscalls from eBPF log (if available)
* Begin logging data every second

CSV saved as:

```
baseline_monitor_log.csv
```

---

# 📁 **Output Files**

### 📄 `system_monitor_log.csv` (eBPF Monitor)

Contains:

* timestamp
* status
* cpu
* packets
* syscalls
* context switches
* energy
* average syscalls

### 📄 `baseline_monitor_log.csv` (Traditional Monitor)

Contains:

* timestamp
* cpu
* packets
* delta_packets
* syscalls
* ctxswitches
* energy

---

# 🔄 **Comparing Both Approaches**

Run in this order:

1. **eBPF monitor first**
2. **Traditional monitor second**

Traditional monitor reads the syscall average from the eBPF log.

### What you can compare:

* Kernel vs user-space latency
* Measurement accuracy
* CPU overhead
* Energy consumption
* Packet & context-switch detection quality

---

# 📂 **Repository Structure**

```
📂 Repository Structure
├── ebpf_monitor.py               # eBPF-based monitoring script
├── traditional_monitor.py        # Baseline monitoring script
├── system_monitor_log.csv        # Output from eBPF monitor
├── baseline_monitor_log.csv      # Output from traditional monitor
└── README.md                     # Documentation
```

---

📶 Dynamic Bandwidth Management (HTB + SFQ)

The eBPF monitor includes an automatic bandwidth control system using Linux Traffic Control (tc) to prevent high energy consumption during heavy loads.

When the system energy level goes high, the script does NOT reduce bandwidth, but instead classifies the current energy state and can trigger bandwidth policies.
Right now the project sets up a clean HTB (Hierarchical Token Bucket) environment for stable packet monitoring.

✔ What the Bandwidth System Does

Removes any existing qdisc root (cleans the interface)

Creates a new HTB root qdisc

Creates a default traffic class at full speed (1000mbit)

Adds SFQ (Stochastic Fairness Queueing) to keep packet flow fair and stable

Ensures consistent and accurate packet sampling for eBPF

✔ Why This Matters

Normal network interfaces have unpredictable buffering and queueing.
To get accurate packet counts for energy analysis, we apply:

HTB → For shaping total bandwidth (fixed predictable rate)
SFQ → For fair distribution of packets to avoid burst spikes

This helps ensure:

Stable Pkt/s (packet per second) measurement

Smoother energy calculation

More accurate traffic monitoring results

# 🏁 **Conclusion**

This project demonstrates the **clear advantage** of eBPF over traditional monitoring systems:

### With eBPF:

* Lower overhead
* Higher accuracy
* Better latency
* More complete system visibility
* More realistic energy estimation

Traditional monitoring is simpler but cannot match kernel-level precision.

---

