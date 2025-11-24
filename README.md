Energy-Aware System Monitor
A comprehensive system monitoring tool that tracks CPU usage, network activity, system calls, and context switches to estimate energy consumption. This project includes two implementations: an eBPF-based monitor for deep kernel-level insights and a traditional baseline monitor using standard system utilities.

📋 What Does This Do?
This tool helps you understand how your system is using resources by:

Monitoring CPU usage in real-time
Tracking network packets
Counting system calls (how often programs ask the kernel to do things)
Measuring context switches (how often the CPU switches between tasks)
Estimating energy consumption based on these metrics
Automatically adjusting network bandwidth when energy usage is high


🔧 Components
1. eBPF Monitor (ebpf_monitor.py)
The advanced version that uses eBPF (Extended Berkeley Packet Filter) to monitor system activities at the kernel level.
Features:

Real-time kernel-level monitoring
Automatic traffic control based on energy consumption
Color-coded status indicators (🟢 Normal, 🟡 Elevated, 🟠 High, 🔴 Critical)
CSV logging for later analysis

Energy Levels:

🟢 NORMAL: Energy < 6.0J
🟡 ELEVATED: Energy 6.0-8.0J
🟠 HIGH: Energy 8.0-10.0J
🔴 CRITICAL: Energy > 10.0J
2. Traditional Monitor (baseline_monitor.py)
A simpler version that uses standard Linux tools without eBPF.
Features:

Works on any Linux system (no eBPF required)
Can read syscall averages from eBPF logs for comparison
Lightweight and easy to understand
CSV logging


📦 Requirements
For eBPF Monitor:
Linux kernel 6.X with eBPF support
Python 3.12
Root/sudo privileges
Required packages:
sudo apt-get install python3-bpfcc bpfcc-tools linux-headers-$(uname -r)
  pip3 install psutil

For Traditional Monitor:
Linux system
Python 3.12
Required packages:
pip3 install psutil

Running the eBPF Monitor
1. Make the script executable:
chmod +x ebpf_monitor.py

2. Run with sudo (required for eBPF):
 sudo ./ebpf_monitor.py
3. Stop monitoring:
Press Ctrl+C to stop
The script will automatically clean up traffic control settings
Data is saved to system_monitor_log.csv

🔧 How Energy Is Calculated (Software Based Energy Estimation)

Both monitors use the same formula:
Energy(J) = (CPU% * 0.4) + (Syscalls * 0.00005) + (CtxSwitch * 0.0002)
This keeps comparison accurate and fair.
📁 Output Files

system_monitor_log.csv (eBPF monitor)
Contains: timestamp, status, cpu, packets, syscalls, context switches, energy, average syscalls

baseline_monitor_log.csv (Traditional monitor)
Contains: timestamp, cpu, packets, delta_packets, syscalls, context switches, energy
🔄 Comparing Both Approaches
Run the eBPF monitor first, then run the traditional monitor. The traditional monitor will automatically read the average syscall count from the eBPF log, allowing you to compare:

eBPF's kernel-level accuracy vs traditional userspace monitoring
Performance differences between the two approaches
Energy consumption patterns

⏱ Latency Comparison
1. eBPF Monitoring

eBPF runs inside the kernel, directly at tracepoints.

Events (syscalls, context switches, packets) are captured immediately without extra user-space polling.

Latency is very low, usually in microseconds for recording events.

This makes eBPF suitable for real-time monitoring and high-speed networks.

2. Traditional Monitoring

Polling is done in user-space, once per interval (1 second in our code).

Kernel-to-user transitions for each metric introduce additional latency.

Syscalls and context switches can only be measured indirectly (or averaged from eBPF results).

Latency is higher, sometimes causing delayed response in rapidly changing traffic conditions.

📂 Repository Structure
├── ebpf_monitor.py               # eBPF-based monitoring script
├── traditional_monitor.py        # Baseline monitoring script
├── system_monitor_log.csv        # Output from eBPF monitor
├── baseline_monitor_log.csv      # Output from traditional monitor
└── README.md                     # Documentation

  
