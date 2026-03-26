# STEP 3：.ovpn ファイル配布手順書

---

## .ovpn ファイルとは

```
VPN クライアントの「接続設定書」
  - 接続先エンドポイントの DNS 名
  - 証明書・暗号化方式
  - プロトコル（UDP/TCP）・ポート番号
  - SAML 認証用の設定

※ユーザー個人の認証情報は含まない（OneLogin でログインする）
※エンドポイントごとに1ファイル（全ユーザー共通）
```

---

## 【日本側作業】.ovpn ファイルの取得

1. AWS コンソール → `VPC` → `Client VPN エンドポイント`
2. 対象エンドポイントを選択 → `クライアント設定をダウンロード`
3. `downloaded-client-config.ovpn` を取得

---

## ⚠️ 海外拠点向けは TCP 443 に変更して配布

デフォルトは UDP のため、海外（中国等）向けは書き換えが必要。

```diff
- proto udp
+ proto tcp

- remote cvpn-endpoint-XXXXXXXXXXXXXXXXX.prod.clientvpn.ap-northeast-1.amazonaws.com 1194
+ remote cvpn-endpoint-XXXXXXXXXXXXXXXXX.prod.clientvpn.ap-northeast-1.amazonaws.com 443
```

### 配布ファイルの管理

| ファイル名 | 用途 |
|---|---|
| `client-config.ovpn` | 国内ユーザー用（UDP 1194） |
| `client-config-overseas.ovpn` | 海外拠点用（TCP 443・GFW 対策） |

---

## 【ユーザー側作業】インストール・設定

### AWS VPN Client ダウンロード

| OS | 取得先 |
|---|---|
| Windows / macOS | AWS 公式（aws.amazon.com/vpn/client-vpn-download/） |

### .ovpn の読み込み手順

1. AWS VPN Client を起動
2. `File` → `Manage Profiles` → `Add Profile`
3. Display Name を入力・`.ovpn` ファイルを選択 → `Add Profile`

---

## 配布チェックリスト

| # | 確認項目 | 状態 |
|---|---|---|
| 1 | .ovpn を AWS コンソールからダウンロード済み | ⬜ |
| 2 | 海外拠点用に TCP 443 へ書き換え済み | ⬜ |
| 3 | 国内テストユーザーに配布済み | ⬜ |
| 4 | 海外拠点テストユーザーに配布済み | ⬜ |
| 5 | 全テストユーザーが AWS VPN Client インストール済み | ⬜ |
