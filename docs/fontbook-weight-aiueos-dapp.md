# app-fontbook（書体判定）— itonami（dapp）上の分散型設計

状態: 第 1 スライス実装（2026-10-20）。正本はこの文書。
追跡: Linear JUN-128（親 JUN-116）。

## 位置づけ

app-fontbook は Font Book —— 導入済み書体の一覧。capability は `fs/browse`
（ファイル探索のみ。パースは kotoba-lang/glyph、読み取りは provider）。
**フェイスが何であり、同一家族のフェイスがどう関係するか**を決める。

## このスライス: 書体判定コア

| 層 | 実体 | 状態 |
|---|---|---|
| 判定コア guest | `aiueos/fontbook_weight.cljk` — style-regular（空白・canonical-regular 5 名は Regular、他は capitalize）/ weight-of（CSS 重み表: thin=100..black/heavy=900、**未知は -1 = 400 にしない**）/ weight-preference（**ファイル宣言 → style 名 → 不明**）/ face-id-ok（**[family, style] で鍵、パス不可**）/ command-admissible（open/quicklook/copy-path/export のみ、**install/remove は無い**） | **実装済み・実測** |
| 記述子 | `aiueos/fontbook-weight.edn` | **実装済み** |
| parity | `test/app_fontbook/weight_parity_test.cljk` — **実物の model.cljk**（mokuroku+amu+kotoba-sema クラスパス）と一致（normalize-style / derive-weight / face-id / commands） | **実装済み・実測** |

実測: `amu test aiueos/fontbook_weight.cljk` → **18/18**（jvm-kir / js / wasm）。
parity → 3 tests / 16 assertions / 0 failures。

## 正本の正確な規則（guest はこれに追従 —— 明文の設計意図つき）

- **style 正規化**: `#{regular book roman normal text}` はすべて Regular。
  「正規化しなければ 1 つの family が 3 つの Regular 行に分かれ、3 つの
  フェイスに見える」。
- **重みは数値**（CSS numeric weights）。style 文字列ソートは「Black が Bold
  が Light の前に来て重み順に読めるが 2 箇所で逆」。
- `derive-weight` は**ファイル宣言 → style 名**の順。どちらも無ければ **nil**
  —— 「未知の重みは regular ではない」（400 と推定しない）。
- `face-id` は **[family, normalized-style]**。「同じフェイスはシステムと
  ユーザーに二重に導入されうる。パスで鍵にすると区別のつかない 2 行になる」。
- コマンドは open / quicklook / copy-path / export のみ。「現在レンダリング中
  のフォントを外せる viewer は別のもっと危険なプログラム」。

## 未実装境界

- glyph パース（フェイス書式の解析）は kotoba-lang/glyph = 別 repo。
- 重み regex の最左マッチ詳細（"Semibold" 中の "Bold" 問題）は正本の
  代替順序に依存 —— derive-weight 実行 parity で実測済み（Semibold=600）。
- mon（DNA）化、aiueos session 載せ（fleet 共通）。
