# 📡 Open5GS + UERANSIM Setup Guide (VirtualBox)

## 1. Virtual Machine Setup

### Prerequisites
- Virtualization software: **VirtualBox (v7.2.6)**

### VM Configuration

Create two separate Ubuntu-based virtual machines:

#### 🔹 VM 1: Open5GS Core Network
- **Name:** `Open5GS`
- **RAM:** 3 GB
- **Storage:** 30 GB
- **CPU:** 3 cores

#### 🔹 VM 2: UERANSIM (RAN + UE)
- **Name:** `UERANSIM`
- **RAM:** 3 GB
- **Storage:** 25 GB
- **CPU:** 2 cores

---

## 2. Open5GS Installation (Core Network VM)

📖 Reference Guide:  
https://open5gs.org/open5gs/docs/guide/01-quickstart/

---

### 2.1 Install MongoDB

#### Import MongoDB GPG Key
```bash
sudo apt update
sudo apt install gnupg

curl -fsSL https://pgp.mongodb.com/server-8.0.asc | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list
```
#### Install the MongoDB packages
```bash
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod
```

### 2.2 Ubuntu (Install Open5GS)
```bash
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs
```

### 2.3 Install WebUI of Open5GS

#### Install Nodejs in Ubuntu
```bash
# Download and import the Nodesource GPG key
sudo apt update
sudo apt install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg

# Create deb repository
NODE_MAJOR=20
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list

# Run Update and Install
sudo apt update
sudo apt install nodejs -y

# Install WebUI of Open5GS
curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
```

---

## 3. UERANSIM Installation (RAN + UE VM)

📖 Official Guide:  
https://github.com/aligungr/UERANSIM/wiki/Installation

Follow the official documentation for installation. The steps are well-structured and should be executed as-is.

---

## 4. Network Configuration (Host-Only Adapter)

To enable communication between the two VMs, configure a private network using a **Host-Only Adapter**.

---

### 4.1 Shutdown Virtual Machines

Ensure both VMs are powered off before making network changes.

---

### 4.2 Create Host-Only Network

1. Open **VirtualBox**
2. Navigate to:  
   **File → Tools → Network**
3. Create a new **Host-Only Adapter**
4. Ensure:
   - A subnet is assigned (e.g., `192.168.x.x`)
   - **DHCP is enabled**

---

### 4.3 Attach Host-Only Adapter to VMs

For **each VM**:

1. Go to **Settings → Network**
2. Enable **Adapter 2**
3. Set:
   - **Attached to:** Host-Only Adapter
4. Select the newly created host-only network (auto-populated)

---

### 4.4 Boot and Verify Network Interface

Start both VMs and verify that the new interface (e.g., `enp0s8`) has been assigned an IP address.

```bash
ip a