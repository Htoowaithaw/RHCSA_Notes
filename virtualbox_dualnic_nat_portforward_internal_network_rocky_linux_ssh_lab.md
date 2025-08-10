🖥 VirtualBox VM-to-VM SSH + Host Access Setup (Rocky Linux)
📜 Scenario

    Host: Using public Wi-Fi (cannot bridge VMs directly to network)

    Initially: Only one Rocky Linux VM in VirtualBox

    Goal:

        Have two Rocky Linux VMs in VirtualBox

        SSH from host → VM1 and host → VM2 (via NAT port forwarding)

        SSH VM1 ↔ VM2 directly (internal network)

        Keep internet access for both VMs

🛠 Step 1 — Clone the First VM

    In VirtualBox → select VM → Right-click → Clone

    Choose Full Clone (safe, independent)

    Give the second VM a new name (e.g., Rocky-2)

    Clone while powered off (safer to avoid inconsistencies)

🌐 Step 2 — Network Setup in VirtualBox

Do for both VMs:
Adapter 1 — NAT (Internet + Host Port Forwarding)

    Settings → Network → Adapter 1

    Enable → Attached to: NAT

    Advanced → Port Forwarding:

        VM1: Host Port 2222 → Guest Port 22

        VM2: Host Port 2223 → Guest Port 22

Adapter 2 — Internal Network (VM ↔ VM)

    Settings → Network → Adapter 2

    Enable → Attached to: Internal Network

    Name: intnet (must be identical for both VMs)

⚙ Step 3 — Assign Internal Network IPs in Rocky Linux

Do inside each VM for enp0s8 (Internal Network interface).

You can use nmtui (menu) or nmcli (commands).
We’ll use nmtui for easy setup.
VM1 (192.168.56.101)

nmtui

    Edit a connection → Add → Ethernet

    Profile name: intnet1

    Device: enp0s8

    IPv4 Configuration: Manual → Address: 192.168.56.101/24

    Save → Activate

VM2 (192.168.56.102)

nmtui

    Edit a connection → Add → Ethernet

    Profile name: intnet2

    Device: enp0s8

    IPv4 Configuration: Manual → Address: 192.168.56.102/24

    Save → Activate

🔍 Step 4 — Verify IP Assignments

ip a show enp0s8

You should see:

    VM1 → 192.168.56.101/24

    VM2 → 192.168.56.102/24

🧪 Step 5 — Test SSH Connections
Host → VM1

ssh -p 2222 user@127.0.0.1

Host → VM2

ssh -p 2223 user@127.0.0.1

VM1 → VM2

ssh user@192.168.56.102

VM2 → VM1

ssh user@192.168.56.101

📌 Notes & Tips

    NAT Port Forwarding is host ↔ VM only, not VM ↔ VM.

    Internal Network gives direct VM ↔ VM communication.

    Both adapters can coexist — NAT for internet, Internal for local lab.

    If you see enp0s8 but no IP → it means no NetworkManager profile exists → create it via nmtui or nmcli.

    This setup works even on public Wi-Fi since the VMs are not exposed to the outside network.

✅ End Result

    Host can SSH into both VMs using different forwarded ports.

    VMs can SSH into each other using static IPs over Internal Network.

    Both VMs have internet via NAT adapter.

