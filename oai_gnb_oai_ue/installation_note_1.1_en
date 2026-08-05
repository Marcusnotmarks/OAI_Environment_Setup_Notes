# OAI gNB (CU/DU Split) and OAI NR-UE RF Simulator Installation Guide

This document records the complete process of setting up the OAI environment, including all required commands and configurations.

## Prerequisites

An Ubuntu environment is required before building the OAI environment.

It is recommended to prepare the following:

- Ubuntu 22.04 LTS
- VMware Workstation
- Git
- Internet access
- VPN connection (if access to the laboratory network is required)

---

## Working Directory Setup

To avoid using personal usernames or fixed paths throughout this document, the following environment variables are recommended:

```bash
export WORKDIR="$HOME/oai-test"
export OAI_DIR="$WORKDIR/openairinterface5g"
export GUIDE_DIR="$WORKDIR/5g-nr-rfsim-guides"
export BUILD_DIR="$OAI_DIR/cmake_targets/ran_build/build"
```

Create the working directory:

```bash
mkdir -p "$WORKDIR"
cd "$WORKDIR"
```

Verify the current directory:

```bash
pwd
```

The following commands in this document will use `$WORKDIR`, `$OAI_DIR`, `$GUIDE_DIR`, and `$BUILD_DIR` instead of personal paths such as `/home/<username>/...`.

> **Note:** If a new terminal window is opened, the environment variables must be exported again, or added to your shell configuration file.

---

# OAI Environment Setup

### 1. Verify the Ubuntu Version

Run the following command in the terminal:

```bash
lsb_release -a
```

Verify that the Ubuntu version is **22.04.x LTS**.

Example:

```text
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.5 LTS
Release:        22.04
Codename:       jammy
```

---

### 2. Update the Package Index

Run the following command to update the package index:

```bash
sudo apt update
```

This command updates the package repository information before installing the required software.

![Verify Ubuntu Version](../Images/oai_seup_2.png)

---

### 3. Install Git

Install Git:

```bash
sudo apt install -y git
```

Verify the installed Git version:

```bash
git --version
```

The result in this setup is:

```text
git version 2.34.1
```

The Git version does not have to be exactly the same, as long as it supports cloning, checking out branches, and pulling updates.

---

### 4. Clone and Update the OAI Repository

**1.** Clone the OpenAirInterface repository.

```bash
git clone https://github.com/OPENAIRINTERFACE/openairinterface5g.git
```

**2.** Enter the repository and verify the current directory.

```bash
cd "$OAI_DIR"

pwd
```

**3.** Switch to the `develop` branch.

```bash
git checkout develop
```

**4.** Synchronize the latest source code.

```bash
git pull origin develop
```

![Clone OAI Repository](../Images/oai_setup_3.png)

---

### 4.1 Record the Tested Commit (Recommended)

Since the `develop` branch is continuously updated, recording only the branch name is not sufficient for reproducing the same environment in the future.

Retrieve the current commit SHA:

```bash
git rev-parse HEAD
```

Record the output in your README or documentation:

```text
Tested OAI Commit:
<TESTED_COMMIT_SHA>
```

When rebuilding the environment later, switch to the recorded commit:

```bash
git checkout <TESTED_COMMIT_SHA>
```

This ensures that the same OAI source version is used.

---

### 5. Prepare the OAI Build Environment

**1.** Enter the build directory.

```bash
cd "$OAI_DIR/cmake_targets"
```

Verify the current directory:

```bash
pwd
```

**2.** Load the OAI environment.

```bash
source oaienv
```

![Load OAI Environment](../Images/oai_setup_4.png)

**3.** Install all required dependencies.

```bash
./build_oai -I --install-optional-packages
```

This command installs the required dependencies for building OAI, including:

- GCC
- CMake
- ASN.1 Compiler
- UHD
- Wireshark
- SCTP Library
- Other required libraries

![Install Dependencies](../Images/oai_setup_5.png)

---

### 6. Build the gNB and NR-UE

**1.** Build the OAI gNB and NR-UE to generate the executable files required for RF simulation.

```bash
./build_oai --gNB --nrUE
```

If the build is successful, the output should end with:

