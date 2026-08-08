# OAI NR-UE (ZMQ) 與 OCUDU gNB 建置與驗證筆記

本文件詳細記錄 OAI NR-UE（ZMQ RF Simulator）與 OCUDU gNB 的建置流程、設定方式、驗證方法及常見問題，作為後續研究與 Lab Handover 的參考。

---

## Tested Environment

| Item | Version |
|------|---------|
| Ubuntu | 22.04.5 LTS |
| OAI Branch | develop |
| Radio | ZMQ RF Simulator |
| gNB | OCUDU |
| Core Network | Open5GS |

---

## 前置條件

**1.** 建立新的工作資料夾：

```bash
mkdir -p ~/marcus/nrue
cd ~/marcus/nrue
pwd
```

**2.** 安裝必要系統套件：

```bash
sudo apt-get update
sudo apt-get install libzmq3-dev libczmq-dev
```

---

# OAI NR-UE 安裝筆記

## 1. 複製儲存庫

```bash
git clone https://github.com/OPENAIRINTERFACE/openairinterface5g.git
cd openairinterface5g
```

---

## 2. 切換至 `develop` 分支

```bash
git checkout develop
git pull origin develop
```

---

## 3. 建立 OAI 編譯環境

載入 OAI 環境並安裝所有必要的相依套件：

```bash
source oaienv
cd cmake_targets
./build_oai -I --gNB --nrUE --ninja -w ZMQ
```

![安裝相依套件](../Images/nure_setup_1.png)

編譯完成後，確認 `nr-softmodem` 與 `nr-uesoftmodem` 是否已成功產生：

```bash
cd ~/marcus/nrue/openairinterface5g/cmake_targets/ran_build/build
ls -l nr-softmodem nr-uesoftmodem
```

![確認程式](../Images/nrue_setup_2.png)

確認目前 Source Code 中 `max_mimo_layers` 的位置：

```bash
grep -n "max_mimo_layers" \
~/marcus/nrue/openairinterface5g/openair2/LAYER2/NR_MAC_UE/nr_ue_procedures.c
```

![版本確認](../Images/nrue_setup_3.png)

---

## 4. 建置特定目標

進入 Build 目錄：

```bash
cd ~/marcus/nrue/openairinterface5g/cmake_targets/ran_build/build
```

執行：

```bash
cmake --build . --target nr-softmodem nr-uesoftmodem ldpc params_libconfig oai_zmqdevif
```

建置成功後，預期輸出類似：

```text
[426/426] Linking CXX executable nr-softmodem
```

此步驟會建置 OAI 的主要執行檔與 ZMQ RF Simulator Library，確認後續 NR-UE 執行所需的 Binary 已成功建立。

應確認以下檔案存在：

```text
nr-softmodem
nr-uesoftmodem
liboai_zmqdevif.so
```

![安裝成功](../Images/nrue_setup_4.png)

建置成功後，建立 RF Simulator Library：

```bash
cp liboai_zmqdevif.so librfsimulator.so
```

確認：

```bash
ls -l librfsimulator.so
```

若能看到 `librfsimulator.so`，即可進入下一步原始碼修補。

![安裝成功2](../Images/nrue_setup_5.png)

---

## 5. 原始碼修補

為了避免 `max_mimo_layers` 為 `0` 時造成 Assertion Failure，需要修改 OAI NR-UE Source Code。

回到專案根目錄：

```bash
cd ~/marcus/nrue/openairinterface5g
```

開啟：

```bash
nano openair2/LAYER2/NR_MAC_UE/nr_ue_procedures.c
```

搜尋：

```text
AssertFatal(max_mimo_layers
```

![原始碼修補](../Images/nrue_setup_6.png)

找到以下程式碼：

```c
int max_mimo_layers = 0;
if (sc_info->maxMIMO_Layers_PDSCH)
  max_mimo_layers = *sc_info->maxMIMO_Layers_PDSCH;
else
  max_mimo_layers = mac->uecap_maxMIMO_PDSCH_layers;

AssertFatal(max_mimo_layers > 0,
            "Invalid number of max MIMO layers for PDSCH\n");
```

在 `AssertFatal(max_mimo_layers...)` 前加入：

```c
if (max_mimo_layers == 0) {
  max_mimo_layers = 2;
}
```

