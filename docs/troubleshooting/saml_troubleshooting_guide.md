# SAML 認証 トラブルシューティングガイド

> **対象**: AWS Client VPN + OneLogin（SAML 2.0）構成
> **用途**: 接続エラー発生時の原因特定・対処

---

## よくあるエラー一覧（早見表）

| # | 症状 | 最多原因 | 対処セクション |
|---|---|---|---|
| 1 | VPN Client でブラウザが開かない | UDP ブロック（海外拠点） | → [A] |
| 2 | OneLogin ログイン後「接続失敗」 | SAML プロバイダー ARN 設定ミス | → [B] |
| 3 | Connected になるが全アクセス拒否 | memberOf 属性が飛んでいない | → [C] |
| 4 | 一部のサーバーにだけアクセスできない | 認可ルールの CIDR 設定漏れ | → [D] |
| 5 | 許可していないサーバーにもアクセスできる | 認可ルールが広すぎる | → [E] |
| 6 | MFA コードが届かない | OneLogin MFA 設定未完了 | → [F] |
| 7 | 再接続のたびに毎回認証が必要 | セッションタイムアウト設定 | → [G] |
| 8 | 特定ユーザーだけ接続できない | ロール割り当て漏れ | → [H] |

---

## 原因切り分けフロー

```
VPN に接続できない
    ↓
① ブラウザ（OneLogin 画面）が開くか？
    │
    ├─ 開かない → [A] UDP/TCP・ポート問題
    │
    └─ 開く
         ↓
      ② OneLogin 認証は通るか？
          │
          ├─ MFA が届かない → [F]
          ├─ ログイン失敗（パスワードエラー） → OneLogin 管理者に確認
          └─ 認証成功 → VPN Client が「Connected」になるか？
                   │
                   ├─ Connected にならない → [B] SAML 設定問題
                   └─ Connected になる
                            ↓
                         ③ ping / アクセスが通るか？
                             ├─ 全て通らない → [C] memberOf 問題
                             ├─ 一部通らない → [D] 認可ルール設定漏れ
                             └─ 通りすぎる  → [E] 認可ルールが広すぎる
```

---

## [A] ブラウザが開かない / OneLogin 画面が表示されない

### 原因
海外拠点（特に中国）で UDP 1194 が GFW によりブロックされている。

### 確認方法
```powershell
# VPN Client のログを確認
# Windows: %APPDATA%\Amazon\AWSVPNClient\logs
```

### 対処
`.ovpn` ファイルの以下の行を変更して再配布：

```diff
- proto udp
+ proto tcp

- remote cvpn-endpoint-XXXXXXXXXXXXXXXXX.prod.clientvpn.ap-northeast-1.amazonaws.com 1194
+ remote cvpn-endpoint-XXXXXXXXXXXXXXXXX.prod.clientvpn.ap-northeast-1.amazonaws.com 443
```

---

## [B] OneLogin 認証後に接続失敗

### 原因候補
1. AWS IAM の SAML プロバイダー ARN が正しくない
2. OneLogin アプリの ACS URL が間違っている
3. Audience URI が間違っている

### 確認方法

**AWS コンソールで確認:**
```
IAM → ID プロバイダー → OneLogin-ClientVPN
→ メタデータの有効期限・ARN を確認
```

**OneLogin で確認:**
```
Applications → AWS Client VPN → Configuration タブ
→ ACS URL: http://127.0.0.1:35001
→ Audience URI: urn:amazon:webservices:clientvpn
```

### 対処
| 項目 | 正しい値 |
|---|---|
| ACS URL | `http://127.0.0.1:35001` |
| Audience URI | `urn:amazon:webservices:clientvpn` |
| SAML Signature Algorithm | SHA-256 |

---

## [C] Connected になるが全アクセス拒否

### 原因
SAML アサーションに `memberOf` 属性が含まれていない。
AWS Client VPN はグループ情報がない場合、すべてのアクセスを拒否する。

### 確認方法

**OneLogin SAML Test で確認:**
```
Applications → AWS Client VPN → More Actions → Test SAML
→ 出力 XML に以下が含まれるか確認:

<Attribute Name="memberOf">
  <AttributeValue>VpnClient-OtherSystem-User</AttributeValue>
</Attribute>
```

**CloudWatch Logs で確認:**
```
/aws/clientvpn/poc → 最新ログ
→ "reason: no authorization rule matched" が出ていれば認可ルールの問題
```

