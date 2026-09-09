# LLC 半橋諧振推導

> 全文件統一使用 LLC 變壓器匝比定義：`n = Np / Ns`。

### 電路直覺

LLC 半橋的功率級可看成：

```text
DC bus → half-bridge square wave → Lr-Cr-Lm resonant tank → transformer → rectifier → output capacitor/load
```

其中 `Q1` 為半橋上臂 MOSFET，`Q2` 為半橋下臂 MOSFET。

主要元件：

| 符號 | 意義 |
|---|---|
| `Vin` | DC bus 電壓 |
| `Lr` | series resonant inductor，可為外加電感加變壓器漏感 |
| `Cr` | resonant capacitor |
| `Lm` | transformer magnetizing inductance |
| `Np` | 一次側匝數 |
| `Ns` | 次級匝數 |
| `n` | 匝比，本文固定使用 `n = Np / Ns` |
| `fs` | switching frequency |
| `fr` | series resonant frequency |
| `fm` | magnetizing resonant frequency |
| `Ro` | output load resistance，`Ro = Vo / Io` |
| `Rac` | 次級整流負載折算到一次側的 AC 等效電阻 |

半橋輸出到 resonant tank 的 switching node 是方波：

```text
vHB(t) ≈ ±Vin / 2
```

LLC 透過改變 `fs` 控制增益：

- `fs ≈ fr`：接近諧振點，增益約為 1，效率通常好。
- `fs > fr`：tank 呈感性，增益下降，常用於高輸入電壓或輕載。
- `fm < fs < fr`：可能取得大於 1 的增益，常用於低輸入電壓滿載，但循環電流上升。
- `fs < fm`：容易進入容性區，ZVS 風險高，通常避免。

### 半橋方波的一次側基波

半橋方波幅度為 `±Vin/2`。對稱方波的 Fourier series 基波峰值為：

```text
V1,pk = 4 × (Vin/2) / π
V1,pk = 2Vin / π
```

基波 RMS：

```text
Vpri1,rms = V1,pk / sqrt(2)
Vpri1,rms = sqrt(2) × Vin / π
```

在 FHA（First Harmonic Approximation）中，只保留基波。這是 LLC 常用的手算近似，適合初估增益、頻率範圍與負載影響。

### series resonance 與 magnetizing resonance

`Lr` 與 `Cr` 的串聯諧振頻率：

```text
fr = 1 / (2π × sqrt(Lr × Cr))
```

若把 `Lr + Lm` 與 `Cr` 視為另一個低頻諧振點：

```text
fm = 1 / (2π × sqrt((Lr + Lm) × Cr))
```

因為 `Lm > 0`，所以：

```text
fm < fr
```

定義特性阻抗：

```text
Zr = sqrt(Lr / Cr)
```

定義電感比：

```text
Ln = Lm / Lr
```

`Ln` 的設計取捨：

- `Ln` 較大：magnetizing current 較小，輕載效率較好，但可用增益範圍較窄。
- `Ln` 較小：可取得較高升壓增益，但循環電流、導通損與變壓器 RMS 電流會增加。

### 次級整流負載折算成 AC 等效電阻

輸出負載：

```text
Ro = Vo / Io
```

對全波整流加輸出大電容的負載，在 FHA 下可等效為基波 AC 電阻。折算到一次側：

```text
Rac = (8 / π^2) × n^2 × Ro
```

其中 `n = Np / Ns`。

設計時常用滿載最低阻抗：

```text
Ro_min = Vo / Io_max
Rac_min = (8 / π^2) × n^2 × Ro_min
```

品質因數使用本文定義：

```text
Q = Zr / Rac
```

注意：有些資料定義為 `Q = Rac / Zr`，那麼增益公式會長得不同。比較公式前要先看 Q 的定義。

### LLC FHA 等效電路

FHA 等效電路可寫成：

```text
Vin1 → Zr_series → node → (Lm parallel Rac)
```

其中：

```text
Zr_series = jωLr + 1 / (jωCr)
Zm = jωLm
Zp = Zm || Rac
```

