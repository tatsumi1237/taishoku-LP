# taishoku-LP

退職給付金サポートのランディングページ（静的サイト、ビルド不要）。

## 構成
- `index.html`: メインLP。`images/01.jpg`〜`11.jpg` を縦に並べた画像主体のページ。要所にLINE誘導リンク(`https://lin.ee/WKzGzve`)とGoogle広告のコンバージョン計測(`gtag_report_conversion`)を仕込んでいる
- フッターのプライバシーポリシー/利用規約/特商法表記/運営会社情報は `index.html` から Google Sites の外部ページ(`sites.google.com/cielis-corp.com/taishoku/...`)へリンクしている
- `policy.html/` `riyoukiyaku.html/` `tokusho.html/` `uneikaisha.html/`: それぞれ画像を1枚だけ含むディレクトリ（`.html`という名前だがディレクトリ）。**`index.html`からもどこからもリンクされていない**未使用/バックアップと思われる資材。削除・整理する場合は先に用途を確認すること

## 開発
- ビルドツール・パッケージマネージャなし。HTML/CSSを直接編集する
- ローカル確認: 適当な静的サーバーで良い（例: `python3 -m http.server`）
- 画像を追加・差し替えたら `index.html` の `<img>` の並び順とリンク(`<a>`)の対応がズレていないか確認する

## 規約
- インラインスタイルのみ使用（外部CSSファイルは無し）。既存スタイルに合わせる
- LINE誘導・広告コンバージョン計測のスクリプトタグ構成は変更しない限り触らない
