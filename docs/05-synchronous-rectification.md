# 輸出同步整流 SR 公式

> 全文件統一使用 LLC 變壓器匝比定義：`n = Np / Ns`。

### MOSFET 耐壓

Center-tap 次級：

```text
VDS_rating >= margin × (2 × Vo + Vspike)
```

Full-bridge 次級：

```text
VDS_rating >= margin × (Vo + Vspike)
```

其中：

| 符號 | 意義 |
|---|---|
| `Vspike` | 漏感、寄生電容與 layout ringing 造成的尖峰 |
| `margin` | 耐壓餘裕，常取 1.2 到 1.5 以上 |

實務上 `Vspike` 必須靠示波器量測，或先用漏感能量與 snubber 設計初估。

若要由一次側電壓粗估次級反射電壓，本文固定使用：

```text
n = Np / Ns
Vsec ≈ Vpri / n
Isec ≈ n × Ipri
```

因此若用一次側/次側匝數比直接乘上輸入電壓來估 SR 耐壓，套到本文的 `n = Np / Ns` 會把方向寫反。電壓從一次側換到次級應除以 `n`，不是乘以 `n`。SR MOSFET 耐壓實務上仍建議以 `2Vo + Vspike` 或 `Vo + Vspike` 這類二次側波形關係為主，再用示波器確認尖峰。

### 導通損

溫度修正後的 Rds(on)：

```text
Rds(on,Tj) = Rds(on,25°C) × kT
```

Center-tap 次級兩顆 SR 交替導通，總導通損近似：

```text
Pcond,total ≈ Io^2 × Rds(on,Tj)
```

若把每顆 SR 導通電流先近似成半週期方波，單顆 RMS 電流為：

```text
IFET,rms ≈ Io / sqrt(2)
```

LLC 在諧振點附近更接近半弦波。若二次側整流電流為：

```text
i(t) = Ipk × sin(θ),  0 <= θ <= π
```

全波整流後的輸出平均電流：

```text
Io = (1/π) × ∫[0 到 π] Ipk × sin(θ) dθ
Io = 2 × Ipk / π
Ipk = π × Io / 2
```

單顆 SR MOSFET 的 RMS 電流以整個 switching period 計算：

```text
IFET,rms^2 = (1 / 2π) × ∫[0 到 π] (Ipk × sin(θ))^2 dθ
IFET,rms^2 = Ipk^2 / 4
IFET,rms = Ipk / 2
IFET,rms = π × Io / 4
IFET,rms ≈ 0.785 × Io
```

單顆平均電流：

```text
IFET,avg = Io / 2
```

Full-bridge 次級任一時刻約兩顆 MOSFET 在電流路徑上，總導通損近似：

```text
Pcond,total ≈ 2 × Io^2 × Rds(on,Tj)
```

若以一次側 resonant current 估算二次側峰值電流：

```text
Isec_pk ≈ n × Ipri_pk
```

其中 `Ipri_pk` 不能只用 `Vin / Zr` 當作最終設計值，因為半橋基波、負載、`Lm` 分流、操作頻率與增益點都會改變 tank current。`Vin / Zr` 只能作為非常粗略的量級估算，最終要用 FHA 計算、simulation 或實測波形確認。

### dead time 與 body diode loss

若 dead time 期間由 body diode 導通：

```text
Pdiode,total ≈ 2 × Vf × Io × tdead × fs
```

此式是假設每週期有兩次 dead time。實際仍要依整流架構、電流波形與控制方式修正。

單顆 body diode dead-time 損耗也可用 duty 形式估算：

```text
Pdiode,FET ≈ Vf × IFET,avg_deadtime × (tdead / T)
```

若估整組 SR，需乘上每個週期實際進入 body diode 的次數與參與的 MOSFET 數量。

### gate drive loss

```text
Pgate,total = Nmos × Qg × Vdrv × fs
```

其中：

| 符號 | 意義 |
|---|---|
| `Nmos` | SR MOSFET 顆數 |
| `Qg` | gate charge，使用實際 `Vdrv` 下的 datasheet 值 |
| `Vdrv` | gate drive voltage |
| `fs` | switching frequency |

### SR timing

Turn-on：

```text
通常以 VDS 變負且超過判斷閾值後導通
```

Turn-off：

```text
SR 應在次級電流過零前關斷
```

常見錯誤：

- turn-on 太早：可能誤導通或造成異常反向電流。
- turn-on 太晚：body diode 導通時間變長，損耗增加。
- turn-off 太晚：次級電流過零後產生反向電流，輕載效率明顯變差。
- 忽略 VDS ringing，導致 SR controller 誤判。
