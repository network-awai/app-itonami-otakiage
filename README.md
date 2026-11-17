# app-itonami-otakiage — Otakiage

お焚き上げ — 品物を登録し、再利用の依頼・引き渡し・期限切れ・供養の依頼と供養までを 1 つの状態機械で扱い、証明書とお祭りの予定を記録します。

Tauri **3.0.0-alpha.6** の独立 app（`tauri-runtime-wry`、OS の WebView）と、同じファイルの static Web ビルドです。記録はすべてこの端末の中（localStorage、key `itonami.otakiage.v1`）に保存し、外部へは通信しません。外部操作の宣言もありません（`dist/descriptor.json` の `operations: []`）。保存に失敗したときは画面に表示します。

## できること

| 操作 | 内容 |
| --- | --- |
| 証明書を発行 (issueCertificate) | — |
| 証明書を記録 (anchorCertificate) | — |
| お祭りを予定 (scheduleMatsuri) | — |
| 記録の数 (coverage) | — |
| 相談を記録 (agentChat) | — |
| 品物を登録 (submitItem) | — |
| 品物を見る (getItem) | — |
| 品物の一覧 (listItems) | — |
| 再利用を依頼 (requestReuse) | — |
| 引き渡す (handover) | — |
| 期限切れにする (expire) | — |
| 供養を依頼 (requestRitual) | — |
| 供養する (ritualize) | — |

記録（collection）: `cloud.itonami.otakiage.agentChat`、`cloud.itonami.otakiage.certificate`、`cloud.itonami.otakiage.item`、`cloud.itonami.otakiage.matsuri`、`cloud.itonami.otakiage.reuseRequest`、`cloud.itonami.otakiage.ritualRequest`

## 補足

- 品物ごとに 1 つの状態機械です: 登録 → 再利用の募集（reuse_open）→ 引き渡し / 期限切れ、または供養の依頼（ritual_pending）→ 供養（ritualized）と証明書。
- `anchorCertificate` は、別の場所で行ったチェーンへの記録の tx hash を受け取って残すだけで、チェーンには書きません。
- `agentChat` は相談のメッセージを記録するだけで、推論（LLM）は呼びません。
- domain の詳しい説明は [`docs/domain.md`](docs/domain.md)（旧 `kotoba/README.md`）にあります。

## 構成

| パス | 役割 |
| --- | --- |
| `src/domain/` | domain logic（TypeScript）。ビルド時に Node の type stripping で JS にします（bundler・依存なし） |
| `src/local-store.js` | この端末内の record store。domain が使う `read` / `write` を localStorage で実装 |
| `src/app-def.js` | 画面の操作と domain 関数・collection の対応 |
| `src/ui/` | 依存なしの UI。DOM はテキストノードだけで組み、HTML 文字列を使いません（CSP `style-src 'self'`） |
| `vendor/` | kotoba-lang/kotoba の `kotoba-client-runtime.js` / consent CSS と `PINS.json`。`scripts/build.mjs` が digest のずれで失敗します |
| `src-tauri/` | Tauri v3 host。read-only command は `awai_host_info` の 1 つだけ。filesystem・network・shell・installer 権限なし |


## コマンド

```sh
npm ci
npm test              # build してから domain と local-only のテスト（local-only.test.mjs, otakiage.test.ts）
npm run build         # dist/ と SHA-256 の release.json
npm run build:desktop # native desktop 実行ファイル（bundle・署名なし）
```

## 状態と制限（AWAI-3079, 2026-11-17）

- 旧構成（Cloudflare Worker の XRPC dispatcher / adapter、`wrangler.jsonc`、`@etzhayyim/sdk`）は削除しました。履歴は git にあります。
- 検証済み: `npm test`、`npm run build`、`cargo check`（Tauri 3.0.0-alpha.6）。
- 未検証: native window の起動、`tauri build`、Android、署名、配布。アイコンは app-soshiki の仮アイコンです。