```text
BUILD SHOULD BE SUCCESSFUL
```

Successful build result:

![Successful gNB Build](../Images/oai_setup_6.png)

**2.** Verify that the executables have been generated.

```bash
cd "$BUILD_DIR"

ls -l nr-softmodem nr-uesoftmodem
```

The following executables should exist:

```text
nr-softmodem

nr-uesoftmodem
```

![Build Result](../Images/oai_setup_7.png)

**3.** Verify that the default configuration files exist.

```bash
cd "$OAI_DIR"

find . -name "cu_gnb.conf"
find . -name "du_gnb.conf"
find . -name "ue.conf"
```

Example output:

```text
./targets/PROJECTS/GENERIC-NR-5GC/CONF/cu_gnb.conf

./targets/PROJECTS/GENERIC-NR-5GC/CONF/du_gnb.conf

./targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf
```

> **Note:** These are the default configuration files provided by the OAI source code. The actual configuration files used in this guide are located in the `5g-nr-rfsim-guides` repository.

**4.** Verify the build directory.

```bash
cd "$BUILD_DIR"

pwd

ls
```

The generated executables should still be present.

![Build Directory](../Images/oai_setup_8.png)
---

## 7. Configure CU / DU / UE

### 7.1 Authenticate GitHub CLI

If the configuration repository is private, GitHub authentication is required before cloning.

Install GitHub CLI:

```bash
sudo apt update
sudo apt install -y gh
```

Run:

```bash
gh auth login
```

Select the following options:

```text
GitHub.com
HTTPS
Authenticate Git with your GitHub credentials
Login with a web browser
```

A one-time Device Code will be displayed. Complete the authentication process in your web browser.

> **Note:** The Device Code is temporary authentication information. Do not include it in documentation, screenshots, or commit it to GitHub.

Verify the login status:

```bash
gh auth status
```

![GitHub CLI Login](../Images/oai_set_up_10.png)

---

### 7.2 Clone the RF Simulator Configuration Repository

```bash
cd "$WORKDIR"

git clone https://github.com/bmw-ece-ntust/5g-nr-rfsim-guides.git
```

Verify that the configuration files exist:

```bash
ls "$GUIDE_DIR/oai_gnb_oai_ue/config"
```

The following files should be present:

```text
cu_gnb.conf

du_gnb.conf

ue.conf
```

![Clone Configuration Repository](../Images/oai_setup_9.png)

---

### 7.3 Back Up the Configuration Files

Before making any modifications, create backup copies of the configuration files.

```bash
cp "$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf" \
   "$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf.bak"

cp "$GUIDE_DIR/oai_gnb_oai_ue/config/du_gnb.conf" \
   "$GUIDE_DIR/oai_gnb_oai_ue/config/du_gnb.conf.bak"

cp "$GUIDE_DIR/oai_gnb_oai_ue/config/ue.conf" \
   "$GUIDE_DIR/oai_gnb_oai_ue/config/ue.conf.bak"
```

---

### 7.4 Verify the Network Parameters

Before modifying the configuration files, confirm the required network parameters with your laboratory administrator.

```bash
export AMF_IP="<AMF_IP>"
export GNB_N2_IP="<GNB_N2_IP>"
export GNB_N3_IP="<GNB_N3_IP>"
```

Parameter descriptions:

| Parameter | Description |
|-----------|-------------|
| `AMF_IP` | IP address of the Open5GS AMF |
| `GNB_N2_IP` | Local IP address used by the CU for the NG (N2) interface |
| `GNB_N3_IP` | Local IP address used by the CU for the GTP-U (N3) interface |

Check the current IP address:

```bash
hostname -I
```

You may also verify the outgoing source IP toward the AMF:

```bash
ip route get "$AMF_IP"
```

The `src` field in the output is typically the local IP address that should be configured.

---

### 7.5 Modify the CU Configuration File

Open the configuration file:

```bash
nano "$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf"
```

Modify the following parameters:

```conf
amf_ip_address = ({ ipv4 = "<AMF_IP>"; });

GNB_IPV4_ADDRESS_FOR_NG_AMF = "<GNB_N2_IP>";
GNB_IPV4_ADDRESS_FOR_NGU    = "<GNB_N3_IP>";
```

