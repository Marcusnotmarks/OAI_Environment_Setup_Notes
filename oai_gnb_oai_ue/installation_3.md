# OAI gNB（CU/DU 分離）與 OAI NR-UE RF 模擬安裝筆記

本文件記錄 OAI gNB（CU/DU Split）與 OAI NR-UE RF Simulator 的環境建置、設定、啟動及驗證流程。

> 本文件使用的帳號、IP、VPN 資訊及 UE 訂閱資料皆以 Placeholder 表示。  
> 實際參數請向實驗室管理者確認，請勿將密碼、Private Key 或正式訂閱資料提交至 GitHub。

---

## 前置作業

建置 OAI 環境前，需要準備一個 Ubuntu 22.04 LTS 環境。

建議需求：

- Ubuntu 22.04 LTS
- VMware Workstation
- Git
- 可正常連線至網際網路
- 如需存取實驗室內網，須先建立實驗室 VPN
- 足夠的 CPU、RAM 與磁碟空間

---

## 工作目錄設定

為避免文件中出現個人帳號或固定路徑，本文使用以下環境變數：

```bash
export WORKDIR="$HOME/oai-test"
export OAI_DIR="$WORKDIR/openairinterface5g"
export GUIDE_DIR="$WORKDIR/5g-nr-rfsim-guides"
export BUILD_DIR="$OAI_DIR/cmake_targets/ran_build/build"
```

建立工作目錄：

```bash
mkdir -p "$WORKDIR"
cd "$WORKDIR"
```

確認：

```bash
pwd
```

後續指令皆以 `$WORKDIR`、`$OAI_DIR`、`$GUIDE_DIR` 與 `$BUILD_DIR` 表示，不需要改成個人的 `/home/<username>/...`。

> 若重新開啟 Terminal，需重新執行上述 `export` 指令，或將它們加入自己的 Shell 設定檔。

---

# OAI 環境建立紀錄

## 1. 確認 Ubuntu 版本

在終端機輸入：

```bash
lsb_release -a
```

確認版本為 Ubuntu 22.04.x LTS，例如：

```text
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 22.04.5 LTS
Release:        22.04
Codename:       jammy
```

---

## 2. 更新基本套件

輸入：

```bash
sudo apt update
```

此步驟只會更新套件索引，不等同於執行完整系統升級。

![更新基本套件](../Images/oai_setup_2.png)

---

## 3. 安裝 Git

安裝 Git：

```bash
sudo apt install -y git
```

確認版本：

```bash
git --version
```

測試環境中的結果例如：

```text
git version 2.34.1
```

Git 版本不一定需要完全相同，只要能正常執行後續 Clone、Checkout 與 Pull 即可。

---

## 4. Clone OAI 並固定測試版本

### 4.1 Clone OAI

```bash
cd "$WORKDIR"

git clone https://github.com/OPENAIRINTERFACE/openairinterface5g.git
```

進入 Repository：

```bash
cd "$OAI_DIR"
pwd
```

### 4.2 取得 develop 分支

```bash
git checkout develop
git pull origin develop
```

![Clone OAI](../Images/oai_setup_3.png)

### 4.3 記錄並固定測試 Commit

`develop` 分支持續更新，只記錄 `develop` 無法保證未來能重現相同結果。

查詢目前 Commit：

```bash
git rev-parse HEAD
```

將輸出的完整 SHA 記錄在文件或 README：

```text
Tested OAI commit: <TESTED_COMMIT_SHA>
```

之後重新建置時，應切換至該 Commit：

```bash
git checkout <TESTED_COMMIT_SHA>
```

確認：

```bash
git rev-parse HEAD
```

> `<TESTED_COMMIT_SHA>` 必須替換為本次實際驗證成功的 SHA，不應自行填寫或猜測。

---

## 5. 建立 OAI 編譯環境

### 5.1 進入建置目錄

```bash
cd "$OAI_DIR/cmake_targets"
pwd
```

### 5.2 載入 OAI 環境

```bash
source oaienv
```

![載入 OAI 環境](../Images/oai_setup_4.png)

### 5.3 安裝相依套件

```bash
./build_oai -I --install-optional-packages
```