輸出相關電壓取在並聯支路 `Zp` 上，因此 tank voltage gain：

```text
M = | Zp / (Zr_series + Zp) |
```

### 阻抗正規化

定義：

```text
ωr = 1 / sqrt(Lr × Cr)
fn = fs / fr = ω / ωr
```

將阻抗除以 `Zr = sqrt(Lr / Cr)`：

```text
jωLr / Zr = jfn
1 / (jωCr × Zr) = -j / fn
```

所以 series branch：

```text
Zs / Zr = j(fn - 1/fn)
```

magnetizing branch：

```text
Zm / Zr = jωLm / Zr
Zm / Zr = j × fn × (Lm/Lr)
Zm / Zr = j × fn × Ln
```

負載 branch：

```text
Rac / Zr = 1 / Q
```

因此 normalized parallel branch：

```text
Zp_norm = (j × fn × Ln) || (1 / Q)
```

### LLC 增益推導結果

由：

```text
M = | Zp / (Zs + Zp) |
```

先令：

```text
a = fn × Ln
x = fn - 1/fn
```

則：

```text
Zs_norm = jx
Zm_norm = ja
Rac_norm = 1 / Q
```

並聯支路：

```text
Zp_norm = Zm_norm || Rac_norm
Zp_norm = (ja × 1/Q) / (ja + 1/Q)
Zp_norm = ja / (1 + jaQ)
```

代入增益式：

```text
M = | Zp_norm / (Zs_norm + Zp_norm) |
M = | [ja / (1 + jaQ)] / [jx + ja / (1 + jaQ)] |
```

分母通分：

```text
jx + ja / (1 + jaQ)
= [jx(1 + jaQ) + ja] / (1 + jaQ)
= [jx + jx × jaQ + ja] / (1 + jaQ)
= [jx - xaQ + ja] / (1 + jaQ)
= [-xaQ + j(x + a)] / (1 + jaQ)
```

因此：

```text
M = | ja / [-xaQ + j(x + a)] |
```

取 magnitude：

```text
M = a / sqrt((xaQ)^2 + (x + a)^2)
```

把 `a = fnLn`、`x = fn - 1/fn` 代回：

```text
M =
fnLn
/
sqrt(
  [Q × fnLn × (fn - 1/fn)]^2
  +
  [(fn - 1/fn) + fnLn]^2
)
```

上下同乘 `fn`，整理可得到常見 FHA 增益式：

```text
M =
Ln × fn^2
/
sqrt(
  [((Ln + 1) × fn^2 - 1)]^2
  +
  [fn × Q × Ln × (fn^2 - 1)]^2
)
```

寫成一行：

```text
M = (Ln × fn^2) / sqrt(((Ln + 1) × fn^2 - 1)^2 + (fn × Q × Ln × (fn^2 - 1))^2)
```

檢查點：

當 `fn = 1` 時：

```text
M = Ln / sqrt(Ln^2 + 0) = 1
```

也就是在 series resonance `fr` 附近，ideal FHA tank gain 約為 1。

### 半橋 LLC 的輸出電壓

半橋 LLC 在 FHA 近似下，常用 DC 轉換關係：

```text
Vo ≈ Vin × M / (2 × n)
```

因此所需增益：

```text
Mreq = 2 × n × Vo / Vin
```

設計流程通常是：

1. 決定輸入範圍 `Vin_min ~ Vin_max`。
2. 決定輸出 `Vo` 與最大負載 `Io_max`。
3. 選擇匝比 `n`，讓 nominal Vin 時 `fs` 靠近 `fr`。
4. 用 `Vin_min` 算最大所需增益 `Mreq_max`。
5. 用 `Vin_max` 算最小所需增益 `Mreq_min`。
6. 選 `Ln`、`Q`、`Lr`、`Cr`，讓頻率範圍能覆蓋所需增益。
7. 檢查 ZVS、RMS current、磁通密度、SR 耐壓與熱。

### 由目標 Q 反推 Lr 與 Cr

