# OAI NR-UE 建置與驗證筆記

詳細記錄我在建立此 OAI 的各項步驟以及使用的程式。

## 前置條件

**1.** 建立一個新的資料夾：
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

## NRUE 安裝筆記

### 1. 複製儲存庫
```bash
git clone https://github.com/OPENAIRINTERFACE/openairinterface5g.git
cd openairinterface5g
```

---

### 2. 切換至 `develop` 分支
```bash
git checkout develop
git pull origin develop
```

---

### 3. 載入 oai 環境：
```bash
source oaienv
cd cmake_targets
./build_oai -I --gNB --nrUE --ninja -w ZMQ
```
![安裝相依套件](../Images/nure_setup_1.png)

輸入以下的程式，確認 `nr-uesoftmodem` 也有產生：
```bash
cd ~/marcus/nrue/openairinterface5g/cmake_targets/ran_build/build
ls -l nr-softmodem nr-uesoftmodem
```
![確認程式](../Images/nrue_setup_2.png)

確認版本 `grep -n "max_mimo_layers" ~/marcus/nrue/openairinterface5g/openair2/LAYER2/NR_MAC_UE/nr_ue_procedures.c` ：
![版本確認](../Images/nrue_setup_3.png)

---

### 4. 

```bash
cmake --build . --target nr-softmodem nr-uesoftmodem ldpc params_libconfig oai_zmqdevif
[426/426] Linking CXX executable nr-softmodem
```
