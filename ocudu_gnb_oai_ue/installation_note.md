# OAI NR-UE 安裝筆記

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

**1.** 複製儲存庫
```bash
git clone https://github.com/OPENAIRINTERFACE/openairinterface5g.git
cd openairinterface5g
```

**2.** 切換至 `develop` 分支
```bash
git checkout develop
git pull origin develop
```

**3.** 載入 oai 環境：
```bash
source oaienv
cd cmake_targets
./build_oai -I --gNB --nrUE --ninja -w ZMQ
```
![安裝相依套件](../Images/nure_setup_1.png)