此步驟會安裝 OAI 編譯所需的相依套件，例如：

- GCC / G++
- CMake
- ASN.1 Compiler
- UHD
- Wireshark
- SCTP Library
- 其他 OAI 所需函式庫

![安裝相依套件](../Images/oai_setup_5.png)

---

## 6. 編譯 gNB 與 NR-UE

### 6.1 編譯

```bash
cd "$OAI_DIR/cmake_targets"

./build_oai --gNB --nrUE
```

若成功，輸出末尾通常會出現：

```text
BUILD SHOULD BE SUCCESSFUL
```

![gNB 與 NR-UE 編譯成功](../Images/oai_setup_6.png)

### 6.2 確認可執行檔

```bash
cd "$BUILD_DIR"

ls -l nr-softmodem nr-uesoftmodem
```

應可看到：

```text
nr-softmodem
nr-uesoftmodem
```

![確認可執行檔](../Images/oai_setup_7.png)

### 6.3 搜尋 OAI 內建設定檔

```bash
cd "$OAI_DIR"

find . -name "cu_gnb.conf"
find . -name "du_gnb.conf"
find . -name "ue.conf"
```

可能找到：

```text
./targets/PROJECTS/GENERIC-NR-5GC/CONF/cu_gnb.conf
./targets/PROJECTS/GENERIC-NR-5GC/CONF/du_gnb.conf
./targets/PROJECTS/GENERIC-NR-5GC/CONF/ue.conf
```

> OAI 原始碼內的設定檔是範例。  
> 本實驗後續執行時，使用的是 `5g-nr-rfsim-guides` Repository 內的設定檔，兩者請勿混淆。

### 6.4 再次確認 Build Directory

```bash
cd "$BUILD_DIR"
pwd
ls
```

![Build Directory](../Images/oai_setup_8.png)

---

## 7. 設定 CU、DU 與 UE

### 7.1 GitHub Private Repository 驗證

若設定檔 Repository 為 Private Repository，需要先完成 GitHub 驗證。

安裝 GitHub CLI：

```bash
sudo apt update
sudo apt install -y gh
```

執行：

```bash
gh auth login
```

依畫面選擇：

```text
GitHub.com
HTTPS
Authenticate Git with your GitHub credentials
Login with a web browser
```

系統會顯示一次性的 Device Code。請在自己的瀏覽器完成授權。

> Device Code 為一次性登入資訊，不應寫入文件、截圖或提交至 GitHub。

確認登入狀態：

```bash
gh auth status
```

![GitHub CLI 登入成功](../Images/oai_set_up_10.png)

### 7.2 Clone RF Simulator 設定檔 Repository

```bash
cd "$WORKDIR"

git clone https://github.com/bmw-ece-ntust/5g-nr-rfsim-guides.git
```

確認設定檔：

```bash
ls "$GUIDE_DIR/oai_gnb_oai_ue/config"
```

應看到：

```text
cu_gnb.conf
du_gnb.conf
ue.conf
```

![Clone 設定檔 Repository](../Images/oai_setup_9.png)

### 7.3 建立設定檔備份

修改前先備份：

```bash
cp "$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf" \
   "$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf.bak"

cp "$GUIDE_DIR/oai_gnb_oai_ue/config/du_gnb.conf" \
   "$GUIDE_DIR/oai_gnb_oai_ue/config/du_gnb.conf.bak"

cp "$GUIDE_DIR/oai_gnb_oai_ue/config/ue.conf" \
   "$GUIDE_DIR/oai_gnb_oai_ue/config/ue.conf.bak"
```

### 7.4 確認網路參數

設定下列參數前，請向實驗室管理者確認：

```bash
export AMF_IP="<AMF_IP>"
export GNB_N2_IP="<GNB_N2_IP>"
export GNB_N3_IP="<GNB_N3_IP>"
```

參數意義：

| 參數 | 說明 |
|---|---|
| `AMF_IP` | Open5GS AMF 的 IP |
| `GNB_N2_IP` | gNB/CU 連接 AMF 時使用的本機 IP |
| `GNB_N3_IP` | gNB/CU 建立 GTP-U 資料面時使用的本機 IP |

