# 基礎電感、Ripple Ratio 與 Duty 公式

> 全文件統一使用 LLC 變壓器匝比定義：`n = Np / Ns`。

## 電感伏秒平衡

穩態時，一個 switching 週期內電感電流的淨變化量為零：

```text
△Ion = △Ioff
Von × ton = Voff × toff
```

因此：

```text
△I = Von × ton / L = Voff × toff / L
```

工作週期：

```text
D = ton / (ton + toff)
T = ton + toff
fs = 1 / T
D = Voff / (Von + Voff)
```

常見觀念：

- 所有拓樸通常在 `Vin_min` 時需要最大 duty `Dmax`。
- 所有拓樸通常在 `Vin_max` 時 duty 最小 `Dmin`。
- 穩態伏秒平衡不成立時，電感或變壓器磁芯可能偏磁，最後導致飽和。

## 電感電流與 ripple ratio

電感電流可分成直流分量與交流 ripple：

```text
Ipk = Idc + △I / 2
r = △I / Idc
Ipk = Idc × (1 + r / 2)
```

其中：

| 符號 | 意義 |
|---|---|
| `Idc` | 電感平均電流 |
| `△I` | 電感峰對峰 ripple current |
| `r` | ripple ratio |
| `Ipk` | 電感峰值電流 |

常見拓樸的 `Idc`：

```text
Buck:       Idc = Io
Boost:      Idc = Io / (1 - D)
Buck-Boost: Idc = Io / (1 - D)
```

電感 ripple：

```text
△I = VL × △t / L
△I = Von × D / (fs × L)
△I = Voff × (1 - D) / (fs × L)
```

ripple ratio：

```text
r = △I / Idc
r = Von × D / (fs × L × Idc)
r = Voff × (1 - D) / (fs × L × Idc)
```

實務上常用 `r ≈ 0.2 ~ 0.4` 作為初估。`r` 越大，所需電感值越小，但峰值電流、RMS 電流與 EMI 壓力會增加。

## 負載、頻率與電感體積

若 `r` 固定：

```text
L ∝ 1 / Idc
L ∝ 1 / fs
```

但是電感能量與電感體積通常更接近由儲能決定：

```text
W = 1/2 × L × Ipk^2
```

### 負載電流增加

若負載電流變成 2 倍，為了維持相同 `r`，電感量可變成 1/2：

```text
Idc' = 2 × Idc
L' = L / 2
Ipk' ≈ 2 × Ipk
W' = 1/2 × (L / 2) × (2Ipk)^2 = 2W
```

結論：

- 負載電流增加時，電感體積通常增加。
- 若 `r` 固定，負載電流與所需電感量成反比，但與電感能量需求成正相關。

### switching frequency 增加

若頻率變成 2 倍，且 `Idc`、`r` 固定：

```text
fs' = 2 × fs
L' = L / 2
Ipk' ≈ Ipk
W' = W / 2
```

結論：

- 頻率越高，所需電感量越小。
- 頻率越高，電感體積通常可降低。
- 但 switching loss、core loss、driver loss 與 EMI 壓力會增加。

## 電感選擇注意事項

### 電感容差

若計算得到的電感值剛好貼近 IC 限流條件，且電感規格為 `±10%`，實際低端值可能只有標稱值的 90%。為了保證輸出功率，設計時應考慮：

```text
L_selected_min >= L_required
L_selected × (1 - tolerance) >= L_required
```

例如 `±10%` 電感：

```text
L_selected >= L_required / 0.9
```

### 電流限制

Buck 可直接用：

```text
Ipk = Io × (1 + r / 2)
Ipk <= Iclim_min
r <= (Iclim_min / Io - 1) × 2
```

Boost / Buck-Boost 必須先換成電感平均電流：

```text
ILdc = Io / (1 - D)
Ipk = ILdc × (1 + r / 2)
Ipk <= Iclim_min
```

Buck 範例：

```text
Iclim_min = 5.3 A
Io = 5 A
r <= (5.3 / 5 - 1) × 2 = 0.12
```

## Buck / Boost / Buck-Boost duty 公式

### Buck

```text
Io = Idc
Iin = Idc × D
η = Po / Pin
η = (Vo × Io) / (Vin × Iin)
η = (Vo × Io) / (Vin × Io × D)
D = Vo / (η × Vin)
```

理想情況 `η = 1`：

```text
D = Vo / Vin
```

### Boost

```text
Idc = Io / (1 - D)
Iin = Idc
η = (Vo × Io) / (Vin × Iin)
η = (Vo × Io) / (Vin × Io / (1 - D))
D = (Vo - η × Vin) / Vo
D = 1 - η × Vin / Vo
```

理想情況 `η = 1`：

```text
D = 1 - Vin / Vo
```

### Buck-Boost

```text
Idc = Io / (1 - D)
Iin = Idc × D
η = (Vo × Io) / (Vin × Iin)
η = (Vo × Io) / (Vin × Io / (1 - D) × D)
D = Vo / (η × Vin + Vo)
```

理想情況 `η = 1`：

```text
D = Vo / (Vin + Vo)
```
