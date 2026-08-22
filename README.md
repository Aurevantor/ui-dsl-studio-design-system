# apps/studio/design — Studio 自身のカンプ

Studio の外装（chrome）を **UIX で書いたデザインカンプ**。Studio で開いて、
Preview を見ながら配色・間隔・字組みを決めるための絵。

**Studio 自身は引き続き React + CSS**（`apps/studio/src/studio.css`）で、
ここから実物へ運ぶのは **Token の値だけ**（#152）。

## いちばん先に読むもの — **「カンプがあること」と「カンプが実物を決めていること」は別**

この 2 つを取り違えると、**絵は正しく見えるのに実物と違う**という壊れ方をする。
`#131` が実測で示した「VRT があることと VRT がそれを見ていることは別」と同じ罠で、
このワークスペースで最も起きやすい事故がこれ。だから**守らないものを先に並べる**。

### 守らないもの

| 守らないもの | なぜ |
|---|---|
| **ピクセル**（VRT を張らない） | カンプは**変えるためのもの**。配色を試すたびに golden を焼き直すと、基準画像が「意図した変更」で埋まって**検出力がゼロになる**（`golden-testing`） |
| **カンプと `studio.css` の一致** | 機械で縛るとカンプの用途（試す）が消える。一致は Token の橋が**一方向に**運ぶ（色 9 件 + 字組みの尺度 25 件・#158 / #204） |
| **カンプの配置と実物の配置** | カンプが決めるのは配色・間隔・字組みで、レイアウトの正本ではない |
| **`--splitter` / `--age`** | 前者は `apps/studio/src/panes.ts` の `SPLITTER_PX` と対（`containment.test.ts` が一致を縛る）、後者は打鍵ごとに変わる実行時の値 |
| **`text-transform`** | DTCG にも IR の `TypographyValue`（`packages/ir/src/value.ts`。5 つしか無い）にも無い。大文字にするのは**書き手の仕事**で、`<Eyebrow label="FILES" />` のように綴りで書く。**`studio.css` に残る手書きはこの 2 件だけ**（#204。`containment.test.ts` が理由つきで固定している） |
| **`font-variant-numeric`** | 同じく持てないが、**#204 で `studio.css` から消した** —— 5 件とも等幅の上にあり、実測で幅もピクセルも 1px 変わらなかった（docs/04 §8） |

**逆に、機械で縛ってあるものは 4 つだけ**（`apps/studio/src/design.test.tsx`）——
①画面と部品が実在する ②診断 0 件 ③**描いて** `RenderIssue` 0 件
④`<ForEach>` / `<If>` を書かない。加えて `omitted:` の宣言と、
「実物に無い State / 階調を発明しない」の記録が固定してある（後述）。

## 設計ループ

**新しいコマンドは要らない。** 2 つの端末で:

```bash
# 端末 1: Studio を立てる
make dev

# 端末 2: uix dev を立てて、このワークスペースを見張らせる
make push WS=apps/studio/design
```

`http://localhost:5173/` を開き、Files から `design` の下の `.uix` / `.tokens.json` を選ぶ。

**端末 2 が要る。** Studio の Editor で編集すると Preview は即座に描き直すが、
**ディスクには書き戻さない**（`apps/studio/src/push.ts` —「編集中のバッファ。保存はしない
（Studio は fs を持たない）」）。そして逆向きも繋がっていない ——
`apps/studio/vite.config.ts` の `readWorkspace` は読んだ実ファイルを
**意図的に `addWatchFile` に載せていない**（#110）ので、**エディタでディスクを編集して
ブラウザをリロードしても古いまま**（dev サーバを立て直すまで変わらない）。
`uix dev` の WebSocket が、その唯一の経路。

繋がっているかは**タイトルバーに出る**（`uix dev` の表示）。繋がっていなければ、
見ている絵はディスクの内容ではない。

### 見るときの注意 — Preview ペインの幅で頭打ちになる

`.specimen` は `max-width: 100%` なので、**Preview ペインの実効幅が preset の幅より狭いと
そこで頭打ちになる**。Mac preset（1280pt）で見るなら、ペインを広げるか窓を広げること。
実寸かどうかは寸法線（`data-testid="preview-size"`）が**実測**して出しているので、
そこに出ている数が preset の数と違えば頭打ちしている。

## 何が在るか

正本はディレクトリそのもの（`screens/` と `components/`）。ここに書くのは
**それぞれが何を決める絵なのか**。

| 画面 | 決めるもの |
|---|---|
| `Catalog` | 部品の Variant を並べる見本帳。**絵の完成度ではなく経路が主題**（Token を編集すると即座に描き直る） |
| `Titlebar` | 道具の名前・開いているファイル・接続状態・件数を並べる 1 行 |
| `Gauge` | コンパイル定規（対数目盛の刻みの色・高さ・ラベルの字組み） |
| `FilesPane` | 左ペインの木。3 件並んだときの縦の律動 |
| `EditorPane` | 行送り・字送り・ガター幅・字下げ・選択行の地色 |
| `PreviewPane` | 方眼の濃さ・デバイスの飾り・選択枠の二重線。**絶対配置の代替がここに集まる** |
| `Shell` | **構図と階調** —— どこに罫が入り、どの面が一段上がり、5 ペインがどの比で並ぶか |