### 対処
OneLogin → Applications → AWS Client VPN → **Parameters タブ**

| 設定項目 | 正しい値 |
|---|---|
| Field name | `memberOf`（小文字 m） |
| Value | `User Roles` |
| Include in SAML assertion | **ON** |
| Multi-value parameter | **ON** |

> ⚠️ `MemberOf`（大文字 M）は認識されない。必ず小文字。

---

## [D] 一部のサーバー・ネットワークにアクセスできない

### 原因
アクセスしたいサーバーの CIDR が認可ルールに設定されていない。

### 確認方法

```
AWS コンソール
→ VPC → Client VPN エンドポイント
→ 対象エンドポイント → 認可ルール タブ
→ 対象 CIDR が登録されているか確認
```

**CloudWatch Logs で確認:**
```
"reason: no authorization rule matched" → 認可ルールに CIDR がない
"reason: access denied" → ルールはあるがグループが一致しない
```

### 対処
不足している CIDR の認可ルールを追加：
```
認可ルールを追加
→ アクセス先 CIDR: [不足しているIPレンジ]
→ アクセスグループID: [対応するOneLoginロール名]
```

---

## [E] 許可していないサーバーにもアクセスできる

### 原因
認可ルールの CIDR が広すぎる（例: `0.0.0.0/0` や `/8` など）。

### 確認方法
```
AWS コンソール → 認可ルール タブ
→ 各ルールの「送信先ネットワーク」列を確認
→ 意図より広い CIDR が設定されていないか確認
```

### 対処
1. 広すぎるルールを削除
2. 必要な CIDR だけに絞ったルールを追加

> **最小権限の原則**: `/32`（ホスト単位）または業務上必要な最小 CIDR で設定する。

---

## [F] MFA コードが届かない

### 原因候補
1. OneLogin の MFA が未設定
2. MFA デバイス（スマホ等）が変わっている
3. 時刻同期がずれている（TOTP の場合）

### 対処

| 症状 | 対処 |
|---|---|
| MFA 設定自体がない | OneLogin ログイン後、Security → MFA で設定 |
| スマホを変えた | OneLogin 管理者に MFA リセットを依頼 |
| コードが毎回無効 | 端末の時刻を自動同期に設定 |

---

## [G] 再接続のたびに毎回認証が必要

### 原因
セッションタイムアウトが短すぎる（デフォルト: 8時間など）。

### 対処
```
AWS コンソール → Client VPN エンドポイント → 編集
→ セッションタイムアウト: 24時間（最大）に変更
```

> ※ OneLogin 側のセッション設定も確認すること。

---

## [H] 特定ユーザーだけ接続できない

### 確認フロー
```
1. そのユーザーが OneLogin にログインできるか確認
2. OneLogin で対象ユーザーのロール割り当てを確認
   → Users → 対象ユーザー → Roles タブ
   → 対応するロールが割り当てられているか

3. SAML Test をそのユーザーで実行
   → memberOf にロール名が含まれているか確認
```

### よくある原因
- OneLogin のロール割り当て漏れ
- ロール名のスペルミス（大文字小文字・スペース）

---

## CloudWatch Logs の読み方

### ロググループ
```
/aws/clientvpn/<エンドポイント名>
```

### 主要なログメッセージ

| メッセージ | 意味 | 次のアクション |
|---|---|---|
| `connection-attempt` | 接続試行 | 正常 |
| `connection-established` | 接続成功 | 正常 |
| `connection-terminated` | 切断 | 正常 |
| `no authorization rule matched` | 認可ルールに一致しない | 認可ルールを確認 |
| `access denied` | アクセス拒否 | グループ名・ルールを確認 |
| `authentication-failure` | 認証失敗 | SAML 設定・OneLogin を確認 |

### ログ確認コマンド（AWS CLI）
```bash
aws logs filter-log-events \
  --log-group-name /aws/clientvpn/poc \
  --filter-pattern "authentication-failure" \
  --region ap-northeast-1
```

---

## 問い合わせ時に確認すべき情報

サポートやエスカレーション時に以下を準備する：

```
1. エラーが発生した日時（タイムゾーン付き）
2. ユーザー名（OneLogin のメールアドレス）
3. クライアント IP（AWS VPN Client の「Connection Info」で確認）
4. エラーメッセージのスクリーンショット
5. CloudWatch Logs の該当ログ
6. OS・AWS VPN Client のバージョン
```
