# AWS Client VPN 認可ルール 設定作業シート

> **用途**: AWS コンソール「認可ルール」タブへの入力作業用
> ※ 会社固有の情報はすべて匿名化しています。

---

## 入力手順（AWS コンソール）

```
VPC → Client VPN エンドポイント → 対象エンドポイントを選択
  → 「認可ルール」タブ → 「認可ルールを追加」
  → アクセス先 CIDR・アクセスグループID を入力
```

> ⚠️ アクセスグループID は OneLogin ロール名と **完全一致**（大文字小文字・ハイフン含む）

---

## ✅ 確定分（即時設定可能）

### グループ 1：VpnClient-OtherSystem-Admin（社内管理者）

| # | アクセス先 CIDR | アクセスグループID | 説明 | 設定済み |
|---|---|---|---|---|
| 1 | 10.1.0.0/16 | `VpnClient-OtherSystem-Admin` | 本社ネットワーク | ⬜ |
| 2 | 10.2.0.0/16 | `VpnClient-OtherSystem-Admin` | SystemA・社内系 | ⬜ |
| 3 | 10.50.0.0/16 | `VpnClient-OtherSystem-Admin` | 大阪支社 | ⬜ |
| 4 | 10.60.0.0/16 | `VpnClient-OtherSystem-Admin` | 名古屋支社 | ⬜ |
| 5 | 10.110.0.0/16 | `VpnClient-OtherSystem-Admin` | 東北支店 | ⬜ |
| 6 | 10.130.0.0/16 | `VpnClient-OtherSystem-Admin` | 広島支店 | ⬜ |
| 7 | 10.140.0.0/16 | `VpnClient-OtherSystem-Admin` | 福岡支店 | ⬜ |
| 8 | 10.200.1.0/24 | `VpnClient-OtherSystem-Admin` | SAP関連 | ⬜ |
| 9 | 10.211.0.0/16 | `VpnClient-OtherSystem-Admin` | AWS・DB関連 | ⬜ |
| 10 | 10.214.0.0/16 | `VpnClient-OtherSystem-Admin` | Webトランジット | ⬜ |

### グループ 2：VpnClient-OtherSystem-User（社内一般ユーザー）

| # | アクセス先 CIDR | アクセスグループID | 説明 | 設定済み |
|---|---|---|---|---|
| 1 | 10.2.1.X/32 | `VpnClient-OtherSystem-User` | SystemA-本番 | ⬜ |
| 2 | 10.200.1.0/24 | `VpnClient-OtherSystem-User` | SAP関連 | ⬜ |
| 3 | ⚠️ 要確認 | `VpnClient-OtherSystem-User` | SystemB 各サーバ | ⬜ |
| 4 | ⚠️ 要確認 | `VpnClient-OtherSystem-User` | SystemC | ⬜ |

### グループ 3：VpnClient-ExtVender-VendorA（外部ベンダーA）

| # | アクセス先 CIDR | アクセスグループID | 説明 | 設定済み |
|---|---|---|---|---|
| 1 | 10.200.1.0/24 | `VpnClient-ExtVender-VendorA` | SAP関連のみ | ⬜ |

### グループ 4：VpnClient-ExtVender-VendorB（外部ベンダーB）

| # | アクセス先 CIDR | アクセスグループID | 説明 | 設定済み |
|---|---|---|---|---|
| 1 | 10.214.0.0/16 | `VpnClient-ExtVender-VendorB` | Webトランジットのみ | ⬜ |

### 共通（全ユーザー）

| # | アクセス先 CIDR | アクセスグループID | 説明 | 設定済み |
|---|---|---|---|---|
| 1 | 10.2.1.X/32 | （全ユーザー許可） | AD-GUEST-01（ADSRV01） | ⬜ |
| 2 | 10.2.1.X/32 | （全ユーザー許可） | AD-GUEST-02（ADSRV02） | ⬜ |

---

## ⚠️ 未確定分（情報確認後に設定）

| グループ | アクセスグループID | 状態 |
|---|---|---|
| グループ 5 | `VpnClient-Main-CORP-Overseas` | ⚠️ アクセス先 IP 確認要 |
| グループ 6 | `VpnClient-Main-CORP-User` | ⚠️ アクセス先 IP 確認要 |
| グループ 7 | `VpnClient-Sub-SAPCloud-Admin` | ⚠️ アクセス先 IP 確認要 |
| グループ 8 | `VpnClient-Sub-SystemC-Admin` | ⚠️ アクセス先 IP 確認要 |

---

## 進捗サマリー

| グループ | 確定ルール数 | 状態 |
|---|---|---|
| OtherSystem-Admin | 10件 | ✅ 設定可能 |
| OtherSystem-User | 2件確定 / 2件未確定 | ⚠️ 一部設定可能 |
| ExtVender-VendorA | 1件 | ✅ 設定可能 |
| ExtVender-VendorB | 1件 | ✅ 設定可能 |
| 共通 | 2件 | ✅ 設定可能 |
| CORP-Overseas〜SystemC-Admin | 0件 | ⚠️ 確認後 |
| **合計** | **16件確定 / 未確定あり** | |