| 部品 | 役 |
|---|---|
| `Eyebrow` | 「道具に印刷された文字」の声（見出し・単位・ラベル） |
| `Rule` | 罫。**per-side border が無いことの代替**（UIX の `borderWidth` は 4 辺に掛かる） |
| `Pane` | 枠と角丸を持つ「札」。見本帳で 2 つ並べるときの器（`Shell` は使わない ―― 実物の 5 ペインは地続きの 1 枚） |
| `Problem` | 診断 1 行。軸は色ではなく `severity` |
| `FilesItem` | Files の木の 1 行（`normal` / `hover` / `selected`） |
| `Toggle` | 押せる語（`normal` / `hover` / `focused` / `selected`） |
| `Splitter` | 仕切り 6pt（`normal` / `hover` / `focused`） |
| `Specimen` | 標本 1 枚。**地が Variant の軸**（Light / Dark を並べないと飾りの半分が絵に出ない） |

## 触るときの決まり

- **`omitted:` を宣言する。** screen の先頭コメントに
  `<!-- studio-design: 名前 / omitted: 省いたものの説明（無ければ「なし」） -->` を書く。
  `design.test.tsx` の `EXPECTED_OMISSIONS` が**理由ごと固定**していて、
  黙って増やしても減らしても落ちる。これが「**静かに省く**」を止める唯一の壁 ——
  無いとカンプは「描いていないことに気づけない絵」になる
- **実物に無い State を発明しない**（#153 / #156 の実測）。`studio.css` の対話の引き金は
  `:hover` / `:focus-visible` /「入り」の属性セレクタの 3 種だけで、
  **`:active` と `:disabled` は 0 本**。とくに `<State name="pressed">` は書くと害がある ——
  `state-css.ts` が `:active` を出すので「押している間だけ変わる」別物になり、
  しかも Inspector で強制表示すれば正しく見えるので**カンプを見ている限り気づけない**。
  「入り」は `selected` という独自名で表す
- **実物に無い階調を発明しない**（#157 の実測）。`--mat-raise` を実際に読んでいるのは
  `apps/studio/src/editor/theme.ts` の `.cm-tooltip` **1 か所だけ**で、しかも CodeMirror の
  shadow root の中に居る。つまり外装で面が上がるのは「浮くもの」だけ ——
  5 ペインはどれも同じ作業面に載っていて、分けているのは面ではなく**罫と仕切り**
- **インクを持ち込まない。** `editor/ink.ts` の 5 色は `MAT_HUE` からの導出値で、
  「手で 1 色だけ触ることが構造的にできない」ことが設計目的。`toColor` は srgb の hex しか
  受けないので、書き写すとその構造保証が壊れる
- **数値を書くのは、正本が別に在るときだけ。** `Specimen` の 375 × 812（`device-presets.ts`）、
  `Splitter` の 6pt（`panes.ts` の `SPLITTER_PX`）、`Shell` の 220 / 420 / 520 / 180
  （`panes.ts` の `DEFAULT_LAYOUT`）は**代表 1 例の写し**なので Token にしない。
  それ以外の寸法は Token を通す（`padding` / `spacing` / `radius` / `background` に生の数値を
  書くと `lint.hardcoded-value` が出る）

## Token の 3 階層

`tokens/primitive.tokens.json` → `semantic.tokens.json` → `component.tokens.json`
（docs/04 §1）。画面と部品が読むのは **semantic（`chrome.*`）と component** で、
primitive は直接参照しない。

**字組みだけ semantic の中でもう 1 段ある**（#204）:

```text
fontSize.eyebrow            （primitive・生の 10px）
  → chrome.fontSize.eyebrow （尺度。**`:root` に出るのはここ**）
    → chrome.type.eyebrow   （役。画面と部品はここを読む）
      → eyebrow.typography  （component）
```

**役（`chrome.type.*`）の `fontSize` / `lineHeight` / `letterSpacing` は必ず尺度を alias する。**
裸の数値を書くと ①尺度を通らないので `:root` に出ず、カンプにだけ値が生まれる
②行送りは 4 以下だと**倍率として畳まれる**（`lineHeight: 3` は 3pt ではなく 30pt）——
**どちらも診断 0 件で通る**ので、`tools/studio-design-tokens.test.ts` が構造として縛っている。

**行送りは px で持つ**（倍率ではない）。`toTypography` は倍率を受けると `fontSize` を掛けて
畳むので Token の真実はどのみち px で、px で書けば畳まれず、上の 4 以下の分岐にも入らない。
**px にすると継承の意味が変わる** —— 無単位は係数として、px は長さとして継承するので、
親の行送りを px にしたら**大きさの違う子が自分の行送りを持つ必要がある**（docs/04 §8）。

`chrome.*` に切ってあるのは、標準の Design System（`color.action.primary` など）と
**名前が 1 つも交わらない**ようにするため —— 客も依存も違う 3 つ目の Token 集合である
ことの表明（#152）。値は `studio.css` の `:root` から写したものだが、
**写しであることは機械で縛らない**（上の「守らないもの」）。
