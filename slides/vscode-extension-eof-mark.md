class: center, middle

# VSCode extension 作ってみた

2018/12/20 @msfukui

---

# Agenda

* 今日お話ししたいこと

* VSCode / VSCode extension って?

* なぜ VSCode extension?

* 作ってみた

* 感想

---

# 今日お話ししたいこと

* VSCode で TypeScript を書くの、とても楽しいです!

* 特に JavaScript に消耗した人には、とてもおすすめです!

---

# VSCode って?

## Visual Studio Code (VSCode)

Microsoft が開発している Electron ベースの軽量エディタ。

## VSCode extension

VSCode のプラグインの様なもの。

VSCode は extension でカスタマイズや機能拡張を行うことが前提になっている。

Extension Marketplace がエディタ内に組み込まれていて簡単にインストール可能。

---

# なぜ VSCode extension?

* JavaScript つらい。とにかくつらい。何もわからない。

* せめて型が欲しい。

* そうだ、 TypeScirpt に行こう。

* しかし TypeScript を Vim で書くのはだいぶつらそうだ。 ※注：個人の感想です。

* VSCode 使ってみる。TypeScript 書いてみる。 <span style="color: red;">やだ、なにこれ、楽しい..。</span> 

* extension は TypeScript で書くらしい。<span style="color: red;">練習代わりに書いてみよう!</span>

---

# 作ってみた

---

## 準備

何はともあれ公式サイトを見てみる。

https://code.visualstudio.com/docs/extensions/overview

* 全部書いてある。