查詢目前主機 IP：

```bash
hostname -I
```

也可以確認連向 AMF 時所使用的來源 IP：

```bash
ip route get "$AMF_IP"
```

輸出中的 `src` 通常就是該介面應使用的本機 IP。

### 7.5 修改 CU 設定檔

開啟：

```bash
nano "$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf"
```

將：

```conf
amf_ip_address = ({ ipv4 = "<AMF_IP>"; });

GNB_IPV4_ADDRESS_FOR_NG_AMF = "<GNB_N2_IP>";
GNB_IPV4_ADDRESS_FOR_NGU    = "<GNB_N3_IP>";
```

替換為實際環境參數。

也可以使用環境變數替換既有值，但執行前必須確認舊值：

```bash
grep -nE \
'amf_ip_address|GNB_IPV4_ADDRESS_FOR_NG_AMF|GNB_IPV4_ADDRESS_FOR_NGU' \
"$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf"
```

不建議在公開文件內寫死實驗室 IP。

### 7.6 驗證 CU 設定

```bash
grep -nE \
'amf_ip_address|GNB_IPV4_ADDRESS_FOR_NG_AMF|GNB_IPV4_ADDRESS_FOR_NGU' \
"$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf"
```

預期格式：

```text
amf_ip_address = ({ ipv4 = "<AMF_IP>"; });
GNB_IPV4_ADDRESS_FOR_NG_AMF = "<GNB_N2_IP>";
GNB_IPV4_ADDRESS_FOR_NGU    = "<GNB_N3_IP>";
```

### 7.7 檢查 UE 設定中的敏感資料

`ue.conf` 可能包含：

- IMSI / SUPI
- Subscriber Key
- OP / OPc
- DNN
- NSSAI

公開 Repository 中應改用 Placeholder：

```conf
imsi = "<TEST_IMSI>";
key = "<TEST_SUBSCRIBER_KEY>";
opc = "<TEST_OPC>";
dnn = "<TEST_DNN>";
```

若 Repository 內保留測試值，必須明確標示：

```text
Demo/Test Only
Do not use in production or public networks.
```

---

## 8. 啟動 CU、DU 與 UE

### 8.1 啟動順序

開啟三個 Terminal，依序執行：

```text
Terminal 1：CU
Terminal 2：DU
Terminal 3：UE
```

每個 Terminal 先重新設定工作路徑，或直接使用完整的環境變數設定。

若新 Terminal 尚未設定變數，先輸入：

```bash
export WORKDIR="$HOME/oai-test"
export OAI_DIR="$WORKDIR/openairinterface5g"
export GUIDE_DIR="$WORKDIR/5g-nr-rfsim-guides"
export BUILD_DIR="$OAI_DIR/cmake_targets/ran_build/build"
```

進入 Build Directory：

```bash
cd "$BUILD_DIR"
```

### 8.2 Terminal 1：啟動 CU

```bash
sudo ./nr-softmodem \
  --rfsim \
  -O "$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf"
```

CU 成功時可看到：

```text
Received NGSetupResponse from AMF
Received NGAP_REGISTER_GNB_CNF
Received F1 Setup Request from gNB_DU
sending F1 Setup Response
```

![CU 成功結果 1](../Images/oai_setup_13.png)

![CU 成功結果 2](../Images/oai_setup_14.png)

### 8.3 Terminal 2：啟動 DU

```bash
sudo ./nr-softmodem \
  --rfsim \
  -O "$GUIDE_DIR/oai_gnb_oai_ue/config/du_gnb.conf"
```

DU 成功時可看到：

```text
Starting F1AP at DU
received F1 Setup Response from CU
RU 0 RF started
CBRA procedure succeeded
```

![DU 成功結果](../Images/oai_setup_15.png)

### 8.4 Terminal 3：啟動 UE

```bash
sudo ./nr-uesoftmodem \
  -r 106 \
  --numerology 1 \
  --band 78 \
  -C 3619200000 \
  --rfsim \
  -O "$GUIDE_DIR/oai_gnb_oai_ue/config/ue.conf"
```

UE 成功時可看到：

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