修改完成後應類似：

```c
int max_mimo_layers = 0;
if (sc_info->maxMIMO_Layers_PDSCH)
  max_mimo_layers = *sc_info->maxMIMO_Layers_PDSCH;
else
  max_mimo_layers = mac->uecap_maxMIMO_PDSCH_layers;

if (max_mimo_layers == 0) {
  max_mimo_layers = 2;
}

AssertFatal(max_mimo_layers > 0,
            "Invalid number of max MIMO layers for PDSCH\n");
```

儲存後重新建置 NR-UE：

```bash
cd ~/marcus/nrue/openairinterface5g/cmake_targets
./build_oai --nrUE --ninja -w ZMQ -c
```

![建置成功結果](../Images/nrue_setup_7.png)

重新建置後，確認執行檔與 ZMQ Library 仍存在：

```bash
cd ~/marcus/nrue/openairinterface5g/cmake_targets/ran_build/build
ls -l liboai_zmqdevif.so librfsimulator.so nr-uesoftmodem
```

---

## 6. UE 設定

先搜尋目前 Repository 中可使用的 UE Configuration：

```bash
find ~/marcus/nrue/openairinterface5g \
-name "oai_ue.conf" -o -name "ue.conf"
```

![查詢結果](../Images/nrue_setup_8.png)

若沒有自己的設定檔，可建立專用資料夾並複製：

```bash
cd ~/marcus/nrue/openairinterface5g/targets/PROJECTS/GENERIC-NR-5GC/CONF

mkdir -p marcus

cp ue.conf marcus/oai_ue.conf
```

UE 設定檔路徑：

```text
openairinterface5g/targets/PROJECTS/GENERIC-NR-5GC/CONF/marcus/oai_ue.conf
```

修改以下參數：

- `imsi`
- `key`
- `opc`
- `dnn`
- `nssai_sst`

原始設定可能類似：

```conf
uicc0 = {
  imsi = "<ORIGINAL_IMSI>";
  key = "<ORIGINAL_KEY>";
  opc = "<ORIGINAL_OPC>";
  pdu_sessions = ({
    dnn = "oai";
    nssai_sst = 1;
  });
}

position0 = {
    x = 0.0;
    y = 0.0;
    z = 6377900.0;
}
```

請修改為與目前 Open5GS Subscriber Database 相符的設定：

```conf
uicc0 = {
  imsi = "<TEST_IMSI>";
  key = "<TEST_KEY>";
  opc = "<TEST_OPC>";
  pdu_sessions = ({
    dnn = "Internet";
    nssai_sst = 1;
  });
}

position0 = {
    x = 0.0;
    y = 0.0;
    z = 6377900.0;
}
```

> **注意：** IMSI、Key、OPc 等 Subscriber Credential 不建議直接公開於 Public Repository，GitHub 文件中應改為 Placeholder。

確認 RF Simulator Library 是否存在：

```bash
ls -l ~/marcus/nrue/openairinterface5g/cmake_targets/ran_build/build/librfsimulator.so
```

![查詢結果](../Images/nrue_setup_9.png)

---

## 7. 啟動 NR-UE

成功啟動後，ZMQ Driver 將完成初始化，並開始等待 gNB 建立連線。

進入 Build 目錄：

```bash
cd ~/marcus/nrue/openairinterface5g/cmake_targets/ran_build/build
```

執行：

```bash
sudo ./nr-uesoftmodem \
-r 106 \
--numerology 1 \
--band 78 \
-C 3489420000 \
--ssb 42 \
--rfsim \
-O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/marcus/oai_ue.conf \
--ue-nb-ant-tx 2 \
--ue-nb-ant-rx 2 \
--log_config.global_log_options level,nocolor \
--device.name oai_zmqdevif \
--zmq.[0].tx_channels "tcp://127.0.0.1:4556,tcp://127.0.0.1:4557" \
--zmq.[0].rx_channels "tcp://127.0.0.1:4558,tcp://127.0.0.1:4559"
```

成功啟動後，預期輸出如下：

```text
[CONFIG] function config_libconfig_init returned 0
[SIM] UICC simulation...
[HW] [RAU] has loaded RFSIMULATOR device.
[HW] [ZMQ] TX socket ...
[HW] [ZMQ] RX socket ...
```

