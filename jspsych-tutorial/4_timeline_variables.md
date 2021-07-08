# 2021/03/15 補足

2021 年 2 月末くらいにリリースされた`v6.3.0`になって`jsPsych.timelineVariables()`の使い方が少し変わりました。関数`function(){}`内で使う際に，それまでのバージョンでは`true`という引数が必要でしたが，それが必要なくなりました。最新版をダウンロードしてこのチュートリアルに望んでいる場合は，その点に留意して以下の内容を読んでください。そのうちちゃんと修正しますが，ひとまずアナウンスにてご了承ください。その他重大な変更点については[全体のまとめ](https://qiita.com/snishym/items/1e0511f8622282993ed1)を参照してください。

**2021/07/08 追記**
引数に `true` を指定する必要がなくなったというだけで， `true` という引数が指定してあってもちゃんと動作します。なので，以下のコードはとりあえずそのままにしておきます。

# はじめに

本記事は，「jsPsych による心理学実験作成チュートリアル」の第 4 回の記事です。[第 3 回](https://qiita.com/snishym/items/bec56308c67922c3b3df)では刺激の提示位置を変更する方法を紹介しました。これまでの記事では`L`は左，`R`は右にだけ提示されるようなコードを書いていました。サイモン課題では位置と反応の競合を生じさせるために，`L`が右に提示されたり，`R`が左に提示される試行も作成する必要があります。今回はそれら 4 条件の試行を見やすく，編集しやすいコードで作成する方法を紹介します。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/1e0511f8622282993ed1)をご一読ください。

# 4 条件分の`trial`変数を作る

まずは，復習を兼ねて，これまでの記事で説明した範囲で 4 条件分の試行が提示される実験を作成してみましょう。

<details><summary>コード例</summary><div>

```html:no-timeline-variable.html
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

  var trial_L_left = {
    type: 'html-keyboard-response',
    stimulus: '<div class="text_left">L</div>',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  var trial_L_right = {
    type: 'html-keyboard-response',
    stimulus: '<div class="text_right">L</div>',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  var trial_R_left = {
    type: 'html-keyboard-response',
    stimulus: '<div class="text_left">R</div>',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  var trial_R_right = {
    type: 'html-keyboard-response',
    stimulus: '<div class="text_right">R</div>',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  var trials = [trial_L_left, trial_L_right, trial_R_left, trial_R_right];

  trials = jsPsych.randomization.repeat(trials, 2);

  jsPsych.init({
    timeline: trials,
    default_iti: 250
  });
</script>

</html>
```

</div></details>

基本的には，前回の記事で作成した`trial_L`, `trial_R`をコピペして，`<div>`タグの`class`を適宜変更すれば，目的のプログラムは書けると思います。合わせて，変数の名前も提示される文字とその位置が一目でわかるような名前に変更しておくと良いと思います。

これでサイモン課題は完成しました！あとはデータの保存です！

と言ってしまってもいいんですが，コピペをしながらコードを作成していて，こう思った方もいるのではないでしょうか。

> 各試行の`stimulus`以外は全く同じなんだから，変数は一つだけ作成して，提示する`stimulus`だけを変更することはできないだろうか。

これはもっともな指摘で，重要な視点です。というのも，上記のように 4 条件それぞれに対応する試行変数を作成した上で，もし反応として受け付けるキー（`choices`）や 1 試行の持続時間（`trial_duration`）を変更することになったら，かなり手間ではないでしょうか。その修正をしている際にミスをしてしまうかもしれません。今回はサイモン課題ということで 4 種類の試行しかありませんが，提示する刺激の種類が 100 個もあった場合，それぞれの刺激を指定した試行変数を作るのは現実的ではありません。あるいは，ランダムな計算課題を提示するという場合，これまでの方法で実現するのは不可能でしょう。

ということで，以降では，作成する試行変数を最少にして，必要な部分だけを各試行で変更する方法を紹介します。`jsPsych.timelineVariable()`という関数を使うのですが，これを使う際に連想配列というものを利用するので，まずはそれについて説明します。

# 連想配列

連想配列は，第 2 回で紹介した配列と同様に複数の要素を持つことのできるデータ構造ですが，単なる配列との違いは，保持している要素それぞれに，検索のためのキーをつけます。具体的には，`{キー1: 要素1, キー2: 要素2, ...}`と書きます（配列の場合は`[要素1, 要素2, ...]`）。そして，キーを指定することで，連想配列から要素を取り出すことができます。

```javascript
var dict = { letter: 'L', position: 'left' };

// output
// dict['letter'] -> 'L'
// dict['position'] -> 'left'
// dict.letter や dict.position で値を取り出すことも可能
```

辞書を想像してもらえばわかりやすいかもしれません。辞書ではある言葉を引いてその意味を調べられるように，連想配列でもある言葉からそれに対応する要素を取り出す事ができます。というか，連想配列は英語では dictionary と呼ばれているようです（なので，上のコード例でも変数名を`dict`としてみました）。

`jsPsych.timelineVariable()`は事前に作成した連想配列に対して，`jsPsych.timelineVariable('letter')`とキーを指定して，対応する要素を取り出します。

# timelineVariables（その１）

それでは，連想配列と`jsPsych.timelineVariable()`を用いて，たったひとつの試行変数だけで，4 条件それぞれの試行を実施できるようにしましょう。

まず，4 条件分の連想配列を作成しましょう。今回のサイモン課題では，`stimulus`の部分だけが試行ごとに変わるので，4 条件それぞれの`stimulus`を作成して（コピペしてきて），以下のように，**同じキーをつけて**連想配列にし,それを並べた配列を作成します。

```javascript
var trial_types = [
  { letter: '<div class="text_left">L</div>' },
  { letter: '<div class="text_left">R</div>' },
  { letter: '<div class="text_right">L</div>' },
  { letter: '<div class="text_right">R</div>' },
];
```

わかりにくいかもしれませんが，この`trial_types`という変数は，連想配列を要素に持つ配列になります。`=`の直後に`[`，最後の行に`]`があります。配列の要素は`,`でさえ区切っていれば，縦に並べることができます。もっと配列の要素がシンプルな方がわかりやすいかもしれません。以下の例を見てください。例に出てくる 2 つの配列は，どちらも同じ配列です。

```javascript
var hairetsu1 = [1, 2, 3, 4];
var hairetsu2 = [1, 2, 3, 4];
```

そして，`stimulus: jsPsych.timelineVariable('letter')`とした試行変数を一つだけ作成します。

```javascript
var trial = {
  type: 'html-keyboard-response',
  stimulus: jsPsych.timelineVariable('letter'),
  choices: ['f', 'j'], // 入力キーの指定
  trial_duration: 1000, // 試行の持続時間
};
```

最後に， `jsPsych.init()`の`timeline`を以下のように変更します。

```javascript
jsPsych.init({
  timeline: [
    {
      // [ と { 2つあります
      timeline_variables: trial_types,
      timeline: [trial],
    },
  ], // } と ] 2つあります
});
```

こうすることによって，`stimulus`で使用されている`jsPsych.timelineVariables()`が`trial_types`を参照してくれるようになります。実際に実験を走らせると，`timeline_variables`に指定した連想配列の配列（`trial_types`）から，連想配列`{letter: ...}`が順番に取り出されます。そして，`timeline`で指定されている試行変数`trial`の中で，`jsPsych.timelineVariables('letter')`が用いられている部分に，キー`letter`と対応する連想配列の要素を適応します。３種類のカッコ `()`,`[]`,`{}`が入り乱れているので，それぞれをどこで使っているのかを間違えないように注意してください。

まとめると，以下のようなコードになります。

```html:timeline_variables_1.html
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

  var trial_types = [
    {letter: '<div class="text_left">L</div>'},
    {letter: '<div class="text_left">R</div>'},
    {letter: '<div class="text_right">L</div>'},
    {letter: '<div class="text_right">R</div>'},
  ];

  var trial = {
    type: 'html-keyboard-response',
    stimulus: jsPsych.timelineVariable('letter'),
    choices: ['f', 'j'], // 入力キーの指定
    trial_duration: 1000, // 試行の持続時間
  }

  // trial_typesをランダム化する
  trial_types = jsPsych.randomization.repeat(trial_types, 2);

  jsPsych.init({
    timeline: [{ // [ と { 2つあります
      timeline_variables: trial_types,
      timeline: [trial],
    }], // } と ] 2つあります
    default_iti: 250,
  });
</script>

</html>
```

いかがでしょうか。`trial_L_left`, `trial_L_right`というように，各条件ごとの試行変数を作成するよりもコード全体がスッキリしたのではないでしょうか。また，試行変数を一つにまとめたことによって，もし，`choices`や`trial_duration`共通の項目に対する設定を変更する場合でも非常に容易に済ませることができそうです。

注意しないといけないのは，timeline_variables を設定すると，そこで指定されている配列`trial_types`に入っている要素の数が試行数になることです。ランダム化する場合は，`trial_types`の方に`randomization.repeat()`を適用するようにしてください。

# timelineVariables（その２）

さらにこのように思う人がいるかもしれません。

> `<div class="text_left">L</div>`も，`left`と`L`以外の部分はすべて共通なのだから，`<div class="text_..."></div>`も試行変数の中に入れてしまったほうがいいのではないか。

この発想も`jsPsych.timelineVariable()`で実装することができます。文字や位置をデータとして保存したい場合（**第 5 回（リンク貼る）**）はこれから紹介する方法のほうが便利です。内容はすこし難しいかもしれませんが，理解してもらえればと思います

このセクションの目標は，`<div class="text_位置">文字</div>`の`位置`と`文字`が試行ごとに変わるようにすることです。ということは`trial_types`を以下のように変更すれば大丈夫です。

```javascript
var trial_types = [
  { letter: 'L', pos: 'left' },
  { letter: 'R', pos: 'left' },
  { letter: 'L', pos: 'right' },
  { letter: 'R', pos: 'right' },
];
```

次に stimulus の部分です。位置と文字はそれぞれ`jsPsych.timelineVariable('pos')`，`jsPsych.timelineVariable('letter')`で呼び出せるので，それらを呼び出しつつ，`<div class="text_位置">文字</div>`となるように，以下のようにすればいい気がします。

```javascript
var trial = {
  type: 'html-keyboard-response',
  stimulus: '<div class="text_' + jsPsych.timelineVariable('pos') + '">' + jsPsych.timelineVariable('letter') + '</div>',
  choices: ['f', 'j'], // 入力キーの指定
  trial_duration: 1000, // 試行の持続時間
};
```

すでに`stimulus`の部分が見にくくなってしまいましたね。stimulus の部分で使っている`+`は文字列を連結しています。`timelineVariable()`で呼び出した値がそれぞれ`left`, `L`であれば，`<div class="text_` + `left` + `">` + `L` + `</div>` からすべてを連結して`<div class="text_left">L</div>`となります。

これらをまとめると，と言いたいところなのですが，上記のような`stimulus`の指定方法だと刺激はうまく提示されず，`function() { return timeline.timelineVariable(varname); }`というエラーメッセージが提示されます。なぜうまく回らないのかについては説明しません（できません）が，とにかく，エラーメッセージにあるように`function(){return ...}`という形にする必要があるようです。今回の例であれば，

```javascript
var trial = {
  type: 'html-keyboard-response',
  stimulus: function () {
    return (
      '<div class="text_' + jsPsych.timelineVariable('pos', true) + '">' + jsPsych.timelineVariable('letter', true) + '</div>'
    );
  },
  choices: ['f', 'j'], // 入力キーの指定
  trial_duration: 1000, // 試行の持続時間
};
```

とします。突然`jsPsych.timelineVariable()`に`true`が入ってきたと思いますが，`function(){return ...}`の形で使用する場合はこうする必要があるようです（理由は説明できません）。

ということで，以上をまとめると以下のようなコードになります。

<details><summary>コード例</summary><div>

```html:timeline_variables_2.html
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

  var trial_types = [
    {letter: 'L', pos: 'left'},
    {letter: 'R', pos: 'left'},
    {letter: 'L', pos: 'right'},
    {letter: 'R', pos: 'right'},
  ];

  var trial = {
    type: 'html-keyboard-response',
    stimulus: function(){
      return '<div class="text_' + jsPsych.timelineVariable('pos', true) + '">' + jsPsych.timelineVariable('letter', true) + '</div>'
    },
    choices: ['f', 'j'], // 入力キーの指定
    trial_duration: 1000, // 試行の持続時間
  }

  var trial_types = jsPsych.randomization.repeat(trial_types, 2);

  jsPsych.init({
    timeline: [{ // [ と { 2つあります
      timeline_variables: trial_types,
      timeline: [trial],
    }], // } と ] 2つあります
    default_iti: 250,
  });
</script>

</html>
```

</div></details>

その 1 と比べると，`trial_types`が見やすくなった代わりに，`stimulus`の部分が見にくくなってしまいました。どちらがいいかは好みと必要性によって変わってくるので，ご自身の実験内容に合わせて利用してください。なお，本チュートリアルでは各試行で提示された文字と位置もデータに保存します。その場合，その 2 の方法のほうが簡単なので，以降のチュートリアルで登場するコード例にはその 2 の方法を使います。

念の為補足しておくと，`stimulus:`以外にも`jsPsych.timelineVariable()`を利用することができます。例えば，以下のようにすれば，刺激によって提示時間を変えることができます。

```javascript
var trial_types = [
  { letter: '<div class="text_left">L</div>', duration: 1000 },
  { letter: '<div class="text_left">R</div>', duration: 2000 },
  { letter: '<div class="text_right">L</div>', duration: 3000 },
  { letter: '<div class="text_right">R</div>', duration: 4000 },
];

var trial = {
  type: 'html-keyboard-response',
  stimulus: jsPsych.timelineVariable('letter'),
  choices: ['f', 'j'],
  trial_duration: jsPsych.timelineVariable('duration'),
};
```

# おわりに

今回は`jsPsych.timelineVariable()`を使って必要な部分だけを試行ごとに変更できるようにすることで，試行変数を減らし，コードの管理をしやすくする方法を紹介しました。後半になるにつれて話が複雑になってしまったような気もしますが，`jsPsych.timelineVariable()`がないと jsPsych で実験を作るのが難しくなってしまうと思うので，「その１」の部分だけでも理解してもらえればと思います。

これでサイモン課題自体は完成しました。ただ，このままだと参加者が課題をどのように遂行したのかを知ることができません。そこで次回は，実験データを（手元の PC 上に）保存する方法を紹介します。

[【第 5 回】データの保存](https://qiita.com/snishym/items/be23aa7cbeeffa49d13a)