![UE 成功結果](../Images/oai_setup_16.png)

### 8.5 結束執行

建議依下列順序，在各 Terminal 按下 `Ctrl + C`：

1. UE
2. DU
3. CU

結束程式不會刪除 OAI、設定檔或編譯結果。下次只需再次執行 CU、DU、UE 的啟動指令，不需重新 Clone 或 Build。

---

# 9. 驗證 Checklist

## 9.1 Build 驗證

- [ ] 已確認 Ubuntu 22.04 LTS
- [ ] 已成功 Clone OAI
- [ ] 已切換至並記錄固定的 OAI Commit SHA
- [ ] `build_oai -I` 成功
- [ ] `build_oai --gNB --nrUE` 成功
- [ ] `nr-softmodem` 存在
- [ ] `nr-uesoftmodem` 存在

## 9.2 Config 驗證

- [ ] `cu_gnb.conf` 存在
- [ ] `du_gnb.conf` 存在
- [ ] `ue.conf` 存在
- [ ] `amf_ip_address` 指向正確 AMF
- [ ] `GNB_IPV4_ADDRESS_FOR_NG_AMF` 使用正確本機 IP
- [ ] `GNB_IPV4_ADDRESS_FOR_NGU` 使用正確本機 IP
- [ ] PLMN、TAC、NSSAI 與 Core Network 相符
- [ ] UE 測試訂閱資料與 Core Network 相符
- [ ] 公開文件未包含正式 IMSI、Key、OPc 或 VPN Private Key

## 9.3 執行驗證

- [ ] CU 成功連線 AMF
- [ ] CU 收到 `NGSetupResponse`
- [ ] DU 成功透過 F1 連線 CU
- [ ] DU RF Simulator 成功啟動
- [ ] UE 成功連線 RF Simulator
- [ ] UE 完成 Initial Sync
- [ ] UE 成功解碼 SIB1
- [ ] UE 完成 4-Step Random Access
- [ ] UE 進入 `NR_RRC_CONNECTED`
- [ ] NAS Registration 成功
- [ ] PDU Session 建立成功
- [ ] UE 取得 IPv4
- [ ] `oaitun_ue1` 建立成功
- [ ] 已完成實際資料面測試，例如 Ping

只有前述項目全部完成，才能判定環境已完整重現。

---

# 99. 遇到的問題

## 99.1 Clone 時 Ubuntu 無法連上網路

### 問題

在 VM 中 Clone Repository 時出現：

```text
fatal: unable to access 'https://github.com/...'
Could not resolve host
Temporary failure in name resolution
```

### 可能原因

- VM Network Adapter 未啟用
- VMware NAT / Bridge 設定異常
- DNS 設定異常
- Windows 主機本身沒有網路
- VPN 改變了路由設定

### 初步檢查

確認是否取得 IP：

```bash
ip addr
```

測試網路路由：

```bash
ping -c 4 8.8.8.8
```

測試 DNS：

```bash
ping -c 4 github.com
```

- 若 `8.8.8.8` 可通但 `github.com` 不通，通常是 DNS 問題。
- 若兩者都不通，通常是 VM 網路或路由問題。

### 本次使用的修復方法

在 VMware 主畫面依序選擇：

```text
Edit
→ Virtual Network Editor
→ Restore Defaults
```

重新啟動 VM 後再次測試網路。

### 驗證

```bash
git clone <repository-url>
```

並確認工作目錄中出現：

```text
openairinterface5g
5g-nr-rfsim-guides
```

---

## 99.2 無法存取實驗室內網

### 問題

無法 Ping 或 SSH 到實驗室 VM，亦無法連線實驗室 AMF。

### 原因

使用者位於實驗室網路之外，尚未建立實驗室 VPN。

### 解決方法

使用實驗室核准的 WireGuard 設定建立 VPN。

> WireGuard `.conf`、Private Key、Preshared Key 與 Endpoint 認證資訊不得提交至 GitHub。

連線後測試：

```bash
ping -c 4 <LAB_VM_IP>
```

SSH：

```bash
ssh <LAB_USER>@<LAB_VM_IP>
```

密碼請向實驗室管理者取得，不應寫入文件。