![啟動成功結果](../Images/nrue_setup_10.png)

> 若 OCUDU gNB 尚未啟動，NR-UE 會停留在等待同步與 gNB 訊號的狀態。

---

# OCUDU gNB 安裝筆記

## 1. 複製儲存庫

執行：

```bash
git clone https://gitlab.com/ocudu/ocudu.git
cd ocudu
```

---

## 2. 建置

建立 Build 目錄：

```bash
mkdir build
cd build
```

產生 Build Files：

```bash
cmake ../
```

開始編譯：

```bash
make -j 8
```

若主機資源較多，也可依實際 CPU Core 數量調整平行任務數量。

---

## 3. 測試

執行：

```bash
make test -j $(nproc)
```

若所有測試皆通過，預期顯示：

```text
100% tests passed
```

代表單元測試皆成功通過。

![測試成功結果](../Images/nrue_setup_11.png)

---

## 4. 安裝 gNB 執行檔

切換至 gNB Application 目錄：

```bash
cd apps/gnb
```

安裝：

```bash
sudo make install
```

確認是否安裝成功：

```bash
which gnb
```

預期：

```text
/usr/local/bin/gnb
```

---

## 5. 建立並確認 `test.yaml`

先搜尋系統中是否已存在 `test.yaml`：

```bash
find ~ -name "test.yaml"
```

若 OCUDU Repository 中沒有提供可直接使用的 `test.yaml`，請自行建立。

例如：

```bash
nano test.yaml
```

建立完成後，至少需確認以下設定：

- AMF IP
- gNB Bind Address
- PLMN
- TAC
- Band
- DL ARFCN
- Channel Bandwidth
- ZMQ TX / RX Port
- Antenna Configuration

範例：

```yaml
cu_cp:
  amf:
    addr: <AMF_IP>
    port: 38412
    bind_addr: <GNB_BIND_IP>
    supported_tracking_areas:
      - tac: 1
        plmn_list:
          - plmn: "00101"
            tai_slice_support_list:
              - sst: 1

ru_sdr:
  device_driver: zmq
  device_args: tx_port=tcp://127.0.0.1:4558,tx_port=tcp://127.0.0.1:4559,rx_port=tcp://127.0.0.1:4556,rx_port=tcp://127.0.0.1:4557
  srate: 61.44
  tx_gain: 0
  rx_gain: 0

cell_cfg:
  dl_arfcn: 632628
  band: 78
  channel_bandwidth_MHz: 40
  common_scs: 30
  plmn: "00101"
  tac: 1
  pci: 1
  nof_antennas_ul: 2
  nof_antennas_dl: 2
  tdd_ul_dl_cfg:
    dl_ul_tx_period: 5
    nof_dl_slots: 3
    nof_dl_symbols: 10
    nof_ul_slots: 1
    nof_ul_symbols: 2
  csi:
    csi_rs_enabled: false
  pucch:
    formats: f0_and_f2
    nof_cell_csi_res: 0

log:
  filename: gnb.log
  all_level: debug
```

> **注意：** `<AMF_IP>` 與 `<GNB_BIND_IP>` 請依照實際 Lab Network 設定填入。

---

## 6. 啟動 gNB

進入包含 `test.yaml` 的 gNB 目錄：

```bash
cd ~/marcus/nrue/openairinterface5g/cmake_targets/ran_build/build/ocudu/build/apps/gnb
```

執行：

```bash
sudo ./gnb -c test.yaml
```

成功啟動後，預期輸出類似：

```text
--== OCUDU gNB (commit <COMMIT_SHA>) ==--

Lower PHY in executor sequential baseband mode.
Available radio types: uhd, zmq and realtime_loopback.

Cell pci=1, bw=40 MHz, 2T2R, dl_arfcn=632628 (n78), dl_freq=3489.42 MHz, ...

N2: Connection to AMF on <AMF_IP>:38412 completed

==== gNB started ====

Type <h> to view help
```

![gNB安裝成功結果](../Images/nrue_srtup_13.png)

當出現：

```text
N2: Connection to AMF ... completed
```

表示 gNB 已成功與 Open5GS AMF 建立 N2 Connection。

當出現：

```text
==== gNB started ====
```

