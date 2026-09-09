# BJT、Zener、Diode、MOSFET 與 Power 積分算法

> 全文件統一使用 LLC 變壓器匝比定義：`n = Np / Ns`。

## Power 積分算法

電源設計裡最核心的損耗觀念是：

```text
p(t) = v(t) × i(t)
Pavg = (1 / T) × ∫[0 到 T] p(t) dt
Pavg = (1 / T) × ∫[0 到 T] v(t) × i(t) dt
```

其中：

| 符號 | 意義 |
|---|---|
| `p(t)` | 瞬時功率，單位 W |
| `v(t)` | 元件兩端瞬時電壓，單位 V |
| `i(t)` | 流過元件瞬時電流，單位 A |
| `T` | 計算週期，通常取一個 switching period |
| `Pavg` | 平均功率，也就是熱設計要處理的損耗 |

RMS 電流與 RMS 電壓：

```text
Irms = sqrt((1 / T) × ∫[0 到 T] i(t)^2 dt)
Vrms = sqrt((1 / T) × ∫[0 到 T] v(t)^2 dt)
```

若元件等效為電阻損耗：

```text
Pcond = Irms^2 × R
```

若元件等效為固定壓降，例如 diode：

```text
Pcond = Vf × Iavg
```

這兩個式子不要混用。MOSFET 導通損通常用 `Irms^2 × Rds(on)`，diode 導通損通常用 `Vf × Iavg`。

## 離散波形積分

實測示波器或 simulator 匯出的波形通常是一串取樣點：

```text
t0, t1, t2, ... tn
v0, v1, v2, ... vn
i0, i1, i2, ... in
```

先算每一點瞬時功率：

```text
pk = vk × ik
```

若取樣時間等距：

```text
Pavg ≈ (1 / N) × Σ pk
```

若取樣時間不等距，建議用梯形積分：

```text
E ≈ Σ [ (pk + pk+1) / 2 × (tk+1 - tk) ]
Pavg = E / T
```

switching loss 也用同一個觀念：

```text
Eon  = ∫turn-on  vDS(t) × iD(t) dt
Eoff = ∫turn-off vDS(t) × iD(t) dt
Psw = (Eon + Eoff) × fs
```

若一個週期內有多次事件：

```text
Psw,total = Σ Eevent × fs
```

實務注意：

- 示波器量 `vDS × iD` 時，voltage probe 與 current probe 的 delay 要校正。
- `Eon/Eoff` 積分範圍要固定，不能每次手動拉不同區間。
- diode reverse recovery、MOSFET body diode recovery、Coss 放電都可能藏在 switching loss 裡。
- RMS 電流要用完整週期計算，不要只算導通區間後直接拿來乘損耗。

## Diode

Diode 的主要作用是只允許單方向導通。電源裡常見在整流、freewheel、clamp、bootstrap、ORing 與保護路徑。

理想模型：

```text
順向導通：vD ≈ 0
反向截止：iD ≈ 0
```

實務模型：

```text
順向導通：vD ≈ Vf
反向截止：iD ≈ IR
```

導通損：

```text
Pcond ≈ Vf × IF,avg
```

若要更細，可用動態電阻模型：

```text
vD ≈ Vf0 + IF × rd
Pcond ≈ Vf0 × IF,avg + IF,rms^2 × rd
```

反向耐壓：

```text
VRRM >= margin × (Vreverse,max + Vspike)
```

常見檢查：

```text
IF,avg >= 實際平均電流
IF,rms >= 實際 RMS 電流
IFSM >= 啟動或短路浪湧電流
Trr 越大，reverse recovery loss 越嚴重
```

Schottky diode 沒有明顯 reverse recovery，但反向漏電較大，且高溫時漏電會上升。Fast recovery diode 可承受較高電壓，但 reverse recovery 會造成額外損耗與 EMI。

## Zener

Zener 常用在 voltage clamp、簡易 reference、OVP、gate 保護與吸收尖峰。它在反向崩潰區工作，電壓近似固定為 `Vz`。

基本工作式：

```text
Iz = (Vin - Vz) / Rseries
```

Zener 損耗：

```text
Pz = Vz × Iz
```

限流電阻損耗：

```text
PR = (Vin - Vz) × Iz
PR = Iz^2 × Rseries
```

設計檢查：

```text
Iz_min <= Iz <= Iz_max
Pz <= Pz_rating / margin
```

若 Zener 作為 MOSFET gate-source 保護：

```text
Vz < VGS_abs_max
Vz > 正常 Vdrv
```

例如 MOSFET `VGS_abs_max = ±20 V`，driver 正常為 `12 V`，常見會選 `15 V` 或 `18 V` 等級的 TVS/Zener，但仍要看 spike 能量與封裝功率。

常見錯誤：

- 只看 `Vz`，沒有算 `Pz`。
- 用小訊號 Zener 去吸收高能量尖峰。
- 忘記 Zener tolerance，例如 `5.1 V ±5%`。
- 把 Zener 當成精密 reference，但沒有檢查溫度係數與動態阻抗。

## BJT

BJT 是電流控制元件，常用在小訊號驅動、level shift、放電路徑、保護、簡易線性穩壓與輔助電路。電源主功率開關現在多用 MOSFET，但 BJT 在控制周邊仍很常見。

主動區近似：

```text
IC ≈ β × IB
```

飽和開關用法不能只靠 datasheet 的大訊號 `β`，通常要用 forced beta：

