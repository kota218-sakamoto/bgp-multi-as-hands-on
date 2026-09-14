# BGP Multi-AS Hands-on

Cisco Packet Tracerを使用して、4つのASで構成したeBGPネットワークを構築したハンズオンです。

複数のAS間でBGPネイバーを確立し、経路交換、ベストパス選択、リンク障害時の経路切り替えを確認しました。

## 構成

```text
PC1
 |
R1 (AS65001)
 | \
 |  \
R2  R3
AS65002  AS65003
 |        |
 |        |
 R4 (AS65004)
 |
PC2
```

R1からR4まで、以下の2つの経路を構成しています。

```text
R1 -> R2 -> R4
R1 -> R3 -> R4
```

## 使用技術

- Cisco Packet Tracer
- BGP
- eBGP
- AS_PATH
- BGP Best Path
- 冗長経路
- 障害試験

## AS構成

| Router | AS |
|---|---:|
| R1 | 65001 |
| R2 | 65002 |
| R3 | 65003 |
| R4 | 65004 |

## 実施内容

- 4台のルータを異なるASに配置
- eBGPネイバーの設定
- LANネットワークのBGP広報
- BGPテーブルの確認
- AS_PATHの確認
- 複数経路の学習
- ベストパスの確認
- PC1-PC2間の疎通確認
- R1-R2間リンクの障害試験
- R3経由への経路切り替え確認
- 障害中のPC間通信継続確認
- リンク復旧後のBGPネイバー再確立確認

## 通常時のBGP経路

R1では、192.168.40.0/24に対して2つのBGP経路を学習しました。

```text
*> 192.168.40.0/24   10.0.12.2   65002 65004 i
*                    10.0.13.2   65003 65004 i
```

通常時はR2経由がベストパスとして選択されました。

```text
R1 -> R2 -> R4
```

## 障害試験

R1-R2間リンクを停止しました。

障害発生後、R1ではR3経由の経路がベストパスとして選択されました。

```text
*> 192.168.40.0/24   10.0.13.2   65003 65004 i
```

ルーティングテーブルでもNext HopがR3へ変更されたことを確認しました。

```text
B 192.168.40.0/24 [20/0] via 10.0.13.2
```

障害中にPC1からPC2へpingを実行し、通信が継続できることを確認しました。

```text
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

## リンク復旧

R1-R2間リンク復旧後、R2経由のBGP経路が再度学習され、R1が2つの経路を保持していることを確認しました。

```text
*> 192.168.40.0/24   10.0.13.2   65003 65004 i
*                    10.0.12.2   65002 65004 i
```

今回は特定経路を優先するBGPポリシーを設定していないため、復旧後もR3経由がベストパスとして使用されました。

## 確認コマンド

```text
show ip bgp summary
show ip bgp
show ip route
show ip route bgp
ping
```

## ドキュメント

- [ネットワーク設計](docs/network-design.md)
- [試験結果](docs/test-results.md)

## Config

- [R1](configs/R1.txt)
- [R2](configs/R2.txt)
- [R3](configs/R3.txt)
- [R4](configs/R4.txt)

## リポジトリ構成

```text
bgp-multi-as-hands-on/
├── README.md
├── configs/
│   ├── R1.txt
│   ├── R2.txt
│   ├── R3.txt
│   └── R4.txt
├── docs/
│   ├── network-design.md
│   └── test-results.md
└── packet-tracer/
    └── bgp-multi-as.pkt
```
