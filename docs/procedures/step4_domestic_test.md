# STEP 4：国内接続テスト手順書

---

## なぜ国内テストを先に実施するか

```
海外で問題が発生したとき、原因を切り分けるため:

  国内 OK → 海外で問題 = 現地の通信環境（GFW 等）の問題
  国内 NG → AWS / OneLogin の設定問題

→ 国内で設定の正しさを確認してから海外テストへ
```

---

## 事前確認チェックリスト

- [ ] Client VPN エンドポイントが起動中
- [ ] 認可ルールが設定済み（`03_authorization_rules.md` 参照）
- [ ] OneLogin にロール作成済み・ユーザー割り当て済み
- [ ] SAML の memberOf 属性マッピング設定済み
- [ ] AWS VPN Client インストール済み・.ovpn プロファイル追加済み

---

## STEP 4-1：VPN 接続テスト

1. AWS VPN Client を起動 → プロファイルを選択 → `Connect`
2. ブラウザが開き OneLogin ログイン画面が表示される
3. ID・パスワード入力 → MFA 認証完了
4. `Connected` と表示されることを確認

| 確認項目 | 期待結果 | 合否 |
|---|---|---|
| OneLogin ログイン画面が表示 | ブラウザが自動で開く | ⬜ |
| MFA 認証が動作する | MFA コードの入力を求められる | ⬜ |
| `Connected` と表示 | VPN Client が Connected になる | ⬜ |
| クライアント IP が割り当てられる | IP アドレスが表示される | ⬜ |

---

## STEP 4-2：ADサーバへの疎通確認（全グループ共通）

```powershell
ping 10.2.1.X    # AD-GUEST-01（ADSRV01）
ping 10.2.1.X    # AD-GUEST-02（ADSRV02）
```

| 確認項目 | 期待結果 | 合否 |
|---|---|---|
| AD-GUEST-01 への ping | 応答あり | ⬜ |
| AD-GUEST-02 への ping | 応答あり | ⬜ |

---

## STEP 4-3：グループ別アクセス確認

```powershell
# ✅ 許可範囲内（応答あるべき）
ping 10.2.1.X      # SystemA-本番
ping 10.200.1.1    # SAP関連（代表IP）

# ❌ 許可範囲外（タイムアウトになるべき）
ping 10.214.0.1    # VendorB のみ許可
ping 10.211.10.X   # Admin グループのみ許可
```

| 確認項目 | 期待結果 | 合否 |
|---|---|---|
| 許可範囲内 IP への ping | **応答あり** | ⬜ |
| 許可範囲外 IP への ping | **タイムアウト** | ⬜ |

> ⚠️ 許可範囲外が「応答あり」の場合 → 認可ルールが広すぎる。設定を見直す。

---

## STEP 4-4：CloudWatch Logs 確認

1. AWS コンソール → `CloudWatch` → `/aws/clientvpn/poc`
2. 最新ログストリームに以下が記録されていることを確認：

```
connection-established
client-ip: 172.16.X.X
common-name: <ユーザー名>
```

| 確認項目 | 期待結果 | 合否 |
|---|---|---|
| 接続ログが出力されている | connection-established が記録 | ⬜ |
| ユーザー名が記録されている | OneLogin のユーザー名が表示 | ⬜ |

---

## STEP 4-5：切断・再接続確認

1. `Disconnect` → `Disconnected` 表示を確認
2. 許可範囲内 IP への ping がタイムアウトになることを確認（VPN 切断済み）
3. 再度 `Connect` → `Connected` になることを確認

---

## 問題発生時の切り分けフロー

```
① OneLogin 画面が表示されない
   → .ovpn の proto / port 設定を確認

② Connected になるが ping が通らない
   → 認可ルール・グループ名スペルを確認
   → CloudWatch Logs でエラーを確認

③ 許可範囲外にもアクセスできる
   → 認可ルールの CIDR が広すぎる → 見直し

④ MFA が届かない
   → OneLogin の MFA 設定を確認
```