Replace the placeholders with the actual network parameters.

If you prefer using `sed`, first verify the existing values:

```bash
grep -nE \
'amf_ip_address|GNB_IPV4_ADDRESS_FOR_NG_AMF|GNB_IPV4_ADDRESS_FOR_NGU' \
"$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf"
```

> It is not recommended to hard-code laboratory IP addresses in public documentation.

---

### 7.6 Verify the CU Configuration

Run:

```bash
grep -nE \
'amf_ip_address|GNB_IPV4_ADDRESS_FOR_NG_AMF|GNB_IPV4_ADDRESS_FOR_NGU' \
"$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf"
```

Expected output:

```text
amf_ip_address = ({ ipv4 = "<AMF_IP>"; });
GNB_IPV4_ADDRESS_FOR_NG_AMF = "<GNB_N2_IP>";
GNB_IPV4_ADDRESS_FOR_NGU    = "<GNB_N3_IP>";
```

---

### 7.7 Verify Sensitive Information in the UE Configuration

The `ue.conf` file may contain sensitive information such as:

- IMSI / SUPI
- Subscriber Key
- OP / OPc
- DNN
- NSSAI

When publishing the repository, replace these values with placeholders:

```conf
imsi = "<TEST_IMSI>";
key = "<TEST_SUBSCRIBER_KEY>";
opc = "<TEST_OPC>";
dnn = "<TEST_DNN>";
```

If the repository contains test credentials, clearly indicate that they are for testing purposes only:

```text
Demo/Test Only
Do not use in production or public networks.
```

---

## 8. Start CU, DU, and UE

### 8.1 Startup Order

Open three separate terminal windows.

```text
Terminal 1 : CU

Terminal 2 : DU

Terminal 3 : UE
```

If the environment variables have not been configured in the current terminal, run:

```bash
export WORKDIR="$HOME/oai-test"
export OAI_DIR="$WORKDIR/openairinterface5g"
export GUIDE_DIR="$WORKDIR/5g-nr-rfsim-guides"
export BUILD_DIR="$OAI_DIR/cmake_targets/ran_build/build"
```

Then navigate to the build directory:

```bash
cd "$BUILD_DIR"
```

---

### 8.2 Terminal 1 — Start the CU

Run:

```bash
sudo ./nr-softmodem \
  --rfsim \
  -O "$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf"
```

If the CU starts successfully, the log should contain messages similar to:

```text
Received NGSetupResponse from AMF

Received NGAP_REGISTER_GNB_CNF

Received F1 Setup Request from gNB_DU

sending F1 Setup Response
```

This indicates that:

- The CU has started successfully.
- The NG interface has been established with the AMF.
- The F1 interface has been established with the DU.

![Successful CU Result 1](../Images/oai_setup_13.png)

![Successful CU Result 2](../Images/oai_setup_14.png)

---

### 8.3 Terminal 2 — Start the DU

Run:

```bash
sudo ./nr-softmodem \
  --rfsim \
  -O "$GUIDE_DIR/oai_gnb_oai_ue/config/du_gnb.conf"
```

If the DU starts successfully, the log should contain messages similar to:

```text
Starting F1AP at DU

received F1 Setup Response from CU

RU 0 RF started

CBRA procedure succeeded
```

This indicates that:

- The DU has successfully connected to the CU.
- The RF Simulator has started correctly.
- The UE has completed the Contention-Based Random Access (CBRA) procedure.

![Successful DU Result](../Images/oai_setup_15.png)

---

### 8.4 Terminal 3 — Start the UE

Run:

```bash
sudo ./nr-uesoftmodem \
  -r 106 \
  --numerology 1 \
  --band 78 \
  -C 3619200000 \
  --rfsim \
  -O "$GUIDE_DIR/oai_gnb_oai_ue/config/ue.conf"
```

If the UE starts successfully, the log should contain messages similar to:

```text
Connection to 127.0.0.1:4043 established

Initial sync successful

SIB1 decoded

4-Step RA procedure succeeded

State = NR_RRC_CONNECTED

Received Registration Accept

Received PDU Session Establishment Accept

TUN Interface oaitun_ue1 successfully configured
```

