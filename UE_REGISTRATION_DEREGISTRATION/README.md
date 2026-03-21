## 1. UE Registration – Open5GS Configuration (AMF)

Before registering a UE, update the Open5GS configuration to bind services to the correct VM IP.

---

### 1.1 Configure NGAP Bind Address (AMF)

#### Step 1: Open AMF Configuration File

```bash
sudo nano /etc/open5gs/amf.yaml
```
#### Step 2:Update NGAP Address

Modify the ngap section as shown below:
To check ip use command:
```bash
ip a
```

```yaml
amf:
  sbi:
    server:
      - address: 127.0.0.5
        port: 7777
    client:
#      nrf:
#        - uri: http://127.0.0.10:7777
      scp:
        - uri: http://127.0.0.200:7777
  ngap:
    server:
      - address: 192.168.56.102  # ⚠️ Set this to your Open5GS VM IP
  metrics:
    server:
      - address: 127.0.0.5
        port: 9090
```

#### 1.2 Restart AMF Service
Since only configuration files are modified, no daemon reload is required.

```bash
sudo systemctl restart open5gs-amfd

# To ensure AMF is running correctly, check logs:
tail -f /var/log/open5gs/amf.log 
```

## 2. UE Registration – Open5GS Configuration (UPF / GTP-U)

Configure the GTP-U bind address in the UPF to enable user-plane communication.

---

### 2.1 Open UPF Configuration File

```bash
sudo nano /etc/open5gs/upf.yaml
```

### 2.2 Update GTP-U Bind Address
Set the gtpu server address to the Open5GS VM IP.

```yaml
logger:
  file:
    path: /var/log/open5gs/upf.log
#  level: info   # fatal|error|warn|info(default)|debug|trace

global:
  max:
    ue: 1024  # The number of UE can be increased depending on memory size.
#    peer: 64

upf:
  pfcp:
    server:
      - address: 127.0.0.7
    client:
#      smf:     #  UPF PFCP Client try to associate SMF PFCP Server
#        - address: 127.0.0.4
  gtpu:
    server:
      - address: 192.168.56.102  # ⚠️ Set this to your Open5GS VM IP
  session:
    - subnet: 10.45.0.0/16
      gateway: 10.45.0.1
    - subnet: 2001:db8:cafe::/48
      gateway: 2001:db8:cafe::1
```

### 2.3 Restart UPF Service

```bash
sudo systemctl restart open5gs-upfd

# To ensure UPF is running correctly, check logs:
tail -f /var/log/open5gs/upf.log
```

## 3. UE Registration – UERANSIM Configuration (UE)

---

### 3.1 Prepare UE Configuration File

On the **UERANSIM VM**, navigate to the configuration directory and create a copy of the default UE config file.

```bash
cd ~
cd UERANSIM
cd config
cp open5gs-ue.yaml open5gs-ue1.yaml

# Edit the UE configuration file:
sudo nano ~/UERANSIM/config/open5gs-ue1.yaml
```

Update the gnbSearchList with the UERANSIM VM (gNB) IP:

```yaml
# List of gNB IP addresses for Radio Link Simulation
gnbSearchList:
  - 192.168.56.101
```

### 3.2 Register UE in Open5GS WebUI

On the **Open5GS VM**:

1. Open a browser and navigate to: `http://localhost:9999/`

2. Copy the IMSI number from:  
```bash
# On the UERANSIM virtual machine
cat ~/UERANSIM/config/open5gs-ue1.yaml
``` 

3. Click **+** to register a new subscriber

4. Enter the following:
    - **IMSI:** `<only numerical value copied earlier>`

5. Keep all other fields as default

6. Click **Save**

