# AWS Client VPN SAML 移行演習

Cisco VPN（ASA5512）から AWS Client VPN（SAML 認証）への移行を、実案件をもとに演習した設計・手順書セットです。

> **注記**: 実際の移行案件を素材にした学習演習です。会社名・システム名・IP アドレス等はすべて匿名化しています。

---

## 背景・目的

| 項目 | 内容 |
|---|---|
| 移行前 | Cisco VPN（ASA5512）2026年3月末 EOL |
| 移行後 | AWS Client VPN（SAML 2.0 認証） |
| 対象 | 海外拠点スタッフ約115名（海外拠点 約20名含む） |
| 認証方式 | OneLogin（IdP）+ SAML + MFA |
| リージョン | ap-northeast-1（東京） |

---

## 移行アーキテクチャ

```
【移行前】
ユーザー → Cisco VPN → AD 認証（SID ベース）→ 社内システム

【移行後】
ユーザー → AWS Client VPN → OneLogin（SAML 認証）
                                 ↓
                    memberOf 属性（ロール名）で認可判定
                                 ↓
                            社内システム
```

### 認証・認可の変化ポイント

```
変更前: AD グループ SID（S-1-5-21-XXXXXXXXXX）で認可
変更後: SAML アサーションの memberOf 属性（ロール名文字列）で認可

→ OneLogin ロール名 = AD グループ名と完全一致が必須
→ 大文字・小文字・ハイフン 1文字でも違うと認可失敗
```

---

## 作業フロー

```
STEP 1: AWS コンソール 認可ルール設定（グループ別 CIDR 制限）
  ↓
STEP 2: OneLogin ロール作成・SAML memberOf 属性マッピング設定
  ↓
STEP 3: .ovpn ファイル取得・ユーザーへ配布
  ↓
STEP 4: 国内テスト（AWS・OneLogin 設定の正常確認）
  ↓
STEP 5: 海外拠点テスト（TCP 443 / GFW 対策）
  ↓
STEP 6: 本番移行・Cisco VPN 撤去（Phase 2）
```

---

## 技術的ポイント

### 1. SAML 認証における認可設計

AWS Client VPN の認可ルールは、SAML アサーションの `memberOf` 属性で判定します。

```xml
<!-- OneLogin から届く SAML アサーションの例 -->
<Attribute Name="memberOf">
  <AttributeValue>VpnClient-OtherSystem-User</AttributeValue>
</Attribute>
```

- AWS コンソールの「アクセスグループID」にロール名を完全一致で入力
- OneLogin の Parameters タブで `memberOf` → `User Roles` のマッピングが必須

### 2. グループ別最小権限設計（CIDR 制限）

| グループ種別 | アクセス可能範囲 |
|---|---|
| 管理者 | 全拠点ネットワーク・全サーバ |
| 一般ユーザー | 業務システムのみ（管理 DB 等は除外） |
| 外部ベンダー A | SAP 関連のみ（10.200.1.0/24） |
| 外部ベンダー B | Web トランジットのみ（10.214.0.0/16） |

→ **最小権限の原則**：グループごとに必要最低限の CIDR のみ許可

### 3. 海外拠点（中国）向け GFW 対策

```diff
# 通常設定（国内向け）
- proto udp
- port 1194

# 海外拠点向け（GFW 対策）
+ proto tcp
+ port 443
```

GFW（グレートファイアウォール）による UDP ブロックを回避するため、`.ovpn` ファイルを国内用・海外用の2種類で管理。

### 4. 国内テストを先に実施する理由

```
海外で問題が発生したとき、原因を切り分けるため:

  国内 OK → 海外で問題 = 現地の通信環境（GFW 等）の問題
  国内 NG → AWS / OneLogin の設定問題

→ 国内で設定の正しさを確認してから海外テストに臨む
```

---

## ドキュメント構成

```
aws-client-vpn-saml-migration/
│
├── README.md                          ← このファイル
│
├── docs/design/                       設計資料
│   ├── 01_group_role_mapping.md       ADグループ → OneLoginロール 対応表
│   ├── 02_parameter_sheet.md          VPN エンドポイント パラメータシート
│   └── 03_authorization_rules.md      認可ルール 設定作業シート
│
├── docs/procedures/                   手順書
│   ├── step2_onelogin_setup.md        OneLogin ロール作成・SAML設定
│   ├── step3_ovpn_distribution.md     .ovpn ファイル配布
│   └── step4_domestic_test.md         国内接続テスト
│
└── docs/test/
    └── step5_overseas_test.md         海外拠点接続テスト
```

---

## 学んだこと・習得スキル

- **SAML 2.0 認証フロー**：SP / IdP / アサーションの役割と設定
- **AWS Client VPN**：エンドポイント・認可ルール・CloudWatch Logs 設計
- **OneLogin**：ロール作成・SAML アプリ設定・memberOf 属性マッピング
- **最小権限設計**：グループ別 CIDR 制限による細粒度アクセス制御
- **移行設計書の作成**：パラメータシート・手順書・テスト仕様書
- **現地通信制限への対応**：GFW を考慮したプロトコル・ポート選定

---

## 関連 AWS サービス

`Amazon VPC` `AWS Client VPN` `AWS Certificate Manager` `CloudWatch Logs` `IAM`

---

## 今後の予定

- [ ] 個人 AWS アカウントで PoC 環境を実際に構築してスクショを追加
- [ ] Terraform による IaC 化（`environments/dev/` に追加予定）