若已選定：

```text
fr
Q
Rac
```

由：

```text
Q = Zr / Rac
```

可得：

```text
Zr = Q × Rac
```

再由：

```text
Zr = sqrt(Lr / Cr)
fr = 1 / (2π × sqrt(Lr × Cr))
```

可推得：

```text
Lr = Zr / (2πfr)
Cr = 1 / (2πfr × Zr)
```

選定 `Ln` 後：

```text
Lm = Ln × Lr
```

### 磁化電流與 ZVS

`Lm` 上承受的近似方波電壓與 switching 頻率會決定 magnetizing current。粗略估算：

```text
△Im ≈ Vin / (4 × Lm × fs)
```

此式來自半橋每半週期施加約 `Vin/2`：

```text
△Im_half = (Vin/2) × (T/2) / Lm
△Im_half = Vin × T / (4Lm)
△Im_half = Vin / (4Lmfs)
```

ZVS 需要在 dead time 內，用 tank/magnetizing current 充放 MOSFET `Coss`：

```text
Eoss_total ≈ 1/2 × Coss_eq × Vin^2
```

直覺條件：

```text
available inductive energy > required Coss energy
```

常見設計檢查：

- 高 `Vin`、輕載時最容易失去 ZVS。
- `Lm` 太大時 magnetizing current 太小，ZVS margin 下降。
- `Lm` 太小時 ZVS 變容易，但循環電流與導通損上升。
- dead time 太短可能來不及充放 `Coss`。
- dead time 太長會增加 body diode conduction loss。

### 一個週期的工作狀態

以下整理自舊筆記，並用統一符號描述一次側 Q1/Q2 與二次側 SR 的狀態。S1/S2 代表二次側同步整流 MOSFET。

| 順序 | 階段 | 一次側開關 | 一次側電流行為 | 二次側 SR 狀態 | 輸出電容狀態 |
|---:|---|---|---|---|---|
| 1 | Q1 ZVS 建立死區 | Q1/Q2 皆 off | 正向 `Im` 釋放能量，抽取 Q1 `Coss` 電荷，使 Q1 `Vds` 降到接近 0 | S2 body diode 導通 | `Cout` 供應負載 |
| 2 | S2 turn-on delay | Q1 ZVS 導通 | 正向 resonant current 上升並超過 `Im`，開始傳能 | S2 body diode 續流 | `Cout` 開始轉為充電 |
| 3 | S2 power transfer | Q1 持續導通 | 正向半弦波電流達到正峰值 | S2 channel 導通 | `Cout` 充電並吸收 ripple |
| 4 | S2 turn-off delay | Q1 導通末段 | 正向 resonant current 下降，接近 `Im` | S2 body diode 續流 | `Cout` 供應負載 |
| 5 | Q1 commutation | Q1 關斷 | 正向 `Im` 對 Q1 `Coss` 充電、對 Q2 `Coss` 放電 | S2 截止，`Coss` 充電 | `Cout` 供應負載 |
| 6 | Q2 ZVS 建立死區 | Q1/Q2 皆 off | 負向 `Im` 釋放能量，抽取 Q2 `Coss` 電荷，使 Q2 `Vds` 降到接近 0 | S1 body diode 導通 | `Cout` 供應負載 |
| 7 | S1 turn-on delay | Q2 ZVS 導通 | 負向 resonant current 上升並超過 `Im`，開始傳能 | S1 body diode 續流 | `Cout` 開始轉為充電 |
| 8 | S1 power transfer | Q2 持續導通 | 負向半弦波電流達到負峰值 | S1 channel 導通 | `Cout` 充電 |
| 9 | S1 turn-off delay | Q2 導通末段 | 負向 resonant current 回升，接近 `Im` | S1 body diode 續流 | `Cout` 供應負載 |
| 10 | Q2 commutation | Q2 關斷 | 負向 `Im` 對 Q2 `Coss` 充電、對 Q1 `Coss` 放電 | S1 截止，`Coss` 充電 | `Cout` 供應負載 |

