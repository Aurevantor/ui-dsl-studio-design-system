# ui-dsl-studio-design-system — Studio 自身のデザイン仕様（SSOT）

[`ui-dsl-studio`](https://github.com/Aurevantor/ui-dsl-studio) の Studio（Web の Design IDE）の
外装を **UIX で書いたデザイン仕様**。**Studio の UI の正本（SSOT）はここ**で、
実装（`apps/studio`）はこの仕様を読んで作る（ui-dsl-studio の docs/06 §2・#298）。

以前は「実装が正で、このカンプが追随する」唯一の例外だったが、#298 で実装の現状
（Canvas First の配置・Projects・Bottom View・Flow モード・Web preset）へ揃え、向きを逆にした。

## この仕様が決めるもの・決めないもの

| 決める | 決めない |
|---|---|
| **構図・配置**（どのペインがどこに、どの比で並ぶか） | 文言やデータ（代表例でよい） |
| **部品**（押せるもの・行・操作群の形と State） | ピクセル一致（VRT は張らない） |
| **Token**（色・間隔・角・字組み） | 振る舞い（入力・保存・ドラッグ・ズームの計算） |

## Studio の UI を変えるときの順番

1. **この repo を先に直す**（`screens/` / `components/` / `tokens/`）。Studio で開いて見て決める
2. この repo で commit → push する
3. `ui-dsl-studio` で実装を直す。色と字組みの尺度は `pnpm generate:studio-css` で
   `studio.css` の `:root` に運ぶ（印 `ui-dsl.studio-css` の付いた Token だけ）。
   構図・部品・それ以外の Token は、人と AI が仕様を読んで実装する

**実装が仕様とずれたことは、今は機械で見つけない**（ui-dsl-studio の docs/01「将来構想」）。
ずれに気づいたら、どちらに合わせるかを人が決める。黙って実装側に寄せない。

## 開き方

Studio（`ui-dsl-studio` の `make app`）の Projects で「Project を追加」からこのフォルダを登録する。
次回からは起動時に自動で開く。保存は App の殻で開いたときだけできる（⌘S）。

ブラウザで見るなら、`ui-dsl-studio` で 2 つ立てる:

```bash
pnpm --filter @ui-dsl/studio dev                                    # Studio
node packages/cli/dist/bin.js dev ../ui-dsl-studio-design-system    # このフォルダを配る
```

`http://localhost:5173/?uix-open=<このフォルダの絶対パス>` を開き、Projects から選ぶ。
**`?uix-open=` を付けないと、配られた内容は「開いていないワークスペース」として捨てられる。**

この仕様は macOS の窓を描くので、`uix.json` の `preview.defaultDevice` は `mac`。
Mac preset は 2056 × 1305pt で、Fit で全体、100% で細部を見る。

**繋がっていなくても絵は正しく出る**（#208 の実測）。Token を 1 件変えたのに絵が変わらないときは、
判断する前に**計算値で届いたことを確かめる**（`getComputedStyle`、色なら canvas の probe）。

## 何が在るか

| 画面 | 決めるもの |
|---|---|
| `Shell` | **構図の正本**。既定の状態（Bottom View Off）の窓全体 —— タイトルバー・定規・`Files 220 │ Preview │ Workbench 500`（Editor 3 : Inspector 2）・底の StatusBar |
| `ShellBottomOn` | Bottom View On の窓。底が仕切りと Diagnostics の Drawer（高さ 96）に替わる |
| `FlowMode` | Flow モードの窓。toolbar → `Screens とフロー設定 290 │ 図 │ Inspector 360`。ボタンは枠付き・字は sans |
| `Titlebar` | タイトルバーの 3 状態（安静・未保存・保存の失敗） |
| `Gauge` | コンパイル定規（見出しの列 148・対数目盛・越えてはいけない範囲） |
| `FilesPane` | Files ペインの 2 段（Projects の一覧 → Project の木） |
| `EditorPane` | Editor の行組と、Token ファイルのときの Text / Visual 切替・Token Visualizer |
| `PreviewPane` | 操作バーの 5 群（Device / Web の幅 / Theme / Zoom / Backdrop）・方眼・機種ごとの縁・Aside の面・標本 |
| `InspectorPane` | Inspector（選択の要約 → Workspace → State → Props / Layout / Style）と未選択のとき |
| `DiagnosticsPane` | Diagnostics の Drawer と、問題が無いとき |
| `Catalog` | 部品の Variant を並べる見本帳 |

| 部品 | 役 |
|---|---|
| `Eyebrow` | 「道具に印刷された文字」の声（見出し・単位・ラベル） |
| `Rule` | 罫（per-side border が無いことの代替） |
| `Splitter` | 仕切り 6pt（`normal` / `hover` / `focused`） |
| `Toggle` | 押せる語（`normal` / `hover` / `focused` / `selected`）。枠を持たない |
| `ToggleGroup` | 操作群。Toggle を一段上げた面に束ねる（Device / Theme / Zoom、Design / Flow など） |
| `FlowButton` | Flow モードの押せるもの。枠と角 5 を持つ（`normal` / `hover` / `selected`） |
| `FilesItem` | Files の木の 1 行（名前とパスの 2 行・保留の印） |
| `ProjectItem` | Projects の 1 行（名前と root の 2 行） |
| `StatusBar` | Bottom View Off のときの底の 1 行 |
| `InspectorRow` | Inspector の 1 行（見出しの列と値の列・解決の連鎖） |
| `TokenCard` | Token Visualizer のカード 1 枚 |
| `Problem` | 診断 1 行（軸は色ではなく `severity`） |
| `Pane` | 枠と角丸を持つ札（見本帳で使う。窓の中のペインは地続きなので使わない） |
| `Specimen` | iPhone の標本 1 枚（地が Variant の軸） |

## 書くときの決まり

- **`omitted:` を宣言する。** screen の先頭コメントに
  `<!-- studio-design: 名前 / omitted: 省いたもの（無ければ「なし」） -->` を書く。
  UIX で描けないもの（曲線・グラデーション・絶対配置・折り返し）や、Screen に固定して
  描いていない State を、**黙って省かない**
- **State の「入り」は Screen から固定して描ける。** Component の State input と同名の
  boolean 属性を渡す（例: `<Toggle label="Design" selected="true" />`）。同じ操作群では
  実装の既定に合う 1 つを選ぶ。`hover` / `focused` など操作で変わる State は Inspector
  から強制表示して見る
- **実物に無い State を発明しない。** とくに `pressed` は書かない —— `state-css.ts` が
  `:active` を出すので「押している間だけ変わる」別物になり、強制表示では正しく見えるので
  気づけない。持続する「入り」は `selected` という独自名で書く
- **注釈は絵に混ぜない。** 説明の文は `<Screen>` の外の `<Aside>` に置く。
  線引きは「その文を絵の隣から離したら意味が変わるか」（凡例・寸法・状態の名札は絵の側）
- **作業面は 1 枚。** 一段上がる（`matRaise`）のは浮くもの（補完・吹き出し）・操作群
  （`ToggleGroup`）・Token Visualizer のカード・Flow の入力だけ。ペインを分けるのは罫と仕切り
- **数値を書くのは、正本が実装側に在るときだけ。** `Shell` の 220 / 500 / 96（`panes.ts` の
  `DEFAULT_LAYOUT`）、`Splitter` の 6pt（`SPLITTER_PX`）、iPhone の 375 × 812
  （`device-presets.ts`）は代表の写しなので Token にしない。それ以外の寸法は Token を通す
  （`lint.hardcoded-value`）
- **インクを持ち込まない。** Editor の 5 色（`editor/ink.ts`）は `MAT_HUE` からの導出値で、
  書き写すと「1 色だけ触れない」構造保証が壊れる

## Token の 3 階層

`tokens/primitive.tokens.json` → `semantic.tokens.json` → `component.tokens.json`。
画面と部品が読むのは **semantic（`chrome.*`）と component** で、primitive は直接参照しない。

**字組みは semantic の中でもう 1 段ある**（#204）:

```text
fontSize.eyebrow            （primitive・生の 10px）
  → chrome.fontSize.eyebrow （尺度。`:root` に出るのはここ）
    → chrome.type.eyebrow   （役。画面と部品はここを読む）
      → eyebrow.typography  （component）
```

役（`chrome.type.*`）の `fontSize` / `lineHeight` / `letterSpacing` は必ず尺度を alias する。
行送りは px で持つ（倍率にしない）。

**`studio.css` の `:root` に運ばれるのは、`$extensions` に `ui-dsl.studio-css` の印が付いた
Token だけ**（色 9 + 字組みの尺度 25）。半透明（`veil.*`）は `:root` に出せない。
角丸・間隔・幅などは印を持たず、実装が仕様を読んで同じ値を書く。

## 検査

- **`uix lint`**（`ui-dsl-studio` で `node packages/cli/dist/bin.js lint ../ui-dsl-studio-design-system`）が
  error 0 件であること。この repo だけでは検査を回せない（`@ui-dsl/*` は publish されていない）
- `ui-dsl-studio` の CI が読むのは、この repo ではなく**検査用に固定した標本**
  （`tests/fixtures/studio-design/`）。この repo を直しても CI は何も言わない
- `studio.css` とのずれは `ui-dsl-studio` で `pnpm generate:studio-css --check`