This indicates that:

- The UE has synchronized successfully.
- The UE has completed the Random Access procedure.
- The RRC connection has been established.
- NAS registration has completed successfully.
- The PDU Session has been established.
- The UE tunnel interface has been created successfully.

![Successful UE Result](../Images/oai_setup_16.png)

---

### 8.5 Stop the Simulation

When the simulation is complete, stop the processes in the following order by pressing **Ctrl + C** in each terminal:

1. UE
2. DU
3. CU

Stopping the processes does **not** remove the OAI source code, configuration files, or build results.

The next time the environment is used, simply restart the CU, DU, and UE using the commands above. Rebuilding or cloning the repository is not required unless the source code or configuration files have been modified.
---

# 9. Verification Checklist

After starting the CU, DU, and UE, verify that the entire OAI RF Simulator environment is operating correctly according to the following checklist.

---

## 9.1 Build Verification

Confirm that:

- [ ] Ubuntu 22.04 LTS is installed.
- [ ] The OAI repository has been cloned successfully.
- [ ] The tested OAI commit SHA has been recorded.
- [ ] `build_oai -I --install-optional-packages` completed successfully.
- [ ] `build_oai --gNB --nrUE` completed successfully.
- [ ] `nr-softmodem` has been generated.
- [ ] `nr-uesoftmodem` has been generated.

---

## 9.2 Configuration Verification

Confirm that:

- [ ] `cu_gnb.conf` exists.
- [ ] `du_gnb.conf` exists.
- [ ] `ue.conf` exists.
- [ ] `amf_ip_address` is configured correctly.
- [ ] `GNB_IPV4_ADDRESS_FOR_NG_AMF` is configured correctly.
- [ ] `GNB_IPV4_ADDRESS_FOR_NGU` is configured correctly.
- [ ] The PLMN, TAC, and NSSAI settings match the Open5GS configuration.
- [ ] The UE subscriber information matches the Core Network configuration.
- [ ] No sensitive information (IMSI, Key, OPc, VPN credentials, etc.) is included in the public repository.

---

## 9.3 Runtime Verification

Confirm that:

- [ ] The CU successfully connects to the AMF.
- [ ] The CU receives the `NGSetupResponse`.
- [ ] The DU successfully establishes the F1 connection with the CU.
- [ ] The RF Simulator starts successfully.
- [ ] The UE successfully connects to the RF Simulator.
- [ ] Initial synchronization completes successfully.
- [ ] SIB1 is decoded successfully.
- [ ] The 4-Step Random Access procedure completes successfully.
- [ ] The UE reaches the `NR_RRC_CONNECTED` state.
- [ ] NAS Registration completes successfully.
- [ ] The PDU Session is established successfully.
- [ ] The UE receives an IPv4 address.
- [ ] The `oaitun_ue1` interface is created successfully.
- [ ] Data-plane connectivity (e.g., `ping`) has been verified.

If all of the above items are completed successfully, the OAI RF Simulator environment has been successfully deployed.

---

# 99. Troubleshooting

## 99.1 Unable to Clone the Repository

### Error

```text
fatal: unable to access 'https://github.com/...'
Could not resolve host
Temporary failure in name resolution
```

### Possible Causes

- The VM network adapter is disabled.
- VMware NAT or Bridged networking is misconfigured.
- DNS resolution is unavailable.
- The host computer has no Internet connection.

### Initial Checks

Verify that the VM has obtained an IP address:

```bash
ip addr
```

Test Internet connectivity:

```bash
ping -c 4 8.8.8.8
```

Test DNS resolution:

```bash
ping -c 4 github.com
```

- If `8.8.8.8` is reachable but `github.com` is not, the issue is likely related to DNS.
- If neither is reachable, the problem is likely caused by the VM network configuration.

### Solution

In VMware Workstation:

```text
Edit
→ Virtual Network Editor
→ Restore Defaults
```

Restart the VM and test the network connection again.

### Verification

Clone the repository again:

```bash
git clone <repository-url>
```

Confirm that the following directories have been created:

```text
openairinterface5g
5g-nr-rfsim-guides
```