表示 OCUDU gNB 已成功啟動。

---

# Verification Checklist

確認下列項目皆成功後，即表示 OAI NR-UE 與 OCUDU gNB 已成功完成連線。

| Item | Status |
|------|:------:|
| OCUDU gNB Started | ✅ |
| N2 Connected to AMF | ✅ |
| ZMQ RF Simulator Loaded | ✅ |
| Initial Sync Successful | ✅ |
| SIB1 Decoded | ✅ |
| Random Access Success | ✅ |
| `NR_RRC_CONNECTED` | ✅ |
| Registration Accept | ✅ |
| PDU Session Accept | ✅ |
| UE IPv4 Assigned | ✅ |
| TUN Interface (`oaitun_ue1`) Created | ✅ |

---

# Successful Result

成功建立 OCUDU gNB 與 OAI NR-UE 連線後，可於 UE Log 中觀察到以下關鍵訊息：

```text
Initial sync successful

SIB1 decoded

4-Step RA procedure succeeded

[NR_RRC] State = NR_RRC_CONNECTED

[NAS] Received Registration Accept with result 3GPP

[NAS] Received PDU Session Establishment Accept

UE IPv4: 10.45.x.x

[OIP] TUN Interface oaitun_ue1 successfully configured
```

以上訊息代表：

- UE 已成功完成 Initial Synchronization。
- SIB1 已成功解碼。
- Random Access Procedure 已完成。
- UE 已成功建立 RRC Connection。
- NAS Registration 已成功完成。
- PDU Session 已成功建立。
- Open5GS 已成功分配 UE IPv4。
- TUN Interface (`oaitun_ue1`) 已成功建立。

成功 Log 建議保存於：

```text
log/
├── ue.log
└── gnb.log
```

方便後續不同版本、設定或實驗結果之間進行比對。

---

# 99. Troubleshooting

## 1. 找不到 `uecap.xml`

### 遇到的問題

執行：

```bash
ls -l /opt/oai-nr-ue/etc/uecap.xml
```

出現：

```text
ls: cannot access '/opt/oai-nr-ue/etc/uecap.xml': No such file or directory
```

### 可能原因

目前使用 Source Build 建立 OAI，因此系統中沒有 `/opt/oai-nr-ue/etc/` 目錄，也不存在 `uecap.xml`。

### 解決方法

將啟動指令中的：

```bash
--uecap_file /opt/oai-nr-ue/etc/uecap.xml
```

移除後重新執行 NR-UE。

### 結果

NR-UE 可正常啟動，並成功載入 ZMQ RF Simulator。

---

## 2. `cannot open include file`

### 遇到的問題

第一次啟動 NR-UE 時出現：

```text
file .../marcus/oai_ue.conf - line 16: cannot open include file

config module "libconfig" couldn't be loaded

Segmentation fault
```

### 可能原因

將 `ue.conf` 複製至 `CONF/marcus/` 後，`@include` 使用原本的相對路徑，因此無法找到引用的 Configuration File。

### 解決方法

將：

```conf
@include "channelmod_rfsimu_LEO_satellite.conf"
```

修改為：

```conf
@include "../channelmod_rfsimu_LEO_satellite.conf"
```

### 結果

設定檔可成功載入，NR-UE 可正常啟動。

---

## 3. 找不到 `test.yaml`

### 遇到的問題

執行：

```bash
find ~ -name "test.yaml"
```

找不到任何可供 OCUDU gNB 使用的 `test.yaml`。

### 可能原因

目前使用的 OCUDU Repository 未提供符合目前實驗環境的 `test.yaml`。

### 解決方法

於 gNB 執行目錄自行建立：

```bash
nano test.yaml
```

並確認以下參數符合目前 Lab Network 與 NR-UE 設定：

```text
AMF IP
gNB Bind IP
PLMN
TAC
Band
DL ARFCN
Channel Bandwidth
ZMQ TX Port
ZMQ RX Port
Antenna Configuration
```

### 結果

設定完成後執行：

```bash
sudo ./gnb -c test.yaml
```

成功時可看到：

```text
N2: Connection to AMF on <AMF_IP>:38412 completed

==== gNB started ====
```

表示 OCUDU gNB 已成功啟動並與 AMF 建立連線。

---