![WireGuard 連線成功](../Images/oai_set_up_11.png)

![SSH 連線成功](../Images/oai_set_up_12.png)

---

## 99.3 CU 出現 `Cannot assign requested address`

### 問題

```text
[GTPU] bind: Cannot assign requested address
[GTPU] failed to bind socket
[E1AP] Failed to create CUUP N3 UDP listener
```

### 原因

此錯誤通常不是單純因為「沒有 VPN」，而是 CU 嘗試綁定一個不屬於目前主機的本機 IP。

例如：

```conf
GNB_IPV4_ADDRESS_FOR_NG_AMF = "<OLD_GNB_IP>";
GNB_IPV4_ADDRESS_FOR_NGU    = "<OLD_GNB_IP>";
```

但目前主機沒有 `<OLD_GNB_IP>`。

### 檢查

```bash
hostname -I
```

```bash
ip route get <AMF_IP>
```

```bash
grep -nE \
'GNB_IPV4_ADDRESS_FOR_NG_AMF|GNB_IPV4_ADDRESS_FOR_NGU' \
"$GUIDE_DIR/oai_gnb_oai_ue/config/cu_gnb.conf"
```

### 解決方法

將兩個 gNB 本機位址改成目前主機實際持有的 IP：

```conf
GNB_IPV4_ADDRESS_FOR_NG_AMF = "<GNB_N2_IP>";
GNB_IPV4_ADDRESS_FOR_NGU    = "<GNB_N3_IP>";
```

`amf_ip_address` 則應保持為 AMF 主機的實際 IP：

```conf
amf_ip_address = ({ ipv4 = "<AMF_IP>"; });
```

兩者角色不同：

- `amf_ip_address`：遠端 AMF IP
- `GNB_IPV4_ADDRESS_FOR_*`：本機 gNB/CU IP

---

## 99.4 CU 出現 `Connection refused`

### 問題

```text
[SCTP] Connect failed: Connection refused
```

### 可能原因

- AMF 尚未啟動
- `amf_ip_address` 錯誤
- VPN 或路由尚未建立
- AMF 沒有在 SCTP Port 38412 監聽
- 防火牆拒絕連線

### 檢查

```bash
ping -c 4 <AMF_IP>
```

若實驗室允許，也可測試 AMF 服務狀態或請管理者確認 Open5GS AMF 是否運行。

---

## 99.5 找不到設定檔或執行檔

### 問題

```text
No such file or directory
```

### 檢查目前位置

```bash
pwd
```

搜尋 Repository：

```bash
find "$HOME" -type d -name "openairinterface5g" 2>/dev/null
```

搜尋設定檔：

```bash
find "$HOME" -name "cu_gnb.conf" 2>/dev/null
find "$HOME" -name "du_gnb.conf" 2>/dev/null
find "$HOME" -name "ue.conf" 2>/dev/null
```

請確認目前使用的是哪一份 Clone，避免混用不同工作目錄。

---

# Security Notes

公開 Repository 不應包含：

- WireGuard Private Key
- WireGuard Preshared Key
- VPN 設定檔
- SSH 密碼
- GitHub Device Code
- Personal Access Token
- 正式 IMSI / SUPI
- Subscriber Key
- OP / OPc
- 未經允許公開的實驗室內部 IP
- 個人帳號名稱

建議統一使用：

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

若敏感資訊曾經 Commit，僅刪除目前檔案可能不足，因為 Git History 仍可能保存舊內容。應立即通知 Repository 管理者處理並更換已洩漏的憑證。

---

# 測試版本紀錄

完成測試後填入：

```text
Ubuntu version:
OAI commit SHA:
RF simulator guide commit SHA:
Test date:
Core network:
VPN:
CU/DU/UE result:
Data-plane test result:
```

範例格式：

```text
Ubuntu version: Ubuntu 22.04.x LTS
OAI commit SHA: <TESTED_COMMIT_SHA>
RF simulator guide commit SHA: <GUIDE_COMMIT_SHA>
Test date: YYYY-MM-DD
Core network: Open5GS
VPN: WireGuard
CU/DU/UE result: Passed
Data-plane test result: Passed / Not tested
```
