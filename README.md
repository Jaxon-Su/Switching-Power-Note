# 電源與硬體系統筆記

整理電源電子與硬體系統的學習筆記，涵蓋 Switching Power Supply 計算、半導體元件，以及 PCIe、NVLink 與 GPU Server 基礎。

## 電源電子

包含 Buck、Boost、Buck-Boost、DCM/BCM、磁學基本式、LLC 半橋諧振推導、輸出同步整流 SR、極點／零點與補償器設計，以及常用半導體元件與 Power 積分算法。

> 電源筆記統一使用 LLC 變壓器匝比定義：`n = Np / Ns`。

- [基礎電感、Ripple Ratio 與 Duty 公式](docs/01-basic-inductor-and-duty.md)
- [磁學基本公式](docs/02-magnetics.md)
- [CCM / BCM / DCM](docs/03-ccm-bcm-dcm.md)
- [LLC 半橋諧振推導](docs/04-llc-half-bridge.md)
- [輸出同步整流 SR 公式](docs/05-synchronous-rectification.md)
- [極點、零點與 Type I/II/III 補償器設計](docs/06-poles-zeros-compensation.md)
- [BJT、Zener、Diode、MOSFET 與 Power 積分算法](docs/07-semiconductor-and-power-integration.md)

## 高速互連與 GPU Server

- [PCIe 與 NVLink 必備知識](docs/08-pcie-nvlink-basics.md)：技術角色、傳輸原理、Lane 訊號與供電、GPU／Server 資料流、常見診斷測試與通路故障定位。