```text
IB >= IC / βforced
βforced 常取 5 到 20
```

基極電阻：

```text
RB = (Vdrive - VBE) / IB
```

常用近似：

```text
VBE ≈ 0.7 V
VCE(sat) ≈ 0.1 V 到 0.3 V
```

飽和導通損：

```text
Pcond ≈ VCE(sat) × IC,avg
```

若 BJT 工作在線性區：

```text
PBJT = VCE × IC
```

這時功率可能很大，必須做 SOA 與熱設計檢查。

常見錯誤：

- 用 `IC = β × IB` 設計飽和開關，導致高溫或低 β 元件時打不飽和。
- 忘記 BJT 有 storage time，關斷速度可能比想像慢。
- 基極沒有放電路徑，關斷拖很久。
- 用小訊號 BJT 扛電感性負載，沒有 clamp 反灌能量。

## MOSFET

MOSFET 是電壓控制元件，是 switching power supply 最常見的功率開關。選型時不要只看 `Rds(on)`，還要同時看耐壓、電流、Qg、Coss、SOA、封裝熱阻與實際驅動能力。

### 耐壓

```text
VDS_rating >= margin × (VDS,max + Vspike)
```

常見 margin：

```text
低壓同步整流：1.2 到 1.5
高壓 primary MOSFET：依 surge、ringing、安規與 derating 加大
```

### 導通損

```text
Pcond = ID,rms^2 × Rds(on,Tj)
Rds(on,Tj) = Rds(on,25°C) × kT
```

`kT` 需要從 datasheet 的 normalized Rds(on) vs temperature 曲線讀取，常見高溫下可能是 1.4 到 2 倍以上。

### Gate drive loss

```text
Pgate = Qg × Vdrv × fs
Pgate,total = Nmos × Qg × Vdrv × fs
```

其中 `Qg` 要用接近實際 `VGS` 的 datasheet 條件。若 driver 從輔助電源供電，這個損耗主要出現在 driver supply；若 driver IC 內部發熱，也要檢查 IC 溫升。

### Coss 損耗

硬切換時，MOSFET output capacitance 相關損耗可粗估：

```text
Eoss ≈ 1/2 × Coss × VDS^2
Poss ≈ Eoss × fs
```

更準確要用 datasheet 的 `Eoss`：

```text
Poss ≈ Eoss(VDS) × fs
```

LLC、phase-shift full bridge 等軟切換拓樸會利用諧振電流充放電 Coss 來達成 ZVS，但仍要確認 dead time 與 magnetizing current 是否足夠。

### switching loss

硬切換粗估：

```text
Psw ≈ 1/2 × VDS × ID × (tr + tf) × fs
```

這只是初估。實際上 Miller plateau、driver impedance、layout inductance、diode recovery、Coss、負載電流波形都會改變 `Eon/Eoff`。

較好的算法：

```text
Psw = (Eon + Eoff) × fs
Eon/Eoff 由 datasheet、simulation 或 vDS × iD 實測積分取得
```

### body diode 與 dead time

同步整流或半橋中，dead time 期間可能由 body diode 導通：

```text
Pbody ≈ Vf,body × Iavg,deadtime × Nevent × tdead × fs
```

其中：

| 符號 | 意義 |
|---|---|
| `Iavg,deadtime` | dead time 期間平均電流 |
| `Nevent` | 每個週期 body diode 導通事件數 |
| `tdead` | 單次 dead time |
| `fs` | switching frequency |

dead time 太短會 shoot-through，太長會增加 body diode loss 與 reverse recovery risk。

### MOSFET 選型表

| 項目 | 檢查重點 |
|---|---|
| `VDS` | 最大電壓加 spike 與 margin |
| `ID,rms` | 導通 RMS 電流與封裝能力 |
| `Rds(on)` | 要用高溫值，不只看 25°C |
| `Qg` | 影響 driver loss 與切換速度 |
| `Qgd` | 影響 Miller plateau 與 switching loss |
| `Coss/Eoss` | 影響硬切換損耗與 ZVS 條件 |
| `SOA` | 線性區、啟動、短路、熱插拔尤其重要 |
| `RθJA/RθJC` | 決定散熱與溫升 |

## 熱設計快速檢查

總損耗：

```text
Ploss,total = Pcond + Psw + Pgate + Pbody + 其他損耗
```

接面溫度粗估：

```text
Tj = Ta + Ploss × RθJA
```

若有散熱路徑分段：

```text
Tj = Ta + Ploss × (RθJC + RθCS + RθSA)
```

設計目標：

```text
Tj,max_design < Tj_abs_max
```

實務通常不會把元件長期設計在絕對最大接面溫度附近，因為壽命、Rds(on)、漏電與可靠度都會變差。

## 常見錯誤

- MOSFET 只比 `Rds(on)`，沒有看 `Qg`、`Eoss` 與封裝熱阻。
- diode 只看平均電流，沒有看 RMS、浪湧與 reverse recovery。
- Zener 只看崩潰電壓，沒有算吸收能量與功率。
- BJT 飽和開關沒有用 forced beta。
- switching loss 用 `1/2 × V × I × t × fs` 後就當最終值，沒有用波形積分或實測校正。
- RMS 電流用錯時間基準，導致導通損低估。
- datasheet 條件和實際條件不同，例如 `Rds(on)` 在 `VGS = 10 V`，但實際 gate drive 只有 `6 V`。
