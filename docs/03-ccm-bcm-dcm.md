# CCM / BCM / DCM

> 全文件統一使用 LLC 變壓器匝比定義：`n = Np / Ns`。

## 電感電流波形

定義：

```text
r = △I / IL
```

當電感電流谷值剛好碰到 0：

```text
IL - △I / 2 = 0
△I / IL = 2
r = 2
```

所以：

```text
r < 2  → CCM
r = 2  → BCM / CrCM
r > 2  → DCM
```

若在最大負載 `Io_max` 時設計 `r = 0.4`，進入 BCM 的負載約為：

```text
Io_BCM = Io_max × r / 2
```

範例：

```text
Io_max = 3 A
r = 0.4
Io_BCM = 3 × 0.4 / 2 = 0.6 A
```

### 各拓樸 DCM 邊界

Buck：

```text
IL = Io
r = Voff × (1 - D) / (L × Io × fs)
```

Boost：

```text
IL = Io / (1 - D)
r = Von × D / (L × IL × fs)
r = Von × D × (1 - D) / (L × Io × fs)
```

Buck-Boost：

```text
IL = Io / (1 - D)
r = Voff × (1 - D) / (L × IL × fs)
r = Voff × (1 - D)^2 / (L × Io × fs)
```

Boost 若以固定 `Vo`、`D` 改變 `Vin` 的條件推導，`D = 1/3` 時 ripple ratio 可能出現最大值。若把 `Von` 視為固定常數，`Von × D × (1-D)` 的最大點則是 `D = 1/2`。使用 DCM 邊界式時，要先確認推導條件。
