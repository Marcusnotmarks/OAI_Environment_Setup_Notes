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

### 4. 建置特定目標
執行：
```bash
cd openairinterface5g/cmake_targets/ran_build/build

cmake --build . --target nr-softmodem nr-uesoftmodem ldpc params_libconfig oai_zmqdevif
```
結果應該是長這樣：
```bash
cmake --build . --target nr-softmodem nr-uesoftmodem ldpc params_libconfig oai_zmqdevif
[426/426] Linking CXX executable nr-softmodem
```
這一步的目的，是確認這幾個目標都有成功建出來，尤其是：
```bash
nr-softmodem
nr-uesoftmodem
liboai_zmqdevif.so
```
![安裝成功](../Images/nrue_setup_4.png)
建置成功後，建立 RF 模擬器函式庫的符號連結：
執行：
```bash
cp liboai_zmqdevif.so librfsimulator.so

ls -l librfsimulator.so
```
如果能看到 `librfsimulator.so`，就可以進到下一步：修改 `max_mimo_layers`。 ![安裝成功2](../Images/nrue_setup_5.png)

---

### 5. 原始碼修補
修改 `max_mimo_layers` ，回到專案根目錄： `cd ~/marcus/nrue/openairinterface5g` ，並執行 `nano openair2/LAYER2/NR_MAC_UE/nr_ue_procedures.c` 。<br>
搜尋：
```bash
AssertFatal(max_mimo_layers
```
![原始碼修補](../Images/nrue_setup_6.png)

修補以下程式：
```bash
int max_mimo_layers = 0;
if (sc_info->maxMIMO_Layers_PDSCH)
  max_mimo_layers = *sc_info->maxMIMO_Layers_PDSCH;
else
  max_mimo_layers = mac->uecap_maxMIMO_PDSCH_layers;

AssertFatal(max_mimo_layers > 0,
            "Invalid number of max MIMO layers for PDSCH\n");
```
在 `AssertFatal(max_mimo_layers` 之前加入程式：
```bash
if (max_mimo_layers == 0) {
  max_mimo_layers = 2;
}
```
儲存後，重新建置：
```bash
cd openairinterface5g/cmake_targets
./build_oai --nrUE --ninja -w ZMQ -c
```
![建置成功結果](../Images/nrue_setup_7.png)
接下來先確認執行檔與 `ZMQ library` 都還在：
```bash
-rwxrwxr-x 1 karl karl   674600  八   7 00:10 liboai_zmqdevif.so
-rwxrwxr-x 1 karl karl  1227304  八   7 00:10 librfsimulator.so
-rwxrwxr-x 1 karl karl 48028328  八   7 00:11 nr-uesoftmodem
```
