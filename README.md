# Case Study: Resolving Driver-Level Protocol Throttling on Realtek RTL8821CE Interfaces under Windows 11

## 📌 Project Overview
This repository documents a real-world technical support case study involving a critical hardware-software mismatch on an **HP Laptop 15-dw0xxx** equipped with a **Realtek RTL8821CE 802.11ac PCIe Adapter**. 

The system experienced severe throughput volatility (erratic speed spikes dropping down to 0 Kbps) and was completely locked out of high-speed wireless protocols, remaining restricted to legacy legacy modes despite modern hardware capabilities.

---

## 🔍 Root Cause Analysis (RCA)
1. **Driver Regression:** Windows 11 persistently pulled a generic, outdated driver from its internal backup cache instead of utilizing the proper OEM/Realtek software package.
2. **Protocol Throttling:** The corrupted driver structure restricted the adapter's advanced properties menu, entirely hiding the high-speed **IEEE 802.11ac** option and locking the device into legacy 2.4 GHz bands.
3. **Power Management Interference:** Aggressive Windows OS power-saving features were permitted to suspend power to the PCIe network card, triggering sudden link-state drops to 0 Kbps during active data transmission.

---

## 🛠️ Step-by-Step Resolution Workflow

### Phase 1: Overriding Operating System Power Restrictions
To isolate the physical link drops from systemic power throttling, OS-level power saving was disabled directly on the hardware profile:
* **Action:** Navigated to `Device Manager` -> `Network Adapters` -> `Realtek RTL8821CE Properties` -> `Power Management`.
* **Execution:** Unchecked *"Allow the computer to turn off this device to save power"* to maintain persistent voltage to the PCIe interface.

### Phase 2: Unlocking the 5 GHz Radio Band
Because the card was trapped on the 2.4 GHz spectrum, the advanced configuration parameters were manually altered to prioritize high-frequency bands:
* **Action:** Accessed the `Advanced` properties tab.
* **Execution:** Changed `Preferred Band` value to **Prefer 5.2G band** (5GHz) and verified that **VHT Mode** was enabled to allow wider channel bonding.

### Phase 3: Commencing Command Line Stack Resets
To flush out persistent configuration anomalies within the Windows network architecture, a forced reset of the network socket and IP routing tables was executed via the Administrative Command Line Interface (CLI):
```cmd
:: Reset Winsock catalog to clean state
netsh winsock reset

:: Reset TCP/IP stack parameters and overwrite registry entries
netsh int ip reset
```

### Phase 4: Manual Factory Driver Package Injection
Because the Windows OS driver database remained locked, the generic driver was bypassed completely by manually downloading and deploying the verified OEM factory executable package directly from the HP server topology:
* **Target Package:** `HP SoftPaq sp162860.exe` (Realtek Wireless Driver Package).
* **Deployment:** Executed clean driver installation, stripping the existing cached driver binaries, followed by a system reboot to force hardware re-enumeration.

---

## 🏆 Post-Mortem Verification & Results
Following the deployment of the factory OEM driver package and the CLI network stack purge, the interface successfully re-negotiated its connection parameters with the host router:

| Metric | Pre-Troubleshooting State | Post-Troubleshooting State |
| :--- | :--- | :--- |
| **Active Driver Status** | Generic / Corrupted Cache | Factory Approved OEM (Realtek) |
| **Wireless Protocol** | Legacy 802.11n (2.4 GHz) | **Modern 802.11ac (5 GHz)** |
| **Throughput Stability** | Highly Volatile (Drops to 0 Kbps) | **Stable, Continuous Data Flow** |
| **Advanced Menu Options** | Restrictive / Locked | **Fully Restored (IEEE 802.11a/n/ac Enabled)** |

**Conclusion:** The device now stably maintains its high-bandwidth connection without drops, proving that systemic network bottlenecks can be successfully bypassed via manual driver injection and OS subsystem optimization.
