# ADグループ → OneLogin ロール 対応表

> **用途**: AWS Client VPN 移行（SAML認証）設定作業用
> ※ 会社名・システム名・SID はすべて匿名化しています。

---

## 背景・目的

Cisco VPN 廃止に伴い、AWS Client VPN へ移行する。
現行は **ADグループSID** で認可判定しているが、SAML認証（OneLogin）では
**SAMLアサーションの `memberOf` 属性に含まれるロール名** で判定する。

### 認可ルール設定時の注意

- AWS Client VPN コンソールの「アクセスグループID」に **OneLoginロール名を完全一致で入力**
- 大文字・小文字・ハイフンも完全一致（例: `vpnclient-main` ≠ `VpnClient-Main`）
- OneLogin 側でロール名を作成するときも同じ文字列を使うこと

---

## 対応表（全8グループ）

| # | ADグループ名（移行前） | AD SID（移行前） | OneLoginロール名（移行後） | 対象ユーザー種別 |
|---|---|---|---|---|
| 1 | VpnClient-OtherSystem-Admin | S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX-**9001** | VpnClient-OtherSystem-Admin | 社内管理者 |
| 2 | VpnClient-OtherSystem-User | S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX-**9002** | VpnClient-OtherSystem-User | 社内一般ユーザー |
| 3 | VpnClient-ExtVender-VendorA | S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX-**9003** | VpnClient-ExtVender-VendorA | 外部ベンダーA |
| 4 | VpnClient-ExtVender-VendorB | S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX-**9004** | VpnClient-ExtVender-VendorB | 外部ベンダーB |
| 5 | VpnClient-Main-CORP-Overseas | （⚠️ 要確認） | VpnClient-Main-CORP-Overseas | 海外拠点スタッフ |
| 6 | VpnClient-Main-CORP-User | （⚠️ 要確認） | VpnClient-Main-CORP-User | 国内一般ユーザー |
| 7 | VpnClient-Sub-SAPCloud-Admin | （⚠️ 要確認） | VpnClient-Sub-SAPCloud-Admin | SAP Cloud 管理者 |
| 8 | VpnClient-Sub-SystemC-Admin | （⚠️ 要確認） | VpnClient-Sub-SystemC-Admin | SystemC 管理者 |

> ⚠️ グループ5〜8 の SID は担当者確認後に追記。

---

## グループ別アクセス先

### 1. VpnClient-OtherSystem-Admin（社内管理者）

| アクセス先 | IPアドレス |
|---|---|
| SystemA-制御サーバ | 10.2.1.X/32 |
| SystemA-WorkPc | 10.2.1.X/32 |
| SystemB-本番DB | 10.211.10.X/32 |
| SystemB-開発DB | 10.211.11.X/32 |
| 社内DB-01 | 10.211.10.X/32 |
| 社内DB-02 | 10.211.10.X/32 |
| 本社ネットワーク | 10.1.0.0/16 |
| 大阪支社 | 10.50.0.0/16 |
| 名古屋支社 | 10.60.0.0/16 |
| 東北支店 | 10.110.0.0/16 |
| 広島支店 | 10.130.0.0/16 |
| 福岡支店 | 10.140.0.0/16 |
| Webトランジット環境 | 10.214.0.0/16 |
| AWS・DB関連 | 10.211.0.0/16 |
| SAP関連 | 10.200.1.0/24 |

### 2. VpnClient-OtherSystem-User（社内一般ユーザー）

| アクセス先 | IPアドレス |
|---|---|
| SystemA-本番 | 10.2.1.X/32 |
| SystemB 各サーバ | ⚠️ 要確認 |
| SystemC（本番・検証） | ⚠️ 要確認 |
| SAP関連 | 10.200.1.0/24 |

### 3. VpnClient-ExtVender-VendorA（外部ベンダーA）

| アクセス先 | IPアドレス |
|---|---|
| SAP関連のみ | 10.200.1.0/24 |

### 4. VpnClient-ExtVender-VendorB（外部ベンダーB）

| アクセス先 | IPアドレス |
|---|---|
| Webトランジット環境のみ | 10.214.0.0/16 |

### 5〜8. グループ（海外・SAP Cloud・SystemC）

> ⚠️ アクセス先は担当者確認後に追記。

---

## 共通アクセス（全グループ）

| リソース | IPアドレス | 備考 |
|---|---|---|
| AD-GUEST-01（ADSRV01） | 10.2.1.X/32 | 認証用ADサーバ |
| AD-GUEST-02（ADSRV02） | 10.2.1.X/32 | 認証用ADサーバ（冗長） |

---

## 関連ドキュメント

- OneLogin 設定手順 → `docs/procedures/step2_onelogin_setup.md`
- 認可ルール入力シート → `docs/design/03_authorization_rules.md`
- 確認事項リスト → 内部管理（非公開）
