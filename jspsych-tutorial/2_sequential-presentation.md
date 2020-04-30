# はじめに

本記事は，「jsPsych による心理学実験作成チュートリアル」の第 2 回の記事です。[第 1 回](https://qiita.com/snishym/items/45cfd220f7af8c7a13e0)では jsPsych をインストールし，とりあえず刺激を提示することを体験しました。今回は，刺激の系列提示および for 文について解説します。詳細は右側の見出しリストをご覧ください。

このチュートリアルシリーズの目的・概要等が気になった方はこちらの[全体のまとめ](https://qiita.com/snishym/items/1e0511f8622282993ed1)をご一読ください。

# サイモン課題とは

サイモン効果は，求められた反応と位置とが競合する場合に，競合しない場合と比べて反応が遅くなる効果です。例えば，左手のキーでの反応が求められた刺激が右側に提示されると，その刺激が左側に提示される場合と比べて反応が遅くなります。詳しくは文献等を参照してください。

本シリーズのサイモン課題では，「L」「R」という 2 つの文字を刺激として用いることにします。

# 刺激の素朴な系列提示

初めに，画面中央に L と R を 2 回ずつ提示してみましょう。以下にコードを示していますが，前回の復習も兼ねて，一度ご自身でコードを新しいファイルに書いてみてください。まったく空のファイルから始める必要はなくて，前回作成したコードをコピペしてもらって大丈夫です。

<details><summary>コード例</summary><div>

```html:simple_sequence.html
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
    stimulus: 'L',
    choices: ['f', 'j'], // 入力キーの指定
    trial_duration: 1000, // 試行の持続時間
  }

  var trial_R = {
    type: 'html-keyboard-response',
    stimulus: 'R',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  jsPsych.init({
    timeline: [trial_L, trial_R, trial_L, trial_R],
  });
</script>

</html>
```

</div></details>

さて，いかがでしょうか。実は前回説明していなかったのですが，`timeline:`の`[]`の中に L を提示する試行（コード例では`trial_L`）と R を提示する試行（`trial_R`）を 2 回ずつ入れることで，L と R を 2 回ずつ提示することができます。

ただ，このまま実際の試行数に合わせてコードを編集するのは不便です。例えば L と R を 100 回ずつ提示する場合，`[]`の中に`trial_L`と`trial_R`をそれぞれ 100 個ずつコピペで入れるのは大変ですし，正確性にも欠けます。同じ処理を指定の回数だけ繰り返すのは，`for`文を使うことで簡単に実現することが可能です。

# for 文

javascript での for 文は以下のように書くことができます（動作しません）。

```javascript
for (var i = 0; i < 繰り返したい回数; i++) {
  繰り返したい処理;
}
```

`for`文の直後にある`()`内には`;`区切りで 3 つの要素が入っています。以下のように，それぞれの要素で`for`文の繰り返し回数に関わる設定がなされています。

1. `i`という名前のカウンタを作成し， 0 の状態にする（`var i = 0`）
2. カウンタの上限を指定（`i < 繰り返したい回数`）
3. `{}`の処理が終わるたびにカウンタを 1 つすすめる（`i++`）

`繰り返したい回数`の部分に具体的な数字を入れることができます。jsPsych を使いながら`for`文の挙動を確認してみましょう。以下のコードをコピペして保存したファイルを開いてみてください。

```html:add-for.html
<!DOCTYPE html>
<html>
<head>
  <script src="../jspsych.js"></script>
  <script src="../plugins/jspsych-html-keyboard-response.js"></script>
  <link rel="stylesheet" href="../css/jspsych.css"></link>
</head>
<body></body>
<script>

  var sum = 0
  for (var i = 0; i < 10; i ++) {
    sum = sum + 2 // sum += 2 と書き換えられる
  }

  var show_sum = {
    type: 'html-keyboard-response',
    stimulus: String(sum), // 数字を文字列に変換
  }

  jsPsych.init({
    timeline: [show_sum],
  });
</script>

</html>
```

開いたブラウザに 20 と表示されたはずです。この例では，初期値を 0 に指定した`sum`という名前の変数に，2 ずつ足す`sum = sum + 2`という処理を for 文で繰り返しています。10 回繰り返しているので，20 が表示されます。例えば，`i < 30`にすれば画面には 60 と表示されるはずです。

## 配列

さて，`for`文で処理を指定回数繰り返す方法を紹介しました。今回実際に繰り返したいのは「`[]`の中に`trial_L`と`trial_R`を入れる」という処理です。次にその処理を実行する方法を紹介します。
まず，`[要素, 要素, ...]`について簡単に説明します。これは，配列（array）と呼ばれるデータ型で，これまで見てきたように，`[]`の中にカンマ区切りで複数の要素を入れることができます。この中には，数字や文字列などあらゆる種類の要素を入れることができます。

```javascript
var array1 = [1, 2, 3, 4];
var array2 = ["a", "b", "c"];
```

ここで注目してほしいのは，`trial_L`や`trial_R`のように，配列も`var 任意の名前 = [要素, 要素, ...]`というように，名前を付けて利用できるということです。したがって，最初のサンプルコードの`timeline:`に関わる部分を以下のように変更することができます。

```javascript
var trials = [trial_L, trial_R, trial_L, trial_R];
jsPsych.init({
  timeline: trials,
});
```

そして，`.push(新しい要素)`で，新しい要素を配列の最後に追加することができます。以下のコードの`trials`の最終的な中身は，直前のコードと同様に，`[trial_L, trial_R, trial_L, trial_R]`になります。

```javascript
var trials = [];
trials.push(trial_L);
trials.push(trial_R);
trials.push(trial_L);
trials.push(trial_R);
```

この`.push`を`for`の中で繰り返し実行することで，`trial_L`と`trial_R`を指定した回数だけ提示するプログラムを書くことができます。今回の最初に作成したコードを編集して，L と R が順番に 10 回ずつ提示されるようにしてみてください。

<details><summary>コード例</summary><div>

```html
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
    stimulus: 'L',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  var trial_R = {
    type: 'html-keyboard-response',
    stimulus: 'R',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  var trials = [];
  for (var i = 0; i < 10; i++) {
    trials.push(trial_L);
    trials.push(trial_R);
  }

  jsPsych.init({
    timeline: trials,
  });
</script>

</html>
```

</div></details>

# 刺激のランダム化

`for`文を使って，試行数を簡単に増やせるようになりました。しかし，現状のコードでは，L と R が規則的に提示されてしまっています。これでは参加者が次に提示される刺激を予測できてしまいます。これを回避するために，提示の順番をランダム化する必要があります。

jsPsych には，`jsPsych.randomization.ほにゃらら`というランダム化に関する一連の関数が揃っています（[公式リファレンス](https://www.jspsych.org/core_library/jspsych-randomization/)）。その一つである，`jsPsych.randomization.repeat`を使えば，任意の配列の中身をランダム化することができます。この関数は以下のように使います。

```javascript
var num_array = [1, 2, 3, 4, 5];
var num_array_rand = jsPsych.randomization.repeat(num_array, 1);
```

シャッフルしたい配列を`()`の最初に持ってこればいいだけです。それでは，先ほど for 文で作成した trial の配列`trials`をランダム化してみてください。

<details><summary>コード例</summary><div>

```html
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
    stimulus: 'L',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  var trial_R = {
    type: 'html-keyboard-response',
    stimulus: 'R',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  var trials = [];
  for (var i = 0; i < 10; i++) {
    trials.push(trial_L);
    trials.push(trial_R);
  }

  var trials_rand = jsPsych.randomization.repeat(trials, 1);

  jsPsych.init({
    timeline: trials_rand, // 名前を変更し忘れない
    default_iti: 250, // 同じ文字が繰り返し提示されているのをわかりやすくするため
  });

  // 以下のように元のtrialsを書き換えてしまってもこの場合は問題ない
  // trials = jsPsych.randomization.repeat(trials, 1);
  // jsPsych.init({
  //   timeline: trials,
  //   default_iti: 250
  // });
</script>

</html>
```

</div></details>

`jsPsych.randomization.repeat`の`()`の 2 つ目にある数字については言及していませんでした。この数字は，関数の名前からも推察できるように，指定した配列の各要素を何度ランダムに取り出すのかを決定しています。例えば，`2`とすれば，以下のような配列を作ることができます（以下の例は公式リファレンスから転載）

```javascript
var myArray = [1, 2, 3, 4, 5];
var shuffledArray = jsPsych.randomization.repeat(myArray, 2);

// output: shuffledArray = [1,3,4,2,2,4,5,1,5,3]
```

つまり，この数字を変更することでランダム化された試行系列（配列）`trials`を手に入れることができます。本チュートリアルで完成を目指すサイモン課題では，本記事で紹介した`for`文を使う必要は全く無かったということになります（以下のコード例を参照）。ただ，for 文を利用する機会はいずれあるので，覚えておいて損はないでしょう。

<details><summary>コード例</summary><div>

```html
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
    stimulus: 'L',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  var trial_R = {
    type: 'html-keyboard-response',
    stimulus: 'R',
    choices: ['f', 'j'],
    trial_duration: 1000,
  }

  var trials = [trial_L, trial_R]
  trials = jsPsych.randomization.repeat(trials, 10);

  jsPsych.init({
    timeline: trials,
    default_iti: 250, // 同じ文字が繰り返し提示されているのをわかりやすくするため
  });

</script>

</html>
```

</div></details>

# おわりに

今回は刺激をランダムに系列提示する方法を紹介しました。そのために for 文と`jsPsych.randomization.repeat`を利用しました。

しかしながら，刺激は画面中央にしか提示されていません。サイモン課題の一致条件・不一致条件を作り出すために，次回は刺激の提示位置を変更する方法を紹介します。

[【第3回】刺激の提示位置の調整](https://qiita.com/snishym/items/bec56308c67922c3b3df)