* チュートリアルがあるのでやってみる。

  [Extension Generator](https://code.visualstudio.com/docs/extensions/yocode)
  : yo でテンプレートを生成して中身を見てみる。

  [Example - Hello World](https://code.visualstudio.com/docs/extensions/example-hello-world)
  : 世界にあいさつしてみる。

  [Exsample - Word Count](https://code.visualstudio.com/docs/extensions/example-word-count)
  : 単語の数を数えてみる。

* よし、<span style="color: red;">全部理解した!</span>

---

## 何を書くか

* 簡単で小さなものを、一通り最後まで終わらせてみたい。

* そういえば、秀丸とかTeraPadとかサクラエディタで表示される「これ」を VSCode でも表示したい。

  ![TeraPad](slides/terapad_eof.png)

* 個人的に、ファイルの最後は必ず改行としたい。表示させると一目でわかっていいなあ..。

```settings.json
"files.insertFinalNewline": true
```

---

## 書いてみる

エディタ上の表示は CSS っぽいもので装飾できるそうなので、

このあたりを参考に、

https://github.com/Microsoft/vscode-extension-samples/tree/master/decorator-sample

このあたりの API 仕様を眺めながら、コードを書く。

https://code.visualstudio.com/docs/extensionAPI/vscode-api

---

### 装飾

装飾したいことを書く。

CSS っぽく書ける。

でも CSS そのものではなく、使えるものに制約がある。JSON だし。

```
  const EofMarkDecorationType = vscode.window.createTextEditorDecorationType({
    after: {
      contentText: "[EOF]",
      color: "gray"
    }
  });
```

:after 疑似要素で、コンテンツの後ろに、文字列 \[EOF\] を、灰色で表示する装飾を定義してみる。

---

### 対象の指定

定義した装飾は、エディタ領域の指定したエリアに対して、適用することができる。

```
  function updateDecorations() {
    if (!activeEditor) {
      return;
    }
    const text = activeEditor.document.getText();
    const Eof: vscode.DecorationOptions[] = [];

    const startPos = activeEditor.document.positionAt(text.length - 1);
    const endPos = activeEditor.document.positionAt(text.length);
    const decoration = {
      range: new vscode.Range(startPos, endPos),
      hoverMessage: "End of file."
    };
    Eof.push(decoration);

    activeEditor.setDecorations(EofMarkDecorationType, Eof);
  }
```

先ほどの装飾を、エディタ領域にあるテキストの一番末尾に適用することで、

---

### 動作確認

表示できた!

なんかそれっぽい。いい感じ。

![VSCode](slides/vscode_eof.png)

* F5 を押すと、新しい VSCode が Debug モードで 起動する。

  手元で試しながら、コードを書ける。

---

## テスト

* extension のテスト用に雛形が用意されている。 [mocha](https://mochajs.org/) で書く。
  
  最初、勘違いしてしまったけど、ユニットテスト用ではないので注意。

  テストが起動すると、VSCode + 作った extension に対しての統合テストを実行する。

* API 足りてない気がする。

  特定の箇所で現在設定されている style の情報を取ってくる API が見当たらない。

  しょうがないので、起動時に有効なエディタ領域が空っぽなことだけテスト。ないよりはいい。

  ```
    test("[EOF] is displayed at the end of the file.", function() {
      let ae = vscode.window.activeTextEditor;
      if (ae) {
        assert.equal(ae.document.lineCount, 1);
      }
    }
  ```

---

## CI

### [Azure Pipeline](https://dev.azure.com/)(旧 Azure DevOps)

* CI/CD ツールっぽい位置付け..なのか..?

* チュートリアルまんまでとりあえず動く。

  Windows10, macOS, Linux の 3 つで実行してくれる。

  github.com との連携を設定すると、azure-pipelines.yml の内容で、

  コンテナで環境作って build & test して結果を通知してくれる。

* 雰囲気でやっていく。

* 何度探しても Azure Portal からどう行けばいいかわからない..。そういうところ MS っぽい。

---

## Publish

`npm -g vsce` で入れた vsce コマンドを使う。

```
$ vsce publish
Executing prepublish script 'npm run vscode:prepublish'...

> eof-mark@0.1.0 vscode:prepublish C:\Users\msfukui\Documents\projects\eof-mark
> npm run compile


> eof-mark@0.1.0 compile C:\Users\msfukui\Documents\projects\eof-mark
> tsc -p ./

Publishing msfukui.eof-mark@0.1.0...
Successfully published msfukui.eof-mark@0.1.0!
Your extension will live at ...
$
```

* package.json 内のバージョン番号で Extension Marketplace に Publish してくれる。

  しかし、バージョン番号の変更は、とても、よく、忘れる..。

---

## Extension Marketplace

アイコン作って、色決めて、package.json に書いておくと、こんな感じで見える。

<img src="slides/marketplace_eof.png" alt="Extension Marketplace" width="640x" />

---

### エディタ内の Extensions 

Side Bar の一番下の Extensions で直接 Extension Marketplace を表示できる。

![VSCode](slides/marketplace_in_editor_eof.png)

---

## 気になったところ(1)

* Windows で node.js つらくないですか?

  公式から LTS 版をダウンロードしてインストールするのしんどい。

  yo と generator-code, vsce を global に入れるので、グローバル汚染も気になる。

* yo で 生成されるテンプレートが微妙

  package.json に `publisher` の設定がなくて、そのままだとテスト、デバッグで落ちる。

  だめなやつだ..。

  とりあえず手で追記して回避。

  ```package.json
  ...
  "publisher": "msfukui",
  ...
  ```

---

## 気になったところ(2)

* VSCode での TypeScript の default task は git bash or WSL 使っていると失敗する

  [Typescript tasks does not work when ussing git-bash for windows - Issue #38178 Microsoft/vscode](https://github.com/Microsoft/vscode/issues/38178)

  これは VSCode の問題だけど、1年以上経っているので、bash はあまり使われていない感じ..なのか?

  とりあえず task の設定だけ PowerShell のものに手で書き換えて回避していたりする。

  ```.vscode/task.json
  ...
  "tasks": [
    {
      "type": "typescript",
      "tsconfig": "tsconfig.json",
      "options": {
        "shell": {
          "executable": "powershell.exe"
        }
      }
  ...
  ```

---

## 気になったところ(3)

* なぜか、Debug console の末尾にも \[EOF\] が表示されてしまう。

  Debug console もエディタ領域の一部とみなされているのか..。

  これはバグっぽい匂いがする。

  まだちゃんと調べていない。

---

# 感想

* TypeScript 楽しい。型があることの幸せを感じる。

* VSCode を Windows10 で使うのすごくいい。さくさくコード書ける。

* 作っている途中で気になった細かいバグっぽいものは報告して直せるとよさそう。

---

# おまけ その(1)

## extension vs extention

  * 一応 extension が正しいらしい。
  
    Google さんに聞くと、約 885,000,000 件 vs 約 13,900,000 件

  * でも extention = 拡張子 ということで、ちゃんと存在もするので紛らわしい。

  * VSCode については、extension が正しいみたい。

---

# おまけ その(2)

* このスライドは remark.js で作りました。

* シンプル。ここの雛形と説明で作り方はだいたいわかります。

  https://github.com/gnab/remark/wiki

* Markdown と HTML のファイルを分離できるのがうれしい。

* フォントは Google Fonts で。

* 参考にどうぞ。

  https://github.com/msfukui/vscode-extension-eof-mark-slides

---

# おまけ その(3)

こんな VSCode extension を使っています。

便利なもの、ぜひ教えていただけるとうれしいです!

## TypeScript

* TSLint

  チェッカー。すごい。がんがんエラー & 警告してくれる。

* Prettier

  フォーマッター。すごい。でも手だとよく実行を忘れます。

  gg =G で上から下までフォーマット。治安がよい。

あとはあんまり考えなくても動くのがうれしい。

---

## 共通

* Vim

  手が Vim の人向け。

* Path Intellisense

  ファイルパスを補完候補に上げてくれます。

* open-in-browser

  `Ctrl + K D` で編集中のファイルをブラウザで開いてくれます。

* Settings Sync

  複数のPC間の設定を gist で共有します。便利。

---

# 参考

* [デザインで使いやすい！おすすめGoogleフォント8選 | ジーニアスブログ](https://www.genius-web.co.jp/blog/cat-86/google-font.html)

* [markdown + remark.js + gh-pages でプレゼン資料を公開する - Qiita](https://qiita.com/harasou/items/1fa3cca6ac1ef175c876)