---

## 99.2 Unable to Access the Laboratory Network

### Problem

The laboratory VM cannot be reached using `ping` or `ssh`.

### Possible Cause

The laboratory VPN has not been established.

### Solution

Connect to the laboratory network using the approved WireGuard configuration.

After connecting, verify network connectivity:

```bash
ping -c 4 <LAB_VM_IP>
```

Verify SSH access:

```bash
ssh <LAB_USER>@<LAB_VM_IP>
```

Obtain the SSH password from the laboratory administrator.

> **Do not store VPN configuration files, private keys, or passwords in the repository.**

![WireGuard Connected](../Images/oai_set_up_11.png)

![SSH Login](../Images/oai_set_up_12.png)

---

## 99.3 CU Displays "Cannot assign requested address"

### Error

```text
[GTPU] bind: Cannot assign requested address
[GTPU] failed to bind socket
[E1AP] Failed to create CUUP N3 UDP listener
```

### Possible Cause

The CU is attempting to bind to an IP address that does not belong to the current machine.

### Check

Verify the current IP address:

```bash
hostname -I
```

Check the routing information:

```bash
ip route get <AMF_IP>
```

Verify the CU configuration:

```bash
grep -nE \
'GNB_IPV4_ADDRESS_FOR_NG_AMF|GNB_IPV4_ADDRESS_FOR_NGU' \
"$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf"
```

### Solution

Update the following parameters with the correct local IP address:

```conf
GNB_IPV4_ADDRESS_FOR_NG_AMF = "<GNB_N2_IP>";
GNB_IPV4_ADDRESS_FOR_NGU    = "<GNB_N3_IP>";
```

Keep the AMF address unchanged:

```conf
amf_ip_address = ({ ipv4 = "<AMF_IP>"; });
```

---

## 99.4 CU Displays "Connection refused"

### Possible Causes

- The Open5GS AMF is not running.
- The AMF IP address is incorrect.
- The VPN connection has not been established.
- The SCTP service is unavailable.
- A firewall is blocking the connection.

### Check

```bash
ping -c 4 <AMF_IP>
```

Verify that the Open5GS AMF service is running.

---

## 99.5 Missing Configuration Files or Executables

### Error

```text
No such file or directory
```

### Check

Verify the current working directory:

```bash
pwd
```

Search for the repository:

```bash
find "$HOME" -type d -name "openairinterface5g"
```

Search for the configuration files:

```bash
find "$HOME" -name "cu_gnb.conf"
find "$HOME" -name "du_gnb.conf"
find "$HOME" -name "ue.conf"
```

Ensure that the correct repository is being used.

---

# Security Notes

The following information **must not** be included in a public repository:

- WireGuard private keys
- WireGuard pre-shared keys
- VPN configuration files
- SSH passwords
- GitHub Device Codes
- Personal Access Tokens (PATs)
- Production IMSI / SUPI
- Subscriber Keys
- OP / OPc
- Laboratory internal IP addresses (unless approved)
- Personal usernames

Instead, use placeholders such as:

```text
<LAB_USER>
<LAB_VM_IP>
<AMF_IP>
<GNB_N2_IP>
<GNB_N3_IP>
<TEST_IMSI>
<TEST_SUBSCRIBER_KEY>
<TEST_OPC>
<TESTED_COMMIT_SHA>
```

If sensitive information has already been committed, simply deleting the file is not sufficient because it may still exist in the Git history. Notify the repository administrator and rotate any exposed credentials immediately.

---

# Tested Version Record

Record the environment used for verification.

```text
Ubuntu Version:
OAI Commit SHA:
RF Simulator Guide Commit SHA:
Test Date:
Core Network:
VPN:
CU / DU / UE Result:
Data-plane Test Result:
```

Example:

```text
Ubuntu Version: Ubuntu 22.04.x LTS
OAI Commit SHA: <TESTED_COMMIT_SHA>
RF Simulator Guide Commit SHA: <GUIDE_COMMIT_SHA>
Test Date: YYYY-MM-DD
Core Network: Open5GS
VPN: WireGuard
CU / DU / UE Result: Passed
Data-plane Test Result: Passed
```
