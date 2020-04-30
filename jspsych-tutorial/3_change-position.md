# はじめに

本記事は，「jsPsych による心理学実験作成チュートリアル」の第 3 回の記事です。[第 2 回](https://qiita.com/snishym/items/a121a4d6a02c71b69f3e)では for 文を利用した刺激の系列提示の方法を紹介しました。今回は，刺激の提示位置の変更方法について解説します。詳細は右側の見出しリストをご覧ください。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/1e0511f8622282993ed1)をご一読ください。

# 刺激の提示位置を変更する

さて，さっそく素朴に刺激の提示位置を変更してみましょう。これまで提示する文字列を指定する際に`stimulus: 'L'` `stimulus: 'hello, world'`のように，提示したい文字列を指定してきました。そして，このために，`html-keyboard-response`というプラグインを使用してきました。実はこのプラグイン名にもあるように，`stimulus:`にhtml の構文を記述することができます[^1]。htmlの構文で提示刺激の位置を調整します。

以下のコードをコピペして実行してみてください。

[^1]: というよりも，単に文字列を入れただけのときでも html として処理されています。

```html:change_position.html
<!DOCTYPE html>
<html>
<head>
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-html-keyboard-response.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
</head>
<body></body>
<script>

  var trial_L = {
    type: 'html-keyboard-response',
    stimulus: '<div style="position: absolute; left: 40%; top: 50%">L</div>',
    trial_duration: 1000,
  }

  var trial_R = {
    type: 'html-keyboard-response',
    stimulus: '<div style="position: absolute; right: 40%; top: 50%">R</div>',
    trial_duration: 1000,
  }

  var trials = [trial_L, trial_R]
  trials = jsPsych.randomization.repeat(trials, 2);

  jsPsych.init({
    timeline: trials,
  });
</script>

</html>
```

いい感じに刺激を左右に提示することができたのではないでしょうか。この例では，`<div style="position: absolute; left: 40%; top: 50%">`文字`</div>`というように，スタイルを記述した`<div>`タグというもので提示したい文字列を挟んで位置を調整します。styleで指定している項目はそれぞれ，

- `position: absolute`: 絶対位置を用いる。ここでの絶対位置とは，ブラウザの上下左右の端を0とした位置のこと。
- `left: 40%`: 左から40%の位置（右端が100%）
- `top: 50%`: 上から50%の位置（下端が100％）

となります。`left`, `top`以外にも，`right`, `bottom`も使用することができます。上の例でも，Rを提示する際には`right: 40%`と指定しています。また，位置の単位には`%`以外にもピクセル`px`を用いることができます。基本的には`%`でいいでしょう。

`position`については他にも`static`, `relative`, `fixed`と指定することができます。詳しい説明は，google先生に任せることにしますが，文字列の位置を変更する場合に限っては`absolute`を使えばいいでしょう。

`<div>`タグについてもgoogle先生を参照してください。

## styleを別で作っておく

位置の変更は`<div>タグ`を使いつつ，`style=...`で指定すればよいことがわかりました。しかし，位置を指定したせいで`stimulus:`の部分がかなり読みにくくなってしまいました。例えば，どこが実際に提示したい刺激なのかパッとわかりにくいです。位置の指定のために，styleの部分に様々な設定項目が並んでいるのが読みにくさ一因のような気がします。なんとか位置に関する設定を`<div>`タグの外側に書くことができれば，もう少し読みやすくなりそうです。

HTMLでは，`<div style="...">`で指定していた`style`を，`<head>`タグの部分で，名前をつけて用意しておくことができます。具体的には，以下のようにします。

```html:change_position.html
<!DOCTYPE html>
<html>
<head>
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-html-keyboard-response.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
  <style>
    .text_left {
      position: absolute;
      left: 40%;
      top: 50%;
    }
    .text_right {
      position: absolute;
      right: 40%;
      top: 50%;
    }
  </style>
</head>
<body></body>
<script>

  var trial_L = {
    type: 'html-keyboard-response',
    stimulus: '<div class="text_left">L</div>',
    trial_duration: 1000,
  }

  var trial_R = {
    type: 'html-keyboard-response',
    stimulus: '<div class="text_right">R</div>',
    trial_duration: 1000,
  }

  var trials = [trial_L, trial_R]
  trials = jsPsych.randomization.repeat(trials, 2);

  jsPsych.init({
    timeline: trials,
  });
</script>

</html>
```

ポイントは2点です。

1. `<head>`の中に`<style>`というタグを新しく用意して，その中に`<div style=...>`で指定していた内容を書き写し，`text_left`や`text_right`という名前をつけています。
2. `stimulus`のもともと`<div style="設定項目">`だった部分を，`<div class="スタイル名">`として，その名前の`style`を読み込んでいます。

2で述べたように，事前に設定したスタイルを`class=スタイル名`で読み込むように変更することで，`stimulus`の部分がかなり見やすくなりました。また，`<style>`タグ内でスタイルを作成する際には，`;`で改行して，設定項目を縦に並べることができるため，何をどのように設定しているのかについても見やすくなりました。

## 提示位置の確認

位置を変更することができましたが，私の拙いHTMLの知識ではこれで正しいの不安になるところです。もともと`stimulus:`に文字列だけ指定していたときには，jsPsychのおかげで画面中央に刺激が提示されるようになっていました。もし，先程の例での位置の指定方法に問題がないのであれば，`stimulus:`に文字だけ指定したときと上下の位置に違いはないはずです。`trial_L`, `trial_R`のどちらか一方の`stimulus:`を文字列だけの指定に戻して，上下の位置が一致しているかどうかを確認してみましょう。コード例ではRの提示位置の指定をなくしています。

<details><summary>コード例</summary><div>

```html:check_position.html
<!DOCTYPE html>
<html>
<head>
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-html-keyboard-response.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
  <style>
    .text_left {
      position: absolute;
      left: 40%;
      top: 50%;
    }
    .text_right {
      position: absolute;
      right: 40%;
      top: 50%;
    }
  </style>
</head>
<body></body>
<script>

  var trial_L = {
    type: 'html-keyboard-response',
    stimulus: '<div class="text_left">L</div>',
    trial_duration: 1000,
  }

  var trial_R = {
    type: 'html-keyboard-response',
    stimulus: 'R',
    trial_duration: 1000,
  }

  var trials = [trial_L, trial_R]
  trials = jsPsych.randomization.repeat(trials, 2);

  jsPsych.init({
    timeline: trials,
  });
</script>

</html>
```

</div></details>

実際に確認してみると，私の方法で位置を調整したLが明らかにRよりも低い位置に提示されていることがわかります。すべての刺激に対して同じ指定方法`top: 50%`を用いれば，刺激間で上下の提示位置に差はないため，実験結果に影響はないと思われますが，jsPsychの「中央」からずれているのはすこし気持ちが悪いです。これを修正する方法を考えていきましょう。

## 何の位置が調整されているのか

`position`での位置指定がどのよう行われているのかを理解するために，簡単な実験をしてみましょう。2つの刺激の内，一方の刺激は元の通り`top:50%`で上から50%の位置に提示されるようにし，もう一方の刺激は`bottom: 50%`で下から50％の位置に提示されるようにしましょう。素朴にはどちらも同じ位置を指定しているように思われます。

<details><summary>コード例</summary><div>

```html:change_position.html
<!DOCTYPE html>
<html>
<head>
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-html-keyboard-response.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
  <style>
    .text_left {
      position: absolute;
      left: 40%;
      top: 50%;
    }
    .text_right {
      position: absolute;
      right: 40%;
      bottom: 50%;
    }
  </style>
</head>
<body></body>
<script>

  var trial_L = {
    type: 'html-keyboard-response',
    stimulus: '<div class="text_left">L</div>',
    trial_duration: 1000,
  }

  var trial_R = {
    type: 'html-keyboard-response',
    stimulus: 'R',
    trial_duration: 1000,
  }

  var trials = [trial_L, trial_R]
  trials = jsPsych.randomization.repeat(trials, 2);

  jsPsych.init({
    timeline: trials,
  });
</script>

</html>
```

</div></details>

しかし，試してみると，異なる位置に刺激が提示されたと思います。なぜこのようなことが起こるのでしょうか。実は，`top`や`bottom`で提示刺激（HTMLの要素）のどの部分を上から・下から50%の位置に持ってきているのかが異なります。

- `top`は提示刺激の**上端**が上から50%の位置に来るように調整します。
- `bottom` は提示刺激の**下端**が下から50%の位置に来るように調整します。

フォントのサイズ分だけ刺激の上端と下端の位置は異なるため，結果的に刺激は異なった位置に提示されることになります。

この実験のおかげで，`top`で指定しているのは刺激の上端の位置であること，そのため`top: 50%`と指定しても，刺激は，ウィンドウの縦方向のちょうど真ん中には提示されないということがわかりました。なんとなく，これがjsPsychの標準の処理と違った結果になる原因な気がしてきました。

そして，同じことが`left`，`right`で横方向の位置を指定したときにも当てはまります。`left`は刺激の左端，`right`は刺激の右端の位置を調整します。そのため，`left: 50%`と指定しても，刺激はウィンドウの横方向のちょうど真ん中には提示されませんし，`left: 50%`, `right: 50%`ではそれぞれ異なった位置に刺激が提示されます。

この問題を避けて，`top: 50%`・`bottom: 50%`（`left: 50%`・`right: 50%`）で刺激を縦（横）方向のちょうど真ん中に提示するには，刺激の**中心**が指定した位置に来るように調整すればよさそうです。

# 刺激の中心の位置を調整する

指定した位置に刺激の中心を持ってくる方法を考えていきましょう。例えば，これまでのように`top: 50%`, `left: 40%`とすると，**刺激の左上端**がブラウザの上端から50%, 左端から40%の位置に来ます。この位置に，刺激の中心を移動させたいのであれば，刺激の高さ（縦の長さ）の半分だけ上側に戻し，幅（横の長さ）の半分だけ左側に戻せばよいです。[このページ](https://www.granfairs.com/blog/staff/centering-by-css)の「6. transform」にある図がわかりやすいです。

これを実現するためには，styleに`transform: translateY(-50%) translateX(-50%);`を追加します。`translateY`は縦方向，`translateX`は横方向に対応しており，正の値を指定すると下（右）方向に移動し，負の値だと上（左）方向に移動します。

以下の例では，Lを画面左側に提示し，RはjsPsychの標準の処理で画面中央に提示するようにしています。実行して，LとRの縦方向の提示位置がどうなっているのかを確認してみてください。

```html:change_position_transform.html
<!DOCTYPE html>
<html>
<head>
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-html-keyboard-response.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
  <style>
    .text_left {
      position: absolute;
      left: 40%;
      top: 50%;
      transform: translateY(-50%) translateX(-50%);
      -webkit-transform: translateY(-50%) translateX(-50%);
    }
    .text_right {
      position: absolute;
      right: 40%;
      top: 50%;
      transform: translateY(-50%) translateX(50%);
      -webkit-transform: translateY(-50%) translateX(50%);
    }
  </style>
</head>
<body></body>
<script>

  var trial_L = {
    type: 'html-keyboard-response',
    stimulus: '<div class="text_left">L</div>',
    trial_duration: 1000,
  }

  var trial_R = {
    type: 'html-keyboard-response',
    stimulus: 'R',
    trial_duration: 1000,
  }

  var trials = [trial_L, trial_R]
  trials = jsPsych.randomization.repeat(trials, 2);

  jsPsych.init({
    timeline: trials,
  });
</script>

</html>
```

どうでしょうか。LとRが同じ高さに表示されていたのではないでしょうか。`text_left`の`left`を50%に設定すると，LとRが全く同じ位置に提示されます。つまり，今回作成した位置の設定と，jsPsychが標準で処理してくれる位置の設定が同じであるということがわかります。ぜひ確認してみてください。

上の例では使用していませんが，`text_right`を使うと，刺激の中心が画面の右側のちょうど40%の位置に提示されます。また，`text_right`の`transform`を`text_left`と見比べてみると`translateX`の値の正負が異なっていますが，`left:`，`right:`で指定した位置に刺激の中心を持ってくるという点では，どちらもこれで問題ありません。それぞれの`translateX`の正負を反転させたりしていろいろ試してみてください。

# おわりに

今回は，提示位置の変更方法を紹介しました。ウェブブラウザを使用するjsPsychでは，位置を変更するためにHTMLタグを触る必要があります。これが曲者で，個人的にはかなり苦戦しました。この記事でみなさんが自在に位置調整できるようになったことを願うばかりです。

次回は，LとRをランダムに左右に提示する方法を紹介します。これまでの例ではLとRの提示位置は固定されていて，Lは左，Rは右にしか提示されていませんでした。次回の内容で，ひとまず，サイモン課題が完成します。引き続きがんばりましょう！

[【第4回】キー反応の取得・刺激のランダマイズ](https://qiita.com/snishym/items/ccbf53e64e313584dd48)
