# STEP 2：OneLogin 設定手順書

> **前提**: OneLogin 管理者権限があること

---

## 概要

```
STEP 2-1：AWS IAM に SAML IdP を登録
STEP 2-2：OneLogin に AWS Client VPN アプリを作成
STEP 2-3：OneLogin にロールを作成（8グループ分）
STEP 2-4：ユーザーをロールに割り当て
STEP 2-5：SAML の memberOf 属性マッピングを設定（★最重要）
STEP 2-6：動作確認（SAMLアサーションの確認）
```

---

## STEP 2-1：AWS IAM に SAML IdP を登録

**【OneLogin 側】メタデータ XML をダウンロード**

1. OneLogin 管理コンソール → `Applications` → `Applications`
2. AWS Client VPN アプリを選択
3. `More Actions` → `SAML Metadata` → XML ファイルをダウンロード

**【AWS 側】IAM Identity Provider を作成**

1. AWS コンソール → `IAM` → `ID プロバイダー` → `プロバイダーを追加`
2. 以下を入力：

| 項目 | 値 |
|---|---|
| プロバイダータイプ | SAML |
| プロバイダー名 | `OneLogin-ClientVPN` |
| メタデータドキュメント | ダウンロードした XML をアップロード |

3. 作成後、**プロバイダー ARN** をメモ（後の設定で使用）

---

## STEP 2-2：OneLogin に AWS Client VPN アプリを作成

1. OneLogin → `Applications` → `Add App`
2. `SAML Custom Connector` を選択
3. アプリ名：`AWS Client VPN`

**【Configuration タブ】**

| 項目 | 値 |
|---|---|
| ACS URL | `http://127.0.0.1:35001` |
| Audience URI（Entity ID） | `urn:amazon:webservices:clientvpn` |
| SAML Signature Algorithm | SHA-256 |
| SAML initiator | Service Provider |

---

## STEP 2-3：OneLogin にロールを作成

> ロール名は **ADグループ名と完全一致**（コピペ推奨）。

1. OneLogin → `Users` → `Roles` → `New Role`
2. `Name` にロール名を入力（下表からコピペ）
3. `Apps` タブ → `AWS Client VPN` を追加 → `Save`

### 作成するロール一覧

| # | ロール名（コピペ用） | 対象 | 作成済み |
|---|---|---|---|
| 1 | `VpnClient-OtherSystem-Admin` | 社内管理者 | ⬜ |
| 2 | `VpnClient-OtherSystem-User` | 社内一般ユーザー | ⬜ |
| 3 | `VpnClient-ExtVender-VendorA` | 外部ベンダーA | ⬜ |
| 4 | `VpnClient-ExtVender-VendorB` | 外部ベンダーB | ⬜ |
| 5 | `VpnClient-Main-CORP-Overseas` | 海外拠点スタッフ | ⬜ |
| 6 | `VpnClient-Main-CORP-User` | 国内一般ユーザー | ⬜ |
| 7 | `VpnClient-Sub-SAPCloud-Admin` | SAP Cloud 管理者 | ⬜ |
| 8 | `VpnClient-Sub-SystemC-Admin` | SystemC 管理者 | ⬜ |

---

## STEP 2-4：ユーザーをロールに割り当て

1. OneLogin → `Users` → `Roles` → 対象ロールを選択
2. `Users` タブ → `Add Users` → 対象ユーザーを追加 → `Save`

---

## STEP 2-5：SAML の memberOf 属性マッピングを設定（★最重要）

> **ここを設定しないと全員アクセス拒否になる。**

1. OneLogin → `Applications` → `AWS Client VPN`
2. `Parameters` タブ → `+` をクリック

| 設定項目 | 値 |
|---|---|
| Field name | `memberOf` |
| Value | `User Roles` |
| Include in SAML assertion | **ON**（必須） |
| Multi-value parameter | **ON**（複数ロール対応） |

3. `Save`

### よくあるミス

| ミス | 症状 | 対処 |
|---|---|---|
| `Include in SAML assertion` が OFF | 全アクセス拒否 | ON に変更 |
| Field name が `MemberOf`（大文字M） | 属性未認識 | `memberOf`（小文字m）に修正 |
| `Multi-value parameter` が OFF | 複数ロールで1つしか送られない | ON に変更 |

---

## STEP 2-6：動作確認（SAML Test）

1. OneLogin → `Applications` → `AWS Client VPN`
2. `More Actions` → `Test SAML`
3. 出力 XML に以下が含まれることを確認：

```xml
<Attribute Name="memberOf">
  <AttributeValue>VpnClient-OtherSystem-User</AttributeValue>
</Attribute>
```

### 確認チェックリスト

| 確認項目 | 期待値 | 結果 |
|---|---|---|
| memberOf 属性が存在する | XML に `<Attribute Name="memberOf">` | ⬜ |
| ロール名が完全一致 | ADグループ名と一致 | ⬜ |
| ACS URL | `http://127.0.0.1:35001` | ⬜ |
| Audience URI | `urn:amazon:webservices:clientvpn` | ⬜ |
