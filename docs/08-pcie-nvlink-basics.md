# PCIe 與 NVLink 必備知識

## 1. 各技術的角色

| 技術 | 定義與用途 |
|---|---|
| PCIe（PCI Express） | 系統內的高速互連，連接 CPU、GPU、SSD、網路卡等裝置 |
| NVLink | NVIDIA 高速互連技術；GPU 間大量交換資料時，支援的平台通常可利用它提升通訊效能 |
| NVSwitch | 連接多個 NVLink 埠的交換晶片，讓多顆 GPU 同時交換資料 |
| NVMe（Non-Volatile Memory Express） | 儲存通訊協定；本機 NVMe SSD 通常透過 PCIe 傳輸 |
| M.2 | 外形與連接器規格；M.2 SSD 可能使用 PCIe/NVMe，也可能使用 SATA |

NVSwitch 可讓多組 GPU 同時通訊，並不是一次只選一組連通的 MUX。實際頻寬仍受 GPU 埠與交換網路限制。

```text
GPU A ── NVLink ── NVSwitch ── NVLink ── GPU B
```

NVLink 並非所有 GPU 都支援；支援它也不代表所有軟體會自動把多顆 GPU 當成一顆使用。

## 2. PCIe 如何傳輸

PCIe 採用點對點、序列式、封包化傳輸。

| 協定層 | 主要工作 |
|---|---|
| Transaction Layer／交易層 | 描述記憶體讀寫、設定與訊息等交易 |
| Data Link Layer／資料鏈結層 | 管理相鄰節點間的可靠傳送 |
| Physical Layer／實體層（PHY） | 處理高速收發、編碼、Lane 與連線訓練等 |

連線啟動時經過 Link Training，建立雙方可用的速率與寬度。多 Lane 連線由硬體分配與重組資料；一般程式不能指定某份資料只走 Lane 7。

### Lane 與訊號數量

每條 Lane 包含一對 TX 差動訊號與一對 RX 差動訊號，可同時雙向傳輸。一端 TX 接另一端 RX。

| 寬度 | TX 差動對 | RX 差動對 | 資料訊號導線總數 |
|---|---:|---:|---:|
| x1 | 1 | 1 | 4 |
| x4 | 4 | 4 | 16 |
| x8 | 8 | 8 | 32 |
| x16 | 16 | 16 | 64 |

Gen 代表世代／每 Lane 速率；xN 代表 Lane 數。實際頻寬還受封包開銷、裝置效能與共用上行連線影響。

### 其他常見訊號與供電

以下以標準 PCIe 擴充卡介面為主，其他連接器須查各自規格。

| 項目 | 說明 |
|---|---|
| REFCLK± | 常見為一對 100 MHz 差動參考時脈，整條 Link 共用；部分架構兩端使用獨立參考時脈 |
| PERST# | 低電位有效的重置信號 |
| CLKREQ#／WAKE# | 時脈請求／喚醒；依介面與平台支援 |
| PRSNT#／SMBus | 插卡存在偵測／輔助管理；依介面支援 |
| 標準擴充卡供電 | +12 V、+3.3 V，以及依平台提供的 +3.3 Vaux |
| M.2 NVMe SSD 供電 | 主要為 +3.3 V |

x4、x16 不代表供電電壓。REFCLK 也不是高速資料的逐位元時脈，RX 仍需 CDR（時脈與資料恢復）。

## 3. GPU／Server 實際搬運什麼

| 路徑 | 常見資料 |
|---|---|
| 系統 RAM ↔ GPU | 模型權重、輸入張量、圖形資源、運算結果 |
| CPU ↔ GPU 控制路徑 | 工作提交、控制命令、狀態與完成通知 |
| GPU ↔ GPU | 中間張量、梯度、模型分割後所需資料 |
| RAM ↔ NVMe SSD | 儲存命令、檔案與資料庫內容 |
| RAM ↔ NIC | 網路封包資料、描述符與完成通知 |

大量搬移通常由 DMA 執行：CPU／驅動準備工作與記憶體位置，裝置負責搬移，不必由 CPU 逐位元組複製。

```text
CPU ── 記憶體通道 ── 系統 RAM
 │
 └── PCIe Root Complex
       ├── GPU
       ├── NVMe SSD
       ├── NIC
       └── PCIe Switch ── 更多裝置
```

必須分清三種頻寬：

- GPU 核心 ↔ 本地 GDDR／HBM：顯存頻寬，通常不經 PCIe。
- 系統 RAM ↔ GPU：在一般 PCIe GPU 平台上測的是 PCIe 搬移路徑。
- GPU ↔ GPU：可能走 NVLink、PCIe P2P，或經系統 RAM 中轉，須確認實際拓樸與軟體路徑。

CPU/GPU 都可能有快取階層。GPU 常見 L1 與共享 L2，L3 視架構而定；GDDR／HBM 本身不是 L3。PCIe 的 L0/L1/L2 連線或電源狀態與快取層級無關。

## 4. GPU Diag 常見測試

Diag 是 Diagnostics，意指診斷測試。

| 測試 | 重點 |
|---|---|
| 辨識與初始化 | GPU 數量、驅動、韌體與運算環境是否正常 |
| 運算正確性 | 執行已知運算並核對結果 |
| 顯存完整性 | 寫入讀回比對、檢查 ECC 等錯誤 |
| 顯存頻寬 | 本地記憶體效能 |
| PCIe | 速率、寬度、頻寬、延遲與錯誤 |
| NVLink／NVSwitch | 每條 Link 與 GPU 間路徑的連通性、錯誤及效能 |
| 壓力、功耗、散熱 | 長時間負載下是否穩定 |
| 整機驗證 | 重啟循環、燒機、實際 AI 工作負載 |

測試通過只代表已執行項目符合設定門檻，不代表所有硬體功能都被涵蓋。

## 5. 通路多，如何定位故障

1. 建立拓樸與實體對照：GPU UUID、PCIe BDF、插槽、Switch 埠、NVLink 編號。
2. 確認實際路徑與協商狀態：預期 x16 不代表目前真的以 x16 運作。
3. 分別測 RAM ↔ 各 GPU、GPU ↔ GPU，再測多裝置同時傳輸。
4. 比較測試前後的錯誤增量：PCIe AER／Replay、NVLink 各 Link 計數與 GPU／系統日誌。
5. 找共同故障段：只有一顆異常查其分支；多顆同時異常查共享 Switch、上行、供電等。
6. 依平台維修方式斷電交換已知良品，觀察問題跟著模組移動，還是留在插槽。
7. 縮小至特定 Link 後，再做逐 Lane 診斷與電氣量測。

常用起點：`nvidia-smi topo -m`、`nvidia-smi -q`、Linux `lspci -t`、`lspci -vv`。詳細功能依版本與權限而定。

接收端報錯只能先指向一段通路；對端 TX、走線、連接器與本端 RX 都可能是原因。降低速率後恢復正常提示裕量問題，但不是零件故障的直接證明。
