# 磁學基本公式

> 全文件統一使用 LLC 變壓器匝比定義：`n = Np / Ns`。

## 簡化磁路圖

磁場強度：

```text
H = F / l = N × I / l
```

| 符號 | 意義 | 單位 |
|---|---|---|
| `H` | magnetic field strength | A/m |
| `F` | magnetomotive force | ampere-turn |
| `N` | 匝數 | turn |
| `I` | 電流 | A |
| `l` | 磁路長度 | m |

磁通密度與磁通量：

```text
B = μ × H
B = Φ / Ae
Φ = B × Ae
```

法拉第定律：

```text
VL = L × di/dt
VL = N × dΦ/dt
VL = N × Ae × dB/dt
```

磁通密度變化：

```text
△B = L × △I / (N × Ae)
△B = V × △t / (N × Ae)
```

直流偏磁與峰值磁通：

```text
Bdc = L × Idc / (N × Ae)
Bpk = L × Ipk / (N × Ae)
```

電感量：

```text
L = NΦ / I
L = N × B × Ae / I
L = N × μ × H × Ae / I
L = N × μ × (N × I / l) × Ae / I
L = N^2 × μ × Ae / l
```

實務提醒：

- `Ae` 是有效截面積，不是幾何外觀面積。
- gap 會主導有效磁導率，因此加氣隙後不能只用材料本身的 `μ`。
- `Bpk` 要低於磁芯材料在最高溫度下的飽和值，還要留 margin。
